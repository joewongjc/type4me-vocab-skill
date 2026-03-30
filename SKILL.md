---
name: type4me-vocab
description: 当用户说某个词识别不准、被错误识别、需要纠正时，使用此 skill 添加 ASR 映射词和/或热词到 Type4Me。支持推演变体。
user_invocable: true
arguments: 目标词和（可选的）错误识别形式，如 "Ghostty 被识别成 ghosty" 或 "纠正 Svelte"
---

# Type4Me 词汇纠错 Skill

根据用户提供的目标词，添加 ASR 纠错规则到 Type4Me 的词汇系统。

## 核心概念

**映射词 (Snippets)**: 后处理文本替换。ASR 输出后用正则做 find-replace。匹配规则：大小写不敏感 + 空格不敏感（trigger 中字符间插入 `\s*`）。**所有纠错都走这个**。

**热词 (Hotwords)**: 给 ASR 引擎的加权提示，提高词的识别概率。前提是词必须在模型词表里才有效。**只对已有词表的标准术语有用**。

## 操作的文件

```
~/Library/Application Support/Type4Me/
  snippets.json          # 用户映射词 ← 改这个
  hotwords.json          # 用户热词   ← 改这个
  builtin-snippets.json  # 内置映射词（只读，用于查重）
  builtin-hotwords.json  # 内置热词（只读，用于查重）
  hotwords.txt           # Python 服务用的合并版（需重新生成）
```

**绝不修改 builtin-*.json 文件。**

## 执行步骤

### 0. 前置检查

在执行任何操作前，先验证 Type4Me 环境：

```bash
python3 << 'PYEOF'
import subprocess, sys
from pathlib import Path

app_support = Path.home() / "Library/Application Support/Type4Me"
app_path = Path("/Applications/Type4Me.app")

# 检查 app 是否安装
if not app_path.exists():
    print("ERROR: Type4Me 未安装在 /Applications/Type4Me.app")
    print("请先安装 Type4Me: https://github.com/anthropics/type4me")
    sys.exit(1)

# 检查数据目录
if not (app_support / "builtin-snippets.json").exists():
    print("ERROR: Type4Me 数据目录不完整，请先启动一次 Type4Me")
    sys.exit(1)

# 检查 URL scheme 支持（type4me://reload-vocabulary）
try:
    result = subprocess.run(
        ["/usr/libexec/PlistBuddy", "-c",
         "Print :CFBundleURLTypes:0:CFBundleURLSchemes:0",
         str(app_path / "Contents/Info.plist")],
        capture_output=True, text=True
    )
    has_url_scheme = result.stdout.strip() == "type4me"
except Exception:
    has_url_scheme = False

print(f"Type4Me: 已安装")
print(f"数据目录: OK")
print(f"URL scheme (reload): {'支持' if has_url_scheme else '不支持 (旧版本，文件修改仍生效，热词需手动刷新)'}")
PYEOF
```

如果 URL scheme 不支持，后续步骤 5 的 `open type4me://` 会跳过，并提醒用户去设置界面手动点刷新，或重启 app。

### 1. 解析意图

从 `$ARGUMENTS` 或对话上下文提取：
- **目标词**：正确输出（如 "Ghostty"、"SvelteKit"、"飞书"）
- **已知错误形式**（可选）：用户提到的具体错误识别

如果只有目标词没有错误形式，全靠推演。

### 2. 推演错误识别变体

即使用户给了一个错误形式，也要推演其他可能的变体。

**英文词常见 ASR 错误模式：**
- 同音/近音替换: Claude → Cloud, Clod, Clawed
- 音节边界错切: Anthropic → and tropic, an tropic
- 元音混淆: DeepSeek → deep sick, deep sec
- 辅音混淆: Grok → grock
- 词尾吞音/加音: Svelte → svelt
- 拆词/合词: GitHub → git hub, get hub
- 连读: VS Code → vscode

**中文语境特殊错误：**
- 音译: Cursor → 克色
- 中英混合断词错误

**注意**：不过度推演，每个变体应是真实可能的 ASR 错误。通常 3-8 个变体够了。

### 3. 判断是否加热词

**加** ✓：标准英文单词/短语、知名品牌、常见技术缩写、中文常用词
**不加** ✗：非标准拼写造词（Type4Me）、很新的词（模型训练后才出现）、纯缩写组合

### 4. 查重 & 写入

用一段 Python 完成读取、查重、写入全流程：

