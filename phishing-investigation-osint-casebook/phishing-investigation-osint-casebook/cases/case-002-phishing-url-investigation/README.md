# Case 002 — Phishing URL Investigation

**Focus:** URL anatomy, redirect-chain analysis, hosting context, certificate as a timeline
source, and why reputation data contributed nothing.

**Input:** the URL extracted in [Case 001](../case-001-email-header-investigation/).

**What this case demonstrates**

- Identifying the registrable domain before reading anything else — brand strings in
  subdomains and paths are free and evidentially worthless
- Handling a **victim-unique token** safely: substituted before sandbox submission, never
  published
- Distinguishing adversary-owned infrastructure from **abused legitimate infrastructure**
  (the redirect hop), and treating them differently
- Detecting cloaking, and concluding that automated reputation verdicts on this URL are
  therefore unreliable — including the clean ones
- Producing an IP indicator with an explicit *do not block* action because of co-tenancy

**Key judgement.** The URL leads via an abused redirector to a personalised credential-
collection page. Confidence: High.

**Files:** [`investigation.md`](investigation.md) · [`evidence/`](evidence/)

All data is synthetic and defanged. See [DISCLAIMER](../../DISCLAIMER.md).
