<p align="center">
  <a href="#english">English</a> | <a href="#中文">中文</a>
</p>

---

<a id="english"></a>

# type4me-vocab-skill

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for managing [Type4Me](https://github.com/anthropics/type4me) ASR vocabulary corrections.

Tell your agent which word is being misrecognized and what it should be. The skill automatically:

- **Infers additional misrecognition variants** based on phonetic patterns (homophones, syllable boundary errors, transliterations, etc.)
- **Adds snippet corrections** (post-ASR text replacement, works on any word)
- **Adds hotwords** when appropriate (ASR model weight boosting, only effective for words already in the model's vocabulary)
- **Deduplicates** against both built-in and user entries
- **Triggers a live reload** via `type4me://reload-vocabulary` URL scheme

## Usage

```
/type4me-vocab "Ghostty" is being recognized as "ghosty"
/type4me-vocab fix "Svelte"
/type4me-vocab "queen 3.5" should be "Qwen3.5"
```

The skill handles two distinct mechanisms:

| Mechanism | What it does | When to use |
|-----------|-------------|-------------|
| **Snippets** | Regex find-replace after ASR output | Always. Every correction goes here. |
| **Hotwords** | Boosts word probability in the ASR model | Only for words that exist in the model's vocabulary. |

## Install

Copy `SKILL.md` into your Claude Code skills directory:

```bash
# Clone and copy
git clone https://github.com/joewongjc/type4me-vocab-skill.git
mkdir -p ~/.claude/skills/type4me-vocab
cp type4me-vocab-skill/SKILL.md ~/.claude/skills/type4me-vocab/

# Or just download the file
curl -o ~/.claude/skills/type4me-vocab/SKILL.md \
  https://raw.githubusercontent.com/joewongjc/type4me-vocab-skill/main/SKILL.md
```

## Requirements

- [Type4Me](https://github.com/anthropics/type4me) installed at `/Applications/Type4Me.app`
- Python 3 (for JSON manipulation)
- **URL scheme reload** (`type4me://reload-vocabulary`): requires Type4Me with URL scheme support. The skill auto-detects this and falls back gracefully on older versions.

## License

MIT

---

<a id="中文"></a>

# type4me-vocab-skill

一个 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 技能，用于管理 [Type4Me](https://github.com/anthropics/type4me) 的语音识别词汇纠错。

告诉你的 agent 哪个词识别不准、应该是什么，它会自动：

- **推演其他可能的错误识别变体**（同音词、音节错切、音译等）
- **添加映射词纠错**（识别后的文本替换，适用于任何词）
- **酌情添加热词**（提高 ASR 模型对该词的识别权重，仅对模型词表内的词有效）
- **自动查重**，避免与内置和用户已有条目重复
- **触发实时重载**，通过 `type4me://reload-vocabulary` 通知 App 刷新

## 用法

```
/type4me-vocab Ghostty 被识别成 ghosty
/type4me-vocab 纠正 Svelte
/type4me-vocab queen 3.5 需要写成 Qwen3.5
```

本技能操作两种机制：

| 机制 | 作用 | 使用场景 |
|------|------|---------|
| **映射词** | 识别结果的正则替换 | 所有纠错都走这个 |
| **热词** | 提高词在 ASR 模型中的识别概率 | 仅限模型词表中已有的标准术语 |

## 安装

将 `SKILL.md` 复制到 Claude Code 技能目录：

```bash
# 克隆并复制
git clone https://github.com/joewongjc/type4me-vocab-skill.git
mkdir -p ~/.claude/skills/type4me-vocab
cp type4me-vocab-skill/SKILL.md ~/.claude/skills/type4me-vocab/

# 或者直接下载单文件
curl -o ~/.claude/skills/type4me-vocab/SKILL.md \
  https://raw.githubusercontent.com/joewongjc/type4me-vocab-skill/main/SKILL.md
```

## 环境要求

- [Type4Me](https://github.com/anthropics/type4me) 已安装在 `/Applications/Type4Me.app`
- Python 3
- **自动重载功能**（`type4me://reload-vocabulary`）：需要支持 URL scheme 的 Type4Me 版本。技能会自动检测，旧版本下文件修改仍然生效，仅本地热词需手动刷新。

## 许可证

MIT