```bash
python3 << 'PYEOF'
import json
from pathlib import Path

app_support = Path.home() / "Library/Application Support/Type4Me"

# 读取现有数据
def load_json(path, default):
    return json.loads(path.read_text()) if path.exists() else default

snippets = load_json(app_support / "snippets.json", [])
hotwords = load_json(app_support / "hotwords.json", [])
builtin_snippets = load_json(app_support / "builtin-snippets.json", [])
builtin_hotwords = load_json(app_support / "builtin-hotwords.json", [])

# 构建查重集合
all_snippet_triggers = {s["trigger"].lower().replace(" ", "") for s in builtin_snippets + snippets}
all_hotwords_lower = {w.lower() for w in builtin_hotwords + hotwords}

# ── 在这里填入推演结果 ──
new_snippets = [
    # {"trigger": "错误形式", "replacement": "正确形式"},
]
new_hotwords = [
    # "词1",
]

# 写入映射词（去重）
added_snippets = []
for s in new_snippets:
    key = s["trigger"].lower().replace(" ", "")
    if key not in all_snippet_triggers:
        snippets.append(s)
        all_snippet_triggers.add(key)
        added_snippets.append(f'  {s["trigger"]} → {s["replacement"]}')
    else:
        print(f'跳过(已存在): {s["trigger"]}')

# 写入热词（去重）
added_hotwords = []
for w in new_hotwords:
    if w.lower() not in all_hotwords_lower:
        hotwords.append(w)
        all_hotwords_lower.add(w.lower())
        added_hotwords.append(w)
    else:
        print(f'跳过(已存在): {w}')

# 写 JSON
for path, data in [(app_support / "snippets.json", snippets), (app_support / "hotwords.json", hotwords)]:
    path.write_text(json.dumps(data, ensure_ascii=False, indent=2))

# 生成合并版 hotwords.txt
seen = set()
merged = []
for w in builtin_hotwords + hotwords:
    if w.lower() not in seen:
        seen.add(w.lower())
        merged.append(w)
(app_support / "hotwords.txt").write_text("\n".join(merged))

# 汇报
if added_snippets:
    print(f'添加 {len(added_snippets)} 条映射词:')
    print("\n".join(added_snippets))
if added_hotwords:
    print(f'添加热词: {", ".join(added_hotwords)}')
if not added_snippets and not added_hotwords:
    print("没有新增条目（全部已存在）")
PYEOF
```

### 5. 触发 App 重载

根据步骤 0 的检测结果决定重载方式：

```bash
python3 << 'PYEOF'
import subprocess
from pathlib import Path

app_path = Path("/Applications/Type4Me.app")

# 检测 URL scheme 支持
has_url_scheme = False
try:
    result = subprocess.run(
        ["/usr/libexec/PlistBuddy", "-c",
         "Print :CFBundleURLTypes:0:CFBundleURLSchemes:0",
         str(app_path / "Contents/Info.plist")],
        capture_output=True, text=True
    )
    has_url_scheme = result.stdout.strip() == "type4me"
except Exception:
    pass

if has_url_scheme:
    r = subprocess.run(["open", "type4me://reload-vocabulary"], capture_output=True)
    if r.returncode == 0:
        print("已触发 App 重载（热词同步 + Python 服务重启）")
    else:
        print("App 未运行，下次启动时自动加载")
else:
    print("当前版本不支持 URL scheme 自动重载")
    print("  映射词: 下次录音自动生效（无需操作）")
    print("  热词: 请打开 Type4Me 设置 → 词汇管理 → 点击 ⟳ 刷新按钮，或重启 App")
PYEOF
```

### 6. 反馈

简洁告诉用户：
- 添加了哪些映射词（trigger → replacement）
- 是否添加了热词及原因
- 跳过了哪些已存在的

## 兼容性

| 功能 | 最低版本 | 说明 |
|------|---------|------|
| 映射词编辑 | 任何版本 | 直接写 JSON 文件，下次录音自动读取 |
| 热词编辑 | 任何版本 | 同上，云端 ASR 下次录音自动读取 |
| `type4me://reload-vocabulary` | 需 URL scheme 支持 | 触发 Python 本地服务重启以加载新热词 |

步骤 0 的前置检查会自动检测 URL scheme 支持情况。如果不支持：
- **映射词**：无影响，下次录音立即生效
- **热词（云端 ASR）**：无影响，下次录音立即生效
- **热词（本地 Python 服务）**：需要用户手动操作：设置 → 词汇管理 → 点击 ⟳ 刷新，或重启 App
