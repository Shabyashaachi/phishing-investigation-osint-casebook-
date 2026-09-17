# Case 003 — Phishing Domain Investigation

**Focus:** RDAP interpretation, DNS record analysis, Certificate Transparency as a
*discovery* source, and the limitations of every popular "suspicious domain" indicator.

**Subject:** `meridian-verify[.]example`, the landing domain from [Case 002](../case-002-phishing-url-investigation/).

**What this case demonstrates**

- A worked argument for why **newly registered**, **privacy-protected**, **cheap registrar**
  and **low TTL** are weak indicators individually, with the discriminating power of each
  stated explicitly
- Reading the MX record as the substantive finding — and testing the "hosting default
  template" alternative explanation rather than mentioning it
- Using a CT **string** search to discover a sibling domain never seen in any message, while
  excluding unrelated legitimate organisations that matched the same substring
- Rejecting the strongest-looking link in the case (900+ domains on the same IP) and
  retaining a three-minute timing correlation instead

**Key judgements.** Domain staged specifically for this campaign (High). Independent mail-
receiving capability, likely for exfiltration (Medium). Sibling domain probably related
(Medium-High). Co-hosted domains explicitly **not** related (High, negative).

**Files:** [`investigation.md`](investigation.md) · [`evidence/`](evidence/)

All data is synthetic and defanged. See [DISCLAIMER](../../DISCLAIMER.md).
