# IOC Management

## Schema

Every indicator in [`iocs/`](../iocs/) carries the following fields.

| Field | Values | Notes |
|---|---|---|
| `ioc_id` | `IOC-<case>-<n>` | Stable reference from reports |
| `type` | `domain`, `url`, `ipv4`, `ipv6`, `email`, `sha256`, `md5`, `cert_sha256`, `asn` | One type per row |
| `indicator` | fanged value | Machine-readable field |
| `indicator_defanged` | defanged value | Used in all prose and reports |
| `first_observed` | UTC ISO 8601 | With source of the timestamp |
| `last_observed` | UTC ISO 8601 | Drives ageing |
| `source` | source name + reliability grade | e.g. `crt.sh (A)` |
| `description` | one line | What role the indicator played |
| `confidence` | `low` / `medium` / `high` | Confidence that the indicator is campaign-related |
| `status` | `active`, `aged`, `retired`, `false_positive`, `shared_infrastructure` | Lifecycle state |
| `related_case` | case ID | |
| `recommended_action` | `block`, `monitor`, `alert`, `enrich_only`, `no_action` | |
| `tlp` | `CLEAR`, `GREEN`, `AMBER`, `AMBER+STRICT`, `RED` | All published items here are `TLP:CLEAR` (synthetic) |

## Why `recommended_action` is not derived from `confidence`

A high-confidence indicator can still be unsafe to block. The clearest example in this
casebook: `198.51.100.44` is high-confidence campaign infrastructure *and* shared hosting
with many unrelated tenants. Blocking it is high-confidence and operationally wrong.

Action therefore depends on **confidence × collateral cost**:

| | Low collateral | High collateral |
|---|---|---|
| **High confidence** | Block | Alert / monitor with context |
| **Medium confidence** | Alert | Monitor |
| **Low confidence** | Monitor | Enrich only |

Indicators classed `shared_infrastructure` are never marked `block`, regardless of
confidence. That distinction is the difference between intelligence a SOC can use and
intelligence that causes an outage.

## Indicator durability

Indicators decay at very different rates. Prioritising detection by durability is more useful
than prioritising by confidence:

| Class | Typical lifetime | Defensive value |
|---|---|---|
| URL with victim token | Hours | Retrospective hunting only |
| Domain | Days to weeks | Blocking, until takedown |
| IP (dedicated) | Days | Blocking, with care |
| IP (shared/CDN) | Irrelevant | Never block |
| Certificate fingerprint | Certificate lifetime | Strong pivot, weak block |
| Sender address | Hours | Low |
| Kit file hash | Weeks to months | Good — kits are reused |
| TTP / behavioural pattern | Months to years | **Highest** — survives all of the above |

This is why every case in the casebook ends with behavioural detections and hunt questions,
not just a block list.

## Lifecycle

```
new → active → aged → retired
         ↓
   false_positive / shared_infrastructure
```

- **active** — currently supported by evidence; in enforcement or monitoring.
- **aged** — no observation in 30 days (domains/IPs) or 7 days (URLs). Demoted from block to
  monitor; kept for retrospective hunting.
- **retired** — infrastructure confirmed taken down, sinkholed, or re-registered by a
  legitimate party. **Re-registration matters**: an expired phishing domain bought by an
  innocent third party becomes a false positive that will generate tickets forever if left in
  a blocklist.
- **false_positive** — evidence no longer supports the association; retained with a reason so
  the same mistake is not re-made.

Every state change is dated and reasoned in the case file.

## Defanging

Human-readable output is always defanged (`hxxp://`, `[.]`, `[@]`). Machine-readable CSV/JSON
retains both fanged and defanged columns so that a consumer parses live values only when they
intend to. The `iocs.json` export is structured for straightforward conversion to MISP
attributes or STIX 2.1 indicator objects; field names were chosen with that mapping in mind.

## What is deliberately not published

- Any indicator that could identify a real individual.
- Live malware samples (referenced by hash only).
- Victim-unique URL tokens.
- Internal hostnames or infrastructure of any real organisation.
