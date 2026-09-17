# Implementation Roadmap

The repository as published is complete and self-contained. This is the plan for taking it
from "published" to "the version that closes the gaps a hiring manager would name" — see
[portfolio/hiring-manager-review.md](portfolio/hiring-manager-review.md) for where these items
come from.

Ordered by return on effort, not by size.

---

## Phase A — Publish (done)

Architecture, seventeen-stage methodology, five cases, evidence artefacts, IOC exports,
templates, SOC integration, portfolio materials, ethical framework.

**Before pushing:**

- [ ] Replace `[Your Name]`, `[Author]`, `[link]` and `[email]` placeholders throughout
      (`grep -rn "\[Your Name\]\|\[Author\]\|\[link\]\|\[email\]" .`)
- [ ] Set the repository description and topics: `threat-intelligence`, `osint`, `phishing`,
      `cti`, `soc`, `dfir`, `mitre-attack`
- [ ] Confirm every Mermaid block renders on GitHub (they render differently from local
      previews)
- [ ] Re-read `DISCLAIMER.md` and confirm the synthetic-data statement is accurate for
      everything you have added
- [ ] Check no fanged indicator has crept into prose: `grep -rnE "https?://[a-z0-9.-]+\.(example|invalid|test)" --include="*.md" .`

---

## Phase B — Close the biggest gap (highest priority, ~1 week)

**Case 006 — Inconclusive investigation.**

The most valuable thing missing from this repository is a case that does not resolve. Build
one deliberately: a forwarded copy with no original, so the routing chain is gone; a domain
that stopped resolving before investigation began; an empty passive-DNS result; two sources
that disagree.

Terminate in an honest **"insufficient evidence to assess"** with a stated collection plan and
a list of what would have to become available. Demonstrating comfort with that outcome is
rarer, and more employable, than producing another clean conclusion.

---

## Phase C — Close the tooling gap (~2 days)

**Appendix: real-tool walkthrough.**

Take a phishing domain from a **published, already-remediated** advisory — one that is dead
and already publicly reported — and run the identical methodology against it with real
`dig`, RDAP and crt.sh output. Passive sources only.

This is safe, lawful and closes the "has this person actually run anything" question
completely. Keep it short; it is evidence, not another case study.

---

## Phase D — Sharing output (~half a day)

- [ ] Export the EU-FIN-001 cluster as a MISP event JSON (`iocs/misp-event-EU-FIN-001.json`)
- [ ] Add a STIX 2.1 bundle
- [ ] Note in `iocs/README.md` that both are synthetic and must not be imported into
      production

Sector CSIRT and ISAC participation is routine in European CTI roles. Showing the export
format is concrete and cheap.

---

## Phase E — Regulatory context (~half a day)

Add `methodology/regulatory-context.md` covering, for the EU financial-services scenario:

- When a phishing incident becomes a GDPR Art. 33 notification question, and the 72-hour clock
- DORA incident-reporting considerations for financial entities
- NIS2 applicability
- Why the CTI analyst's job is to surface the trigger, not to make the legal determination

Keep it factual and short, and state plainly that it is not legal advice.

---

## Phase F — Polish (~1 day)

- [ ] Render the flagship technical report and executive brief to PDF
- [ ] Take one Sigma rule through a documented tuning iteration — what it fired on, what was
      excluded, the residual false-positive cost
- [ ] Add a one-page triage decision framework: what makes a report worth an hour rather than
      four minutes
- [ ] Review the entry points (root README, flagship README, executive brief) and make sure
      further additions have not diluted them

---

## Phase G — Optional automation (last, and genuinely optional)

Only after everything above. See [tools/README.md](tools/README.md) for the full reasoning on
what should and should not be automated.

Priority order: `check_tenancy.py` first — it is the check that prevents outages and should
never depend on an analyst remembering — then `validate_iocs.py`, `enrich_indicator.py`,
`ct_monitor.py`, `extract_iocs.py`.

**Constraint that should not be relaxed:** the repository must remain fully useful to a
reviewer who never runs a single script. If automation ever becomes the reason to look at this
work, it has stopped being a CTI portfolio and become a coding one.

---

## Ongoing

- [ ] Invite and act on peer review. One issue thread containing a substantive disagreement
      about a confidence level, and a documented revision in response, is worth more than an
      additional case study. It is also the only evidence available that the confidence labels
      are real rather than decorative.
- [ ] Version key judgements when they change, rather than editing silently. An analytic record
      that never changes its mind is a record nobody has checked.
