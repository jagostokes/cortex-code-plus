# Session Cost Analysis Skill

Analyze Cortex Code CLI session costs with detailed token tracking (input/output/cache).

## Features

- ✅ **Cost-efficient**: Uses offline token counting (no LLM API calls)
- ✅ **Detailed breakdown**: Tracks input/output/cache tokens separately
- ✅ **Tool analysis**: Identifies which tools consume the most tokens
- ✅ **Historical trends**: Analyze spending across multiple sessions
- ✅ **Terminal output**: Beautiful, formatted cost reports

## Installation

### Quick Install (Local to Project)

From your Cortex Code project directory:

```bash
# Copy skill to local skills directory
mkdir -p .cortex/skills
cp -r /Users/jago.stokes/github/cortex-code-plus/skills/session-cost-analysis .cortex/skills/

# Skill is now available in this project
```

### Global Install (All Projects)

```bash
# Copy skill to global skills directory
mkdir -p ~/.snowflake/cortex/skills
cp -r /Users/jago.stokes/github/cortex-code-plus/skills/session-cost-analysis ~/.snowflake/cortex/skills/

# Skill is now available in all Cortex Code sessions
```

## Usage

In Cortex Code, use natural language triggers:

```
"Analyze session costs"
"How much did this session cost?"
"Show me token usage"
"Cost breakdown by tool"
"Historical session costs"
```

Or invoke directly:

```
$session-cost-analysis
```

## Pricing Reference

Current Snowflake Cortex pricing (per 1M tokens):

| Model | Input | Output | Cache |
|-------|-------|--------|-------|
| claude-sonnet-4-5 | $3.00 | $15.00 | $0.30 |
| claude-opus-4-5 | $15.00 | $75.00 | $1.50 |
| claude-haiku-4-5 | $0.80 | $4.00 | $0.08 |

**Key insight**: Cache tokens save 90% vs regular input tokens!

## Example Output

```
╭───────────────────────────────────────╮
│ Session Cost Analysis                 │
│ Session: abc123def456                 │
│ Model: claude-sonnet-4-5              │
│ Timestamp: 2025-03-07T10:30:00        │
╰───────────────────────────────────────╯

Token Usage
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Type            Count        Cost
─────────────────────────────────────────
Input Tokens    45,230      $0.1357
Output Tokens   12,450      $0.1868
Cache Tokens    125,000     $0.0375
─────────────────────────────────────────
Total           182,680     $0.3600

Cost by Tool
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Tool                Calls    Tokens    Cost
────────────────────────────────────────────
snowflake_sql       15       32,450    $0.0974
read                42       28,120    $0.0844
bash                8        12,300    $0.0369
...
```

## Development

### Structure

```
session-cost-analysis/
├── SKILL.md              # Skill definition (for AI agent)
├── README.md             # This file (for humans)
├── pyproject.toml        # Python dependencies
└── scripts/
    └── analyze_costs.py  # Token counter + cost calculator
```

### Dependencies

- `tiktoken` - Offline token counting
- `rich` - Terminal formatting

Dependencies are automatically installed by `uv` when the skill runs.

### Testing

```bash
# Test the script directly
cd /Users/jago.stokes/github/cortex-code-plus/skills/session-cost-analysis

# Analyze most recent session
uv run python scripts/analyze_costs.py

# Analyze all sessions
uv run python scripts/analyze_costs.py --all

# Export to JSON
uv run python scripts/analyze_costs.py --json output.json
```

## How It Works

1. **Session Log Discovery**: Finds Cortex Code session logs on your system
2. **Offline Token Counting**: Uses `tiktoken` to count tokens (no API calls)
3. **Categorization**: Separates input/output/cache tokens
4. **Tool Attribution**: Maps token usage to specific tools (SQL, bash, etc.)
5. **Cost Calculation**: Applies pricing per token type
6. **Terminal Display**: Formats results with `rich`

## Limitations

- Token counts are approximate (different models tokenize slightly differently)
- Requires session logs to exist (can't analyze live sessions)
- Cache token detection depends on session metadata availability

## Contributing

This skill is part of the `cortex-code-plus` repository. PRs welcome!

## License

MIT
