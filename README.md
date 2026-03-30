# type4me-vocab-skill

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for managing [Type4Me](https://github.com/anthropics/type4me) ASR vocabulary corrections.

Tell your agent which word is being misrecognized and what it should be. The skill automatically:

- **Infers additional misrecognition variants** based on phonetic patterns (homophones, syllable boundary errors, Chinese transliterations, etc.)
- **Adds snippet corrections** (post-ASR text replacement, works on any word)
- **Adds hotwords** when appropriate (ASR model weight boosting, only for words in the model's vocabulary)
- **Deduplicates** against both built-in and user entries
- **Triggers a live reload** via `type4me://reload-vocabulary` URL scheme

## Usage

```
/type4me-vocab Ghostty 被识别成 ghosty
/type4me-vocab queen 3.5需要写成Qwen3.5
/type4me-vocab 纠正 Svelte
```

The skill handles two distinct mechanisms:

| Mechanism | What it does | When to use |
|-----------|-------------|-------------|
| **Snippets** (映射词) | Regex find-replace after ASR output | Always. Every correction goes here. |
| **Hotwords** (热词) | Boosts word probability in ASR model | Only for words that exist in the model's vocabulary. |

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
