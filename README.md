# Cedar Bridge Case Study — CB-003
## Phishing Investigation and Cloud-Access Triage

This repository contains a fictional, synthetic phishing and cloud-activity investigation written from a junior security analyst perspective.

### Report and supporting files
- [Week 3 phishing investigation report](report/week-3-phishing-investigation-report.md)
- [Normalized event timeline (WAT)](analysis/timeline.csv)
- [Alert triage](analysis/alert-triage.md)
- [Disclaimer](DISCLAIMER.md)

### Case snapshot
- **Case:** CB-003
- **Evidence date:** 19 October 2026
- **Priority:** High, pending incident-lead review
- **Primary account:** Amara
- **Endpoint:** CB-LT-07
- **Cloud session:** S-77
- **Key concern:** A suspicious email and endpoint/browser events are followed by a successful sign-in and reads from a sensitive cloud storage location.

### Important distinction
The synthetic records support a successful object read through session S-77. They do not, on their own, establish who controlled the session or prove the email caused the cloud activity. A policy-change attempt was denied; no successful bucket-policy change or public exposure is established.

No live links were visited and no attachment was executed.
