# Confidence Assessment

Confidence describes **the quality of the evidential basis for a judgement**. It is not the
analyst's certainty, not a probability, and not a severity rating. Those are separate axes
and conflating them is a common junior mistake.

## Three distinct things

| Axis | Question | Scale used here |
|---|---|---|
| **Source reliability** | How dependable is the source itself? | A–F (see [osint-source-evaluation.md](osint-source-evaluation.md)) |
| **Information credibility** | How well corroborated is this specific item? | 1–6 |
| **Analytic confidence** | How strong is the basis for my *judgement* built on it? | Low / Medium / High |

A judgement can rest on highly reliable sources and still warrant low confidence, if the
inference chain from data to conclusion is long or an alternative explanation survives.

## Confidence levels

### High confidence

- Multiple **independent** sources agree, or primary evidence establishes the point directly.
- Plausible alternative explanations have been tested and excluded.
- The judgement survives any single piece of evidence being wrong.
- Estimative language: *is*, *confirms*, *demonstrates*, *almost certainly*.

> Example: "The message failed SPF, DKIM and DMARC for the displayed domain." — directly
> observable in `Authentication-Results`, corroborated by independent lookup of the domain's
> published SPF record.

### Medium confidence

- Credible sources, but limited corroboration or partly circumstantial evidence.
- One or more alternative explanations remain possible but less likely.
- The judgement would weaken materially if one key item were wrong.
- Estimative language: *likely*, *probably*, *indicates*, *assessed to*.

> Example: "The two domains are likely operated by the same actor." — supported by shared
> certificate SANs and matching page structure, but platform-generated certificate bundling
> has not been excluded.

### Low confidence

- Single source, uncorroborated, fragmentary, or dated evidence.
- Alternative explanations are equally plausible.
- Included because it may matter, explicitly flagged as weak.
- Estimative language: *possibly*, *may*, *cannot be excluded*, *one hypothesis is*.

> Example: "The operator may be reusing a commodity phishing kit." — based on a single
> structural resemblance to publicly described kits, with no kit artefact recovered.

## What does *not* raise confidence

- **Volume of weak indicators.** Ten non-selective signals do not sum to one selective one.
  Newly registered + privacy-protected + cheap TLD + shared hosting describes an enormous
  population of legitimate domains.
- **Repetition across sources.** Three reputation feeds that all ingest the same upstream
  blocklist are one source, not three. Check independence before counting corroboration.
- **Tool verdicts.** A vendor score is an input, not corroboration, and its methodology is
  usually opaque.
- **Narrative coherence.** A story that hangs together well is not thereby better evidenced;
  it may simply be the only story the analyst constructed.

## Recording confidence

Every key judgement in this casebook carries a label and a one-sentence justification:

> **KJ-3.** The credential-collection page was staged at least 48 hours before delivery.
> **Confidence: Medium** — CT issuance and passive-DNS first-seen agree on 2026-02-09, but
> both are lower bounds on existence rather than exact staging times.

## Changing an assessment

Confidence is revisable. When new evidence changes a judgement, the case file records the
change rather than silently overwriting it:

> *Revision 2026-02-14:* KJ-2 lowered from High to Medium following identification of
> `198.51.100.44` as shared hosting with a high co-hosted-domain count, which reinstates the
> alternative explanation previously considered excluded.

Publishing revisions is a feature. An analytic record that never changes its mind is not
being checked.
