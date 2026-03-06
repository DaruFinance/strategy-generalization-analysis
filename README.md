# Strategy Generalization Analysis

Analyzes how well walk-forward-optimized strategies generalize from historical windows into unseen "live proxy" windows. Takes the output produced by [`run_strategies.py`](../runner/run_strategies.py) and answers the question: **given that a strategy passed your in-sample and robustness filters in the past, how often does it stay profitable going forward?**

## Features

- Sliding-window generalization analysis across all WFO windows
- Robustness funnel filters (IS + robustness tag + OOS + OOS robustness tag)
- Beta-binomial lower bound statistics for each filter pipeline
- Portfolio construction from top-ranked strategies
- Challenge-style pass rate simulation (milestone / drawdown / daily loss limits)
- META mode: re-runs across every possible window offset for temporal stability analysis
- Excel export with strategy metrics, trade-list derived stats, and portfolio results

## Requirements

```bash
pip install -r requirements.txt
```

| Package | Purpose |
|---------|---------|
| `numpy` | Numerical computation |
| `pandas` | Data loading and Excel export |
| `matplotlib` | Equity curve and distribution plots |
| `scipy` | Beta-binomial lower bound (statistical confidence) |
| `openpyxl` | Excel `.xlsx` output |

## Input: Strategy Files from run_strategies.py

This script reads the output folder produced by `run_strategies.py`. Each strategy subfolder must contain a `.txt` results file with lines in this format:

```
W01 IS   PF: 1.23  ROI: $456  Trades: 80  Win: 52.3%
W01 OOS  PF: 1.10  ROI: $210  Trades: 40  Win: 54.0%
W01 IS+ENT  PF: 1.18  ...
W01 OOS+ENT PF: 1.05  ...
```

The robustness-tagged lines (`IS+ENT`, `OOS+ENT`, etc.) are generated automatically when you enable robustness switches in `run_strategies.py`:

| Switch in run_strategies.py | Tag produced | Required for robustness tests |
|-----------------------------|--------------|-------------------------------|
| `ENTRY_DRIFT = True` | `ENT` | Yes |
| `INDICATOR_VARIANCE = True` | `IND` | Yes |
| `FEE_SHOCK = True` | `FEE` | Yes |
| `SLIPPAGE_SHOCK = True` | `SLI` | Yes |

> **You must enable at least one robustness switch** in `run_strategies.py` before running this analysis, otherwise no tests will be marked as eligible for export.

## Quick Start

### Step 1 — Point to your strategies folder

Open `strategy-generalization-analysis.py` and set `root_dir` at the top of the CONFIG section:

```python
root_dir = r"C:\Strategies\MyRun"  # Windows
# root_dir = "/home/user/strategies/myrun"  # Linux/Mac
```

This must be the same folder you passed as `base_output` in `run_strategies.py`.

### Step 2 — Run once to see available tests

```bash
python strategy-generalization-analysis.py
```

Read the **TEST ID MAPPING** block printed to the console. Each test shows:
- Its pipeline definition (what filters it applies)
- Whether it requires robustness data
- `Eligible for manual export selection (robustness-required): YES / NO`

### Step 3 — Select a test and re-run

Set `SELECT_TEST_NUMBER` in the CONFIG section to a test ID marked as eligible, then re-run:

```bash
python strategy-generalization-analysis.py
```

An Excel file is saved to `root_dir` with:
- All passing strategies and their metrics
- Trade-list derived stats (ROI in R, PF, MaxDD, equity curves)
- Portfolio combinations with live-proxy performance stats

## Configuration Reference

All settings are in the `# CONFIG` section at the top of the script.

| Parameter | Default | Description |
|-----------|---------|-------------|
| `root_dir` | *(must be set)* | Path to your `run_strategies.py` output folder |
| `EXPECTED_WINDOWS` | `6` | Number of WFO windows per run (match your backtester setting) |
| `SELECT_TEST_NUMBER` | `10` | Which pipeline to export (see TEST ID MAPPING output) |
| `META_MODE` | `True` | Run across all sliding window offsets for temporal stability |
| `LIVE_MODE` | `False` | Re-apply filter without held-out live windows (deployment mode) |
| `ENABLE_PLOTS` | `True` | Show matplotlib plots (set False for headless/batch runs) |
| `PORTFOLIO_SIZE` | `20` | Strategies per sampled portfolio |
| `MAX_PORTFOLIO_COMBOS` | `10000` | Max portfolios to sample |
| `TOP_SHARPE_STRATEGIES` | `24` | Pool size for portfolio sampling (top-N by rank metric) |
| `USE_PCT_MODE` | `True` | Use % compounding for challenge stats instead of fixed R |
| `ACCOUNT_BALANCE` | `1000.0` | Starting balance for % mode |
| `RISK_PCT_PER_R` | `0.1` | % of equity risked per 1R trade (compounded) |

## Understanding the Tests

The script builds a set of **filter pipelines** (called "tests") and measures how often strategies passing each filter remain profitable in the live proxy windows.

- **Base quality gate** (`>100 trades + dedup`): strategies with enough trades and no duplicate signatures. No robustness data needed. Not eligible for the main export.
- **Robustness funnels** (`IS + IS+TAG + OOS`, `IS + IS+TAG + OOS + OOS+TAG`): strategies must pass both the base IS filter and the robustness-tagged version, in both IS and OOS. These are the eligible export tests.

The tighter the funnel (more conditions required), the smaller the surviving set — but typically a higher live-proxy pass rate.

## Output

The exported Excel file (`selected_pipeline_test_N_strategies.xlsx`) contains:

| Sheet | Contents |
|-------|----------|
| `All_Parsed_From_TXT` | All strategy metrics parsed from `.txt` files |
| `TradeList_ExclLive` | Trade-list stats excluding live proxy windows |
| `TradeList_AllWindows` | Trade-list stats across all windows |
| `TradeList_LiveOnly` | Trade-list stats for live proxy windows only |
| `Portfolios` | Sampled portfolio combinations with live stats |
