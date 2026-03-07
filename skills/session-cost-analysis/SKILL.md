---
name: session-cost-analysis
description: "Analyze Cortex Code CLI session costs with detailed token tracking. Use when: analyzing costs, checking token usage, reviewing session expenses, comparing model costs, tracking spending. Triggers: session cost, token usage, how much did this cost, analyze costs, cost breakdown, session expenses, token tracking."
---

# Session Cost Analysis

Efficiently analyze Cortex Code CLI session costs with detailed tracking of input/output/cache tokens at different price points. Uses **offline token counting** (no LLM calls) for cost-efficient analysis.

## When to Use

Use this skill when the user wants to:
- Analyze costs for the current or past sessions
- See token usage breakdown (input/output/cache)
- Compare costs across different models
- Track historical spending trends
- Identify which tools consume the most tokens
- Get detailed cost breakdowns

**Triggers:** "session cost", "token usage", "how much did this cost", "analyze costs", "cost breakdown", "session expenses"

## Workflow

### Step 1: Determine Analysis Scope

**Ask** the user what they want to analyze:

```
What would you like to analyze?

1. Current session only
2. Specific session by ID
3. All sessions (historical view)
4. Last N sessions

Please select (1-4):
```

**Capture:**
- Scope selection
- Session ID (if specific session)
- Number of sessions (if limited historical)

**⚠️ STOP**: Wait for user response before proceeding.

### Step 2: Detect Model Used

**Determine** which model was used in the session(s):

1. **Check** session metadata for model info
2. **Default** to `claude-sonnet-4-5` if not specified
3. **Ask** user if model detection fails:
   ```
   Which model was used?
   
   1. claude-sonnet-4-5 (default)
   2. claude-opus-4-5
   3. claude-haiku-4-5
   ```

**Note:** Model affects pricing:
- **Sonnet**: $3/$15/$0.30 per 1M tokens (input/output/cache)
- **Opus**: $15/$75/$1.50 per 1M tokens (premium)
- **Haiku**: $0.80/$4/$0.08 per 1M tokens (fast)

### Step 3: Run Cost Analysis

**Execute** the analysis script using `uv`:

```bash
# For current/most recent session
uv run --project <SKILL_DIR> python <SKILL_DIR>/scripts/analyze_costs.py \\
  --model <model_name>

# For specific session
uv run --project <SKILL_DIR> python <SKILL_DIR>/scripts/analyze_costs.py \\
  --session <session_id> \\
  --model <model_name>

# For all sessions (historical)
uv run --project <SKILL_DIR> python <SKILL_DIR>/scripts/analyze_costs.py \\
  --all \\
  --model <model_name>

# With detailed tool breakdown
uv run --project <SKILL_DIR> python <SKILL_DIR>/scripts/analyze_costs.py \\
  --breakdown \\
  --model <model_name>
```

**What the script does:**
1. Locates Cortex Code session logs
2. Reads session transcript (JSONL format)
3. Counts tokens using `tiktoken` (offline, no API calls)
4. Tracks input/output/cache tokens separately
5. Identifies tool usage and token consumption per tool
6. Calculates costs based on pricing table
7. Generates formatted terminal output

**Output:** Rich terminal display with:
- Token summary (input/output/cache)
- Cost breakdown by token type
- Tool usage breakdown (top 10 tools)
- Historical trends (if --all flag used)

### Step 4: Present Results

**Display** the analysis results to the user.

**Terminal output includes:**

1. **Session Overview**
   - Session ID
   - Model used
   - Timestamp

2. **Token Usage Table**
   ```
   Type            Count        Cost
   ─────────────────────────────────
   Input Tokens    45,230      $0.1357
   Output Tokens   12,450      $0.1868
   Cache Tokens    125,000     $0.0375
   ─────────────────────────────────
   Total           182,680     $0.3600
   ```

3. **Cost by Tool** (if breakdown requested)
   ```
   Tool                Calls    Tokens    Cost
   ────────────────────────────────────────────
   snowflake_sql       15       32,450    $0.0974
   read                42       28,120    $0.0844
   bash                8        12,300    $0.0369
   grep                5        8,200     $0.0246
   ...
   ```

4. **Historical Summary** (if --all flag used)
   - List of recent sessions
   - Per-session costs
   - Total spending across all sessions

