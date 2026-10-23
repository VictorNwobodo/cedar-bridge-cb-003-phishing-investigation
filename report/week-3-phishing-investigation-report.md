# Week 3 Phishing Investigation Report — CB-003

**Evidence date:** 19 October 2026  
**Priority:** High for investigation and evidence preservation  
**Timezone:** WAT (UTC+1); original UTC timestamps are retained where supplied  
**Case type:** Fictional phishing and cloud-access investigation

## 1. Executive summary

The supplied evidence describes a suspicious email impersonating Cedar Bridge Finance, an endpoint save of an HTML file named to resemble a PDF, a browser connection to a lookalike domain, authentication failures followed by a successful password sign-in, and cloud-storage activity under session ID `S-77`.

The strongest technical join is the session identifier: I04 records a successful sign-in that created S-77, and C01–C03 record subsequent cloud actions under S-77. C01 and C02 show successful listing and retrieval of a finance invoice object. C03 shows an attempted bucket-policy change that was denied for insufficient permission.

**The evidence supports a high-priority investigation into suspicious authentication and cloud access. It does not independently prove who controlled S-77, prove that the email caused the sign-in, establish malware execution, or show a successful bucket-policy change or public exposure.**

## 2. Scope and evidence reviewed

The report uses the supplied email evidence E01, endpoint events P01–P02, identity events I02–I04, and cloud audit events C01–C03. All timestamps are normalized to WAT (UTC+1) where the source timestamp is UTC.

No live links were opened and the attachment was not executed. The supplied evidence does not include the full original email headers, attachment bytes, complete endpoint telemetry, full identity-provider logs, or all cloud audit records. The assessment therefore distinguishes logged actions from unverified explanations.

## 3. Email analysis — E01

| Field | Supplied evidence |
|---|---|
| Received | 2026-10-19 08:04 UTC / 09:04 WAT |
| Display name | Cedar Bridge Finance |
| From | `support@invoice-review.example` |
| Reply-To | `review@invoice-review.example` |
| Gateway | `mx.cedarbridge.example` |
| Authentication | SPF/DKIM/DMARC pass for `invoice-review.example` |
| Displayed portal | `https://portal.cedarbridge.example` |
| Actual link (defanged) | `hxxps://portal.cedarbridge.example.invoice-review.example/verify` |
| Attachment | `invoice-notice.pdf.html` |
| Declared content type | `text/html` |
| Size | 2,048 bytes |

The sender domain differs from the Cedar Bridge domain despite the display name. The actual link is hosted under `invoice-review.example`; the Cedar Bridge-looking hostname is only a subdomain label. The attachment extension and declared HTML content type are also suspicious in an invoice-themed message.

SPF/DKIM/DMARC passing for `invoice-review.example` indicates authentication for that domain; it does **not** establish that Cedar Bridge sent or approved the message. The attachment bytes were not supplied, so its content or behavior cannot be confirmed from the filename alone.

## 4. Endpoint and identity events

| Record | WAT timestamp | Host/account | Observation | Interpretation |
|---|---|---|---|---|
| P01 | 09:09 | CB-LT-07 / Amara | `mailclient.exe` saved `Downloads/invoice-notice.pdf.html` | File save observed; execution not established |
| P02 | 09:10 | CB-LT-07 / browser | Connection to `portal.cedarbridge.example.invoice-review.example:443` | Browser connection observed; encrypted page content not supplied |
| I02 | 09:12 | Amara | Failed sign-in from `198.51.100.77` | Failed authentication |
| I03 | 09:12:20 | Amara | Second failed sign-in from `198.51.100.77` | Failed authentication |
| I04 | 09:13:10 | Amara | Successful password sign-in from `198.51.100.77`; session S-77 created | Successful sign-in; actor identity remains unverified |

The timing and lookalike domain are concerning, but the supplied records do not show whether the recipient entered credentials, whether the saved HTML file was opened, or whether code executed. The two failed sign-ins followed by success do not by themselves prove credential theft.

## 5. Cloud audit analysis

| Record | WAT timestamp | Session | Action | Resource | Outcome |
|---|---|---|---|---|---|
| C01 | 09:14 | S-77 | `ListObjects` | `cb-finance` | Success |
| C02 | 09:15 | S-77 | `GetObject` | `cb-finance/invoices-2026.csv` | Success; 24,576 bytes returned |
| C03 | 09:17 | S-77 | `PutBucketPolicy` | `cb-finance` | Denied: insufficient permission |

S-77 is the strongest correlation key because I04 creates it and C01–C03 use it. The logs support that the session successfully listed the finance bucket and retrieved `invoices-2026.csv`. The 24,576-byte return indicates a successful object response in the supplied record, not how the data was subsequently used or whether it left an authorized environment.

C03 was denied. There is no evidence in these records that the bucket policy was successfully changed or that the bucket became public.

The supplied role description says `invoice_reader` permits `ListObjects` and `GetObject` on `cb-finance`, but not `PutBucketPolicy`. This is consistent with C01/C02 succeeding and C03 being denied. Whether this activity was legitimate still requires confirmation from the account owner and cloud audit context.

## 6. Alert queue and supporting findings

