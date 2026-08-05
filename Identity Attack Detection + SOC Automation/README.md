# Blue Team Portfolio: Identity Attack Detection → SOC Automation

An end-to-end detection engineering + SOC automation portfolio, built in an
isolated VirtualBox Active Directory lab.

```
Generate attacks  →  Detect (Splunk SPL)  →  Reduce false positives  →  Automate triage
   (Kali)              (Project 1)              (Project 1)              (Project 2)
```

## Project 1 — Identity Attack Detection & False Positive Reduction
See [`docs/project1_identity_attack_detection.md`](docs/project1_identity_attack_detection.md)

Detects password spraying, brute force, account lockout abuse, Kerberoasting,
and AS-REP roasting against a lab Active Directory domain, with documented
initial rules, identified false positives, and tuned final SPL queries.

## Project 2 — SOC Automation
See [`docs/project2_soc_automation.md`](docs/project2_soc_automation.md) and
[`soc_automation/`](soc_automation/)

When a Project 1 detection fires, this Flask-based pipeline automatically:
1. Enriches the alert (account privilege, source reputation, asset criticality)
2. Calculates a 0–100 risk score
3. Generates a Markdown incident summary
4. Posts it to Slack

## Repo structure

```
portfolio/
├── README.md
├── docs/
│   ├── project1_identity_attack_detection.md
│   └── project2_soc_automation.md
└── soc_automation/
    ├── app.py                # Flask webhook receiver
    ├── enrichment.py
    ├── risk_scoring.py
    ├── incident_summary.py
    ├── notifier.py
    ├── requirements.txt
    └── data/
        ├── mock_ad.json
        ├── asset_criticality.json
        └── ip_watchlist.json
```

## Quick start (SOC automation)

```bash
cd soc_automation
pip install -r requirements.txt
export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/..."
python app.py
```

Then point a Splunk "Webhook" alert action (on any Project 1 detection) at
`http://<this-host>:5000/webhook/splunk-alert`.
