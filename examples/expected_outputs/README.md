# Expected Outputs

When you run the script against the `sample_strategy_output` folder (with `ENTRY_DRIFT = True` data present), it produces:

```
selected_pipeline_test_N_strategies.xlsx
```

This Excel workbook contains the following sheets:

| Sheet | Contents |
|-------|----------|
| `All_Parsed_From_TXT` | All strategy metrics parsed from `.txt` files |
| `TradeList_ExclLive` | Trade-list stats excluding live proxy windows |
| `TradeList_AllWindows` | Trade-list stats across all windows |
| `TradeList_LiveOnly` | Trade-list stats for live proxy windows only |
| `Portfolios` | Sampled portfolio combinations with live-proxy stats |

> **Note:** The sample data in `../sample_strategy_output/` contains only 2 strategies.
> In a real run you would have hundreds or thousands of strategies and the export would
> be meaningfully populated. The sample data is only meant to verify the script runs
> end-to-end without errors.
