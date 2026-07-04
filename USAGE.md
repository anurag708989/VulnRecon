# Usage Guide

## Basic Scan

```bash
python3 vulnrecon.py -t example.com
```

Runs subdomain discovery + Wayback analysis + JS discovery with default thresholds (priority 0, confidence 0.0 — i.e. shows everything).

## Full-Feature Scan

```bash
python3 vulnrecon.py -t example.com \
  --shodan YOUR_SHODAN_KEY \
  --telegram BOT_TOKEN:CHAT_ID \
  --priority 15 \
  --live-preview
```

## Bulk / Multi-Target Scan

`domains.txt`:
```
example.com
target-two.com
# comments and blank lines are ignored
target-three.com
```

```bash
python3 vulnrecon.py --targets-file domains.txt --priority 50 --confidence 0.7
```

Each target gets its own timestamped output directory — results are never mixed between targets.

## High-Signal / Bug Bounty Triage Mode

```bash
python3 vulnrecon.py -t example.com --priority 80 --confidence 0.8 --samples 25
```

Surfaces only admin panels, IDOR-suspect URLs, `.env`/`.sql` exposures, and high-confidence API endpoints.

## Silent / CI-Friendly Mode

```bash
python3 vulnrecon.py --targets-file domains.txt --no-output --quiet -o results/
```

Suppresses interactive tables — useful when scheduling scans via cron/CI and just consuming the JSON output afterward.

## Feeding Results Into Nuclei / FFUF

```bash
TARGET_DIR=$(ls -d vulnrecon_results/example_com_* | tail -1)

nuclei -l "$TARGET_DIR/nuclei/nuclei_targets.txt" -severity critical,high,medium -o nuclei_findings.txt

ffuf -w "$TARGET_DIR/ffuf/ffuf_targets.txt:FUZZ" -u FUZZ -mc 200,204,301,302,307,401,403
```

## Interpreting Priority/Confidence Output

| Score band | Suggested action |
|---|---|
| Priority ≥ 90 | Investigate manually immediately (potential critical exposure) |
| Priority 70–89 | Queue for Nuclei template matching / manual API testing |
| Priority 50–69 | Low-priority backlog, review if time permits |
| Priority < 50 | Generally noise — safe to ignore for triage purposes |

## Detailed In-Tool Help

```bash
python3 vulnrecon.py --detailed-help
```

## Common Recipes

**Only look for exposed data files:**
```bash
python3 vulnrecon.py -t example.com --priority 90
```

**Fast pass across a huge scope list, revisit high scorers later:**
```bash
python3 vulnrecon.py --targets-file huge_scope.txt --fast --priority 60 --no-output
```

**Deep single-target dive with everything on:**
```bash
python3 vulnrecon.py -t example.com --deep --shodan $SHODAN_KEY --telegram $BOT:$CHAT --live-preview -v
```
