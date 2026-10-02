# Demo guide

Runs end to end with no Slack, Microsoft, Azure or Shopify account. Only n8n is needed.

## Setup (5 minutes)

1. Start n8n locally: `docker run -it --rm -p 5678:5678 n8nio/n8n`
2. Import `workflows/01-intake-diagnosis.json`, `03-error-handler.json` and `04-health-watchdog.json`.
3. Create a data table named `support_cases` with the columns in `config/support_cases.columns.txt`.
4. Open the **Data Table: Upsert Case** node once and refresh its columns.
5. Leave `test_mode` and `mock_classify` set to `true` in each Config node.

## Run 1: intake

Execute **Schedule: Every 10 min** manually. Twelve mock emails flow through order lookup, attachment filter, photo analysis, classification, rules and drafting. Open **Code: TEST MODE - Show Card** to read each card as text.

| # | Case | Expected compensation |
|---|---|---|
| 01 | Cracked, no cause, no photo | none, asks for evidence |
| 02 | Danish, cause stated | 30% |
| 03 | French, marketplace order, photo | 100% |
| 04 | No signal | none, escalate |
| 05 | Reply on an open case | 30% |
| 06 | Rude, no evidence | 0%, polite decline |
| 07 | Detached part, no tray in frame | none, manual review |
| 08 | Receipt only | none, asks for evidence |
| 09 | Russian, no photo | none |
| 10 | German, two units | 30% |
| 11 | Video attachment only | none, warning on card |
| 12 | Empty email | none, escalate |

## Run 2: watchdog

Execute **Manual Trigger (demo)**. Seven mock cases are checked. Six are flagged and one is healthy.

| Case | Reason |
|---|---|
| CS-DEMO-0001 | missing_slack |
| CS-DEMO-0002 | missing_sharepoint |
| CS-DEMO-0003 | ai_failed |
| CS-DEMO-0004 | sla_breach |
| CS-DEMO-0005 | stale_ruleset |
| CS-DEMO-0007 | sla_breach (retries exhausted) |

**Code: Build Run Report** lists the planned action per case. The junk folder and downtime branches also fire. No action runs because the live nodes are disabled.

## Run 3: error handler

Execute **Manual Trigger (demo)**. Two sample errors are classified. A rate limit is logged only. An auth failure builds an ops alert.

## Not demoable yet

The Slack actions workflow (approve, edit, park, forward) is a scaffold. It needs a public webhook and a Slack app.