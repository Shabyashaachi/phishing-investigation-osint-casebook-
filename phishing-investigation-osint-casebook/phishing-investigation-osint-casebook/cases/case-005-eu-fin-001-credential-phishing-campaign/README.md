# Case 005 — EU-FIN-001 · Credential Phishing Campaign (Flagship)

**Full investigation of a targeted, MFA-defeating credential phishing campaign against a
fictional EU financial-services organisation.**

| Document | Audience | Purpose |
|---|---|---|
| [`investigation.md`](investigation.md) | Analyst / reviewer | Complete working record: reasoning, alternative explanations, rejected hypotheses, self-critique |
| [`report-technical.md`](report-technical.md) | SOC, detection engineering, mail/IAM teams | Formal CTI report |
| [`executive-brief.md`](executive-brief.md) | CISO, IT leadership | One page, no jargon |
| [`evidence/`](evidence/) | — | Synthetic artefacts and manifest |

## Why this case is the flagship

It is the only case where the landing infrastructure turns out to be something other than
what the first four cases would have predicted. The page was an **adversary-in-the-middle
reverse proxy**, not a static credential clone — which means MFA was relayed, session tokens
were the target, and password resets alone would have left the compromise live.

That finding came from reading the sandbox network capture rather than accepting the
screenshot. It is recorded in the case as an anchoring near-miss.

## What it demonstrates

- Interpreting `spf=pass` + `dkim=pass` + `dmarc=fail` correctly — and explaining to a
  non-specialist why the first two do not mean "genuine"
- Identifying that the **root cause of delivery was an internal gateway allow-list**, not the
  attacker's sophistication
- Distinguishing AiTM proxying from static cloning on network evidence
- Cross-wave correlation using four *independent* weak links, while rejecting the
  strongest-looking one (shared mass-hosting IP)
- ATT&CK mapping constrained by evidence, with an explicit "not mapped" section
- IOC actions that diverge from IOC confidence, to avoid operationally damaging blocks
- Two-audience reporting where the executive version answers different questions rather than
  being a shorter technical report
- Recommending the one control that removes the technique, and saying plainly that the rest
  only raise cost

## Key judgements

| | Judgement | Confidence |
|---|---|---|
| KJ-1 | Targeted credential phishing impersonating Meridian SSO | High |
| KJ-2 | AiTM reverse proxy capable of session-token capture | High |
| KJ-3 | Deliberate targeting of payment-authorising roles | Medium-High |
| KJ-4 | Same operator as the January wave | Medium-High |
| KJ-5 | Two sessions exposed; no evidence tokens were used | High / Low |
| KJ-6 | Probable objective is payment fraud | Medium |

No attribution to a named threat actor is offered.

All data is synthetic and defanged. See [DISCLAIMER](../../DISCLAIMER.md).
