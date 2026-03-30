# type4me-vocab-skill

[English](README.md)

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