### A01 — Low, likely benign if approved
A01 relates to activity that remains benign only if it stayed within the authorized, non-destructive staging tests under change CHG-301. Validate the change record, scope, actor, and timestamps before closing it.

### A02 — High
External sign-in failures followed by a successful sign-in and sensitive object read. Key records: I02, I03, I04, C01, C02, C03. The successful object read under S-77 makes this the primary escalation item.

### A03 — Medium
W02 records a script-like/XSS pattern that was blocked with HTTP 403. The supplied record does not show successful exploitation.

### A04 — Medium, pending authentication review
W03 shows sequential invoice identifiers with HTTP 401 and 402 responses and an HTTP 200 response, but the authenticated user, object owner, and response body are not supplied. These status codes alone do not prove data disclosure or an insecure direct object reference.

### Vulnerability and configuration observations
- **V01:** A banner/version match suggests a potentially outdated component, but the version and exposure require validation against inventory, build revision, vendor advisories, patch history, and backport status. No exploitation is established.
- **V02:** A missing security header was observed on staging; treat as a low-priority hardening issue unless broader context changes its impact.
- **V03:** Public bucket access was not detected in the supplied evidence. Do not report public exposure as a finding.

## 7. Joined timeline

See [timeline.csv](../analysis/timeline.csv). Key sequence:
1. 09:04 — E01 received.
2. 09:09 — attachment-like HTML file saved to the endpoint.
3. 09:10 — browser connected to the lookalike host over port 443.
4. 09:12–09:13 — failed sign-ins followed by a successful password sign-in that created S-77.
5. 09:14 — S-77 successfully listed objects in `cb-finance`.
6. 09:15 — S-77 successfully retrieved `invoices-2026.csv`, with 24,576 bytes returned.
7. 09:17 — S-77 attempted `PutBucketPolicy`; the action was denied.

The ordering is consistent with a possible phishing-related compromise, but time correlation alone is not proof of causation. Validate with original email headers, endpoint telemetry, and identity-provider/session evidence.

## 8. Impact and confidence

### Established by supplied records
- A suspicious email used a Cedar Bridge Finance display name while sending from `invoice-review.example`.
- The endpoint saved an HTML file with a PDF-like filename and connected to a lookalike hostname.
- A successful password sign-in created S-77.
- S-77 successfully listed a finance bucket and retrieved an invoice CSV object.
- A bucket-policy modification attempt was denied.

### Not established
- Whether Amara submitted credentials or opened/executed the attachment.
- Whether S-77 was controlled by an unauthorized actor.
- Whether the successful sign-in and email were causally related.
- Whether additional objects were accessed or data was misused.
- Whether a successful permission change or public exposure occurred.
- Whether the potentially outdated component was exploitable.

**Overall confidence:** High that the logged sequence warrants investigation; moderate that the email is phishing based on the visible indicators; undetermined whether the cloud session was unauthorized until identity and business context are checked.

## 9. Escalation ticket

### CB-003 — High priority
**Title:** Suspicious email followed by successful identity session and finance-object retrieval

**Priority rationale:** I04 creates session S-77 after two failed sign-ins, and C01/C02 show successful access to the finance bucket and retrieval of a sensitive invoice CSV. The attempted policy change in C03 was denied, but the preceding successful read requires validation.

**Affected account/device/data:** Amara; CB-LT-07; cloud bucket `cb-finance`; object `invoices-2026.csv` (24,576 bytes returned); session S-77.

**Key record IDs:** E01, P01, P02, I02, I03, I04, C01, C02, C03.

**Immediate evidence-preservation requests:**
1. Preserve E01 in original form with full headers, gateway trace, URL rewrite/sandbox results, and attachment hash/bytes.
2. Preserve CB-LT-07 endpoint telemetry, browser history/cache where policy permits, file creation metadata, process execution history, DNS and network logs.
3. Preserve identity-provider authentication details, MFA/challenge results, session issuance metadata, source/network context, and session revocation history.
4. Preserve complete cloud audit logs and object-access records for `cb-finance`, including all S-77 actions and any other object reads.
5. Record collection times, source systems, collector, hashes, and chain-of-custody information.

**Ownership:** Incident lead coordinates triage and impact assessment. The cloud administrator should perform session revocation, credential changes, and storage-policy changes only through the approved containment process.

**Questions to resolve:**
- Did Amara receive, open, or interact with E01?
- Who or what controlled S-77, and was the sign-in expected?
- Was MFA required, challenged, or satisfied?
- Were any other objects read or downloaded?
- Was any bucket policy or permission successfully changed in records outside this sample?
- Was the access part of an approved business operation?

**Escalation criteria:** Evidence of credential submission, malicious process execution, unauthorized session control, additional sensitive-object reads, or a successful permission change increases severity and supports containment. Evidence that S-77 belongs to a verified, approved business action may reduce concern but must be documented.

## 10. Conclusion

CB-003 should remain a High-priority investigation until the identity/session and business context are verified. The evidence establishes successful cloud object access under S-77, not the identity of the actor or the cause of the session. Preserve evidence first, validate the session, assess the full scope of object access, and carry out containment only through the approved incident-response process.
