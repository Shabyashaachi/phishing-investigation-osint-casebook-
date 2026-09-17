# Tools

**The casebook is complete and useful without any automation.** The analytical content is the
deliverable; scripts would only reduce typing.

That is a deliberate position. A repository whose value depends on its code is a coding
portfolio. This one should be readable, and assessable, by someone who never runs anything.

## What automation would sensibly add

If extended, in order of actual usefulness:

| Script | Purpose | Why it is worth automating |
|---|---|---|
| `extract_iocs.py` | Parse a `.eml` into a structured first-pass indicator list | Manual extraction is error-prone and boring; the *interpretation* stays manual |
| `enrich_indicator.py` | RDAP + DNS + CT lookup for a domain, with UTC timestamps | Consistent, timestamped collection; reduces transcription errors |
| `check_tenancy.py` | Co-hosting density for an IP before it reaches a blocklist | **The highest-value script.** This is the check that prevents outages, and it should never depend on an analyst remembering |
| `ct_monitor.py` | Poll CT logs for brand-lookalike issuance | Both waves in this casebook were visible in CT days before delivery |
| `validate_iocs.py` | Schema and defanging check on the CSV exports | Catches a fanged value in a published report before a reader clicks it |

## What automation should never do

- **Assign confidence.** Confidence is an analytic judgement about an evidential basis. A
  script can count corroborating sources; it cannot assess their independence.
- **Decide `recommended_action`.** That depends on collateral cost, which requires context the
  script does not have.
- **Draw correlation edges.** Automated pivoting produces exactly the over-connected graphs
  this casebook argues against. A tool can surface candidate links; an analyst decides which
  survive.
- **Interact with live adversary infrastructure** without a deliberate, logged decision.

## If you build these

Rate-limit politely, cache aggressively, timestamp everything in UTC, record the exact query
alongside the result, and default to passive sources. Dependencies: `requirements.txt`.
