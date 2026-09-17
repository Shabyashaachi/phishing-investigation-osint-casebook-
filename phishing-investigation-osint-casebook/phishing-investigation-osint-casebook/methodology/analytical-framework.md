# Analytical Framework

The structure applied to every significant finding in this casebook. Its purpose is to keep
observation and inference separate on the page, so a reader can check the reasoning rather
than trust the conclusion.

## The seven fields

| Field | Question it answers | Failure mode it prevents |
|---|---|---|
| **Observation** | What was directly seen, stated neutrally? | Smuggling judgement into description ("a malicious domain was observed") |
| **Evidence** | Which artefact, source and timestamp supports it? | Unsourced assertion |
| **Interpretation** | What could this indicate? | Jumping to the only interpretation the analyst thought of first |
| **Alternative explanation** | What benign or different cause produces the same observation? | Confirmation bias |
| **Assessment** | What can reasonably be concluded, given both? | Overclaiming |
| **Confidence** | How strong is the evidential basis? | False precision |
| **Intelligence gap** | What remains unknown? | Presenting a partial picture as complete |

## Worked example

> **Observation.** `secure-meridian-fin[.]example` resolved to `198.51.100.44` on
> 2026-02-11T09:14Z. `meridian-verify[.]example` resolved to the same address on
> 2026-02-11T09:16Z.
>
> **Evidence.** `dig +short` output, `evidence/dns-queries.txt`, queries 4 and 5; both
> timestamped UTC.
>
> **Interpretation.** Shared hosting could indicate both domains are operated by the same
> actor as part of one campaign.
>
> **Alternative explanation.** `198.51.100.44` belongs to a shared-hosting provider and
> announces a reverse-DNS name typical of mass virtual hosting. Reverse-IP data shows a high
> count of unrelated domains on the same address. Two unrelated customers of the same cheap
> host produce this observation trivially.
>
> **Assessment.** Co-resolution alone does **not** support common operation. A second,
> independent and more selective link is required — and in this case exists: both names
> appear as SANs on a single certificate (serial `0x4d…`, issued 2026-02-09), which requires
> that one party demonstrated control of both names to the CA simultaneously. On the
> combined evidence, common operation is assessed as likely.
>
> **Confidence.** Medium-High. The certificate link is selective and independent of the
> hosting link; it remains theoretically possible that a hosting platform bundled both names
> into one certificate on behalf of separate customers, which is why this is not High.
>
> **Intelligence gap.** Whether the SAN bundle was customer-generated or platform-generated
> is unknown. The CA's issuance metadata and the hosting platform's certificate policy would
> resolve it; neither is publicly available.

The example demonstrates the discipline that matters most: the first, obvious inference was
**rejected**, and the conclusion was carried by a different piece of evidence entirely.

## Language discipline

Words are load-bearing. This casebook uses them consistently:

| Term | Means |
|---|---|
| **Observed** | Directly seen in evidence, reproducible from the artefact |
| **Reported** | Stated by a third-party source, not independently verified |
| **Consistent with** | Compatible with a hypothesis; does not discriminate between hypotheses |
| **Indicates** | Evidence favours this over alternatives |
| **Confirmed** | Verified by at least two independent sources, or directly by primary evidence |
| **Assessed** | An analytic judgement, not a fact |
| **Unknown** | No evidence either way — distinct from "no evidence found", which is itself evidence of a kind |

"Likely", "probably" and similar estimative terms are always paired with an explicit
confidence label, never used alone.

## When the framework is skipped

Routine, non-load-bearing observations (a TTL value, a registrar name) are recorded in tables
without the full structure. The framework is applied wherever a finding carries an inference
that a reader might otherwise accept unexamined — which, in a phishing investigation, is
most of the interesting ones.