**Interpret** results for user:
- Highlight highest-cost operations
- Identify optimization opportunities (e.g., cache more, use cheaper model)
- Compare costs across sessions if relevant

### Step 5: Offer Export (Optional)

**Ask** if user wants to export analysis to JSON:

```
Would you like to export this analysis to JSON for further processing? (yes/no)
```

**If yes:**
```bash
uv run --project <SKILL_DIR> python <SKILL_DIR>/scripts/analyze_costs.py \\
  --session <session_id> \\
  --model <model_name> \\
  --json analysis_output.json
```

## Tools

### Script: analyze_costs.py

**Description**: Offline token counter and cost calculator with input/output/cache tracking.

**Usage:**
```bash
uv run --project <SKILL_DIR> python <SKILL_DIR>/scripts/analyze_costs.py [OPTIONS]
```

**Arguments:**
- `--session <id>`: Specific session ID to analyze (default: most recent)
- `--all`: Analyze all sessions (historical view)
- `--model <name>`: Model used (default: claude-sonnet-4-5)
- `--breakdown`: Show detailed tool breakdown
- `--json <file>`: Export analysis to JSON file

**Examples:**
```bash
# Analyze current session
uv run --project /path/to/skill python /path/to/skill/scripts/analyze_costs.py

# Analyze specific session with Opus pricing
uv run --project /path/to/skill python /path/to/skill/scripts/analyze_costs.py \\
  --session abc123 --model claude-opus-4-5

# Historical view with breakdown
uv run --project /path/to/skill python /path/to/skill/scripts/analyze_costs.py \\
  --all --breakdown
```

**When to use:** Anytime you need cost analysis
**When NOT to use:** For real-time cost tracking during active session (this is post-session analysis)

## Pricing Table

Current Snowflake Cortex pricing (per 1M tokens):

| Model | Input | Output | Cache |
|-------|-------|--------|-------|
| claude-sonnet-4-5 | $3.00 | $15.00 | $0.30 |
| claude-opus-4-5 | $15.00 | $75.00 | $1.50 |
| claude-haiku-4-5 | $0.80 | $4.00 | $0.08 |

**Key Points:**
- Output tokens cost **5x** input tokens
- Cache tokens cost **10%** of input tokens (huge savings!)
- Larger context = more cache opportunities

## Stopping Points

- ✋ Step 1: After determining analysis scope (wait for user selection)
- ✋ Step 5: After presenting results (ask about export)

## Prerequisites

### Install uv (if not already installed)

Check if uv is installed:
```bash
uv --version
```

If not installed:
```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Or with Homebrew
brew install uv
```

After installation, restart your terminal.

## Output

**Primary output:** Formatted terminal summary with:
- Token usage breakdown (input/output/cache)
- Cost calculation per token type
- Tool-by-tool breakdown
- Historical trends (if requested)

**Optional output:** JSON export for further analysis

## Success Criteria

- ✅ Token counts are accurate (using tiktoken)
- ✅ Input/output/cache tokens tracked separately
- ✅ Costs calculated with correct pricing per token type
- ✅ Tool breakdown identifies highest-cost operations
- ✅ Output is clear and actionable

## Cost Efficiency Note

This skill uses **offline token counting** via `tiktoken` library:
- Zero LLM API calls during analysis
- All computation happens locally
- Only reads session logs (no new tokens consumed)
- Analyzing 100 sessions costs $0.00 in API fees

## Troubleshooting

**Issue:** "Could not find Cortex Code session logs"
- **Solution:** Sessions are stored in `~/.snowflake/cortex/sessions/` or similar. Check the error message for tried locations.

**Issue:** "No transcript found"
- **Solution:** Session might be too old or logs not yet written. Try analyzing a more recent session.

**Issue:** Token counts seem off
- **Solution:** Token counting is approximate. Actual costs may vary slightly due to model-specific tokenization nuances.

**Issue:** Cache tokens showing as 0
- **Solution:** Cache tokens are only present when context is reused. Short sessions or new conversations won't have cache hits.

## Notes

- Token counting uses `tiktoken` with cl100k_base encoding (Claude uses similar tokenization to GPT-4)
- Cache tokens represent prompt tokens that were cached and reused (not reprocessed)
- Tool breakdown shows tokens in tool inputs/outputs, helping identify expensive operations
- Historical analysis helps track spending trends over time
