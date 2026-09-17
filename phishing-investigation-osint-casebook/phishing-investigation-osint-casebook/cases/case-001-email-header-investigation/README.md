# Case 001 — Email Header Investigation

**Focus:** full header-chain analysis, SPF/DKIM/DMARC interpretation, routing anomalies,
first-pass IOC extraction.

**Scenario.** A Finance user at *Meridian Financial Group* (fictional EU financial-services
firm) reports an email claiming their mailbox storage is exceeded. Three further recipients
received the same message within ninety seconds.

**What this case demonstrates**

- Reading a `Received` chain bottom-up and identifying the first trusted hop
- The difference between SPF (envelope) and DMARC (alignment with the visible `From`)
- Why a `From`/`Reply-To`/`Return-Path` mismatch is a weak indicator in isolation
- Why the spoofed sender address must **not** go into a blocklist
- Separating triage priority from analytic verdict

**Files**

| File | Contents |
|---|---|
| [`investigation.md`](investigation.md) | Full investigation and reasoning |
| [`evidence/case-001-headers.txt`](evidence/case-001-headers.txt) | Synthetic raw header block |
| [`evidence/case-001-dns.txt`](evidence/case-001-dns.txt) | SPF/DMARC lookup record |
| [`evidence/MANIFEST.md`](evidence/MANIFEST.md) | Evidence manifest |

**Key judgement.** Credential phishing impersonating internal IT support, sent from
infrastructure unauthorised for the displayed domain. Confidence: High.

All data is synthetic and defanged. See [DISCLAIMER](../../DISCLAIMER.md).
