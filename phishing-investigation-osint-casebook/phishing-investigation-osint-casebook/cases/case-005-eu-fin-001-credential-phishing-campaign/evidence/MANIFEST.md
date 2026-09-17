# Evidence Manifest — CASE-005 / EU-FIN-001

All artefacts are **synthetic**. Hashes are placeholders marked `SYNTHETIC`; a real case
would carry genuine SHA-256 digests computed at acquisition.

| File | SHA-256 | Acquired (UTC) | Method | Custodian | Class |
|---|---|---|---|---|---|
| `eu-fin-001-headers.txt` | `SYNTHETIC-005-HDR` | 2026-02-11T09:07Z | Gateway export, header extraction | Analyst | Derived from original |
| `eu-fin-001-dns.txt` | `SYNTHETIC-005-DNS` | 2026-02-11T09:14Z | `dig`, authoritative | Analyst | Derived |
| `eu-fin-001-redirect-chain.txt` | `SYNTHETIC-005-RDR` | 2026-02-11T09:41Z | Sandbox retrieval, substituted token | Analyst | Derived |
| `eu-fin-001-certificates.txt` | `SYNTHETIC-005-CRT` | 2026-02-11T10:02Z | CT log search | Analyst | Derived |

## Handling record

- Recipient identities pseudonymised (`user-1182`) at acquisition; no real personal data held.
- Victim-unique URL token substituted with `CONTROL` before any sandbox submission and
  redacted in all published material. The original was never submitted to a third-party
  service.
- No credentials were submitted to the phishing infrastructure at any point.
- All external collection was passive (RDAP, DNS, CT) or sandboxed. No unauthorised access
  to any system was attempted.
- The original `.eml` is not published: it would contain recipient-identifying material in a
  real case. Only the pseudonymised header extract is included.
