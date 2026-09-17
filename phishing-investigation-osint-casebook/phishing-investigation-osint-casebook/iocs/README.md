# IOCs

Consolidated indicators from all five cases. **All values are synthetic** (RFC 2606/5737/5398
reserved ranges) and cannot resolve to real infrastructure.

| File | Contents |
|---|---|
| `domains.csv` | Domain indicators |
| `ips.csv` | IPv4 indicators |
| `urls.csv` | URL indicators (victim tokens stripped) |
| `emails.csv` | Email-address indicators |
| `hashes.csv` | File/kit hashes (placeholders) |
| `certificates.csv` | Certificate fingerprints |
| `iocs.json` | Combined export, structured for conversion to MISP attributes or STIX 2.1 |

## Reading the `recommended_action` column

Action is **not** derived from confidence. It is derived from confidence *and* collateral
cost. Two rows make the point:

- `198.51.100.44` — High confidence campaign infrastructure, `status=shared_infrastructure`,
  `recommended_action=monitor`. It hosts 900+ unrelated sites. Blocking it is high-confidence
  and operationally reckless.
- `no-reply@meridian-fin.example` — High confidence indicator, `recommended_action=enrich_only`.
  It is the **victim organisation's own address**, spoofed. Blocking it would block internal mail.

An IOC feed that ignores this distinction produces outages, and an analyst who exports "all
observed indicators" into a blocklist will cause one.

## Ageing

Every row carries `last_observed`. Domains and IPs age to `monitor` after 30 days without
observation; URLs after 7. Retired infrastructure is retained for retrospective hunting but
removed from enforcement — a lapsed phishing domain re-registered by an innocent party
becomes a permanent false positive if left in a blocklist. See
[ioc-management.md](../methodology/ioc-management.md).

## Defanging

`indicator` is fanged for machine parsing; `indicator_defanged` is the value used in all
prose and reports. Consumers should parse the fanged column deliberately, not accidentally.
