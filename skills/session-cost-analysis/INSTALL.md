# Installation Guide - Session Cost Analysis

## Quick Install

### Option 1: Global Installation (Recommended)

Install globally to use across all Cortex Code projects:

```bash
# Copy skill to global Cortex skills directory
cp -r /Users/jago.stokes/github/cortex-code-plus/skills/session-cost-analysis \
  ~/.snowflake/cortex/skills/

# Install dependencies (if using without uv)
pip3 install tiktoken rich
```

### Option 2: Project-Local Installation

Install locally for this project only:

```bash
# From your project directory
mkdir -p .cortex/skills
cp -r /Users/jago.stokes/github/cortex-code-plus/skills/session-cost-analysis \
  .cortex/skills/

# Install dependencies
pip3 install tiktoken rich
```

## Prerequisites

### Python 3.11+

Check your Python version:
```bash
python3 --version
```

### Install uv (Optional but Recommended)

uv automatically manages dependencies:

```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Or with Homebrew
brew install uv
```

### Manual Dependency Installation

If not using uv:

```bash
pip3 install tiktoken rich
```

## Usage

Once installed, use in Cortex Code with natural language:

```
"Analyze session costs"
"How much did this session cost?"
"Show me token usage"
"Compare opus vs sonnet costs"
"Historical session costs"
```

Or invoke directly:
```
$session-cost-analysis
```

## Testing the Installation

Test the script directly:

```bash
# With uv (auto-installs dependencies)
cd ~/.snowflake/cortex/skills/session-cost-analysis
uv run python scripts/analyze_costs.py --help

# Without uv (requires manual dependency install)
python3 scripts/analyze_costs.py --help
```

Run analysis:

```bash
# Analyze most recent session (Sonnet pricing)
python3 scripts/analyze_costs.py

# Analyze with Opus pricing
python3 scripts/analyze_costs.py --model claude-opus-4-5

# Historical analysis
python3 scripts/analyze_costs.py --all

# Export to JSON
python3 scripts/analyze_costs.py --json output.json
```

## Model Options

Specify which model pricing to use:

- `claude-sonnet-4-5` (default) - $3/$15/$0.30 per 1M tokens
- `claude-opus-4-5` - $15/$75/$1.50 per 1M tokens
- `claude-haiku-4-5` - $0.80/$4/$0.08 per 1M tokens

**Note**: Since Cortex doesn't log which model was actually used, you must specify it manually with `--model`.

## Troubleshooting

### "Could not find Cortex Code conversations directory"

Cortex stores conversations in `~/.snowflake/cortex/conversations/`. If this directory doesn't exist:

1. Make sure you've run Cortex Code at least once
2. Check alternative locations: `~/.config/snowflake/cortex/conversations/` or `~/.local/share/cortex/conversations/`

### "No conversations found"

You need at least one Cortex Code session to analyze. Start a new Cortex session and run a few commands, then try again.

### "Required packages not installed"

Install dependencies manually:
```bash
pip3 install tiktoken rich
```

Or use uv which auto-installs them.

## What It Analyzes

✅ **Input tokens** - User messages, tool results  
✅ **Output tokens** - Assistant responses, tool calls  
✅ **Tool breakdown** - Which tools cost the most  
✅ **Historical trends** - Spending across all sessions

⚠️ **Cache tokens** - Not tracked (Cortex doesn't log cache hits yet)

## Example Output

```
╭────────────────────────────────────────╮
│ Session Cost Analysis                  │
│ Session: abc123...                     │
│ Model: claude-sonnet-4-5               │
│ Timestamp: 2026-03-07                  │
╰────────────────────────────────────────╯

Token Usage
─────────────────────────────
Type            Count    Cost
─────────────────────────────
Input Tokens    1,977   $0.01
Output Tokens   4,128   $0.06
Cache Tokens        0   $0.00
─────────────────────────────
Total           6,105   $0.07
```

## Cost Comparison Example

Same session analyzed with different model pricing:

| Model | Input | Output | Total Cost |
|-------|-------|--------|------------|
| Haiku | 1,977 | 4,128 | **$0.02** |
| Sonnet | 1,977 | 4,128 | **$0.07** |
| Opus | 1,977 | 4,128 | **$0.34** |

**Insight**: Opus costs ~5x more than Sonnet for the same session!

## Uninstallation

Remove the skill:

```bash
# Global
rm -rf ~/.snowflake/cortex/skills/session-cost-analysis

# Local
rm -rf .cortex/skills/session-cost-analysis
```

## Support

For issues or questions, open an issue in the cortex-code-plus repository.
