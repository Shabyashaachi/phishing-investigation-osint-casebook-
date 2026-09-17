# Disclaimer, Scope and Ethics

## What this repository is

An **independent security-research and portfolio project** demonstrating phishing
investigation and OSINT methodology.

## What it is not

- Not professional client work
- Not a record of employment or consulting engagement
- Not an incident report concerning any real organisation
- Not a claim of professional experience

The organisations, employees, campaigns, artefacts and indicators in this repository are
**fabricated**. Any resemblance to real organisations or individuals is coincidental.

## Data conventions

All values use ranges reserved by standards bodies for documentation, which guarantees they
cannot resolve to real infrastructure:

| Element | Convention | Reference |
|---|---|---|
| Domains | `.example`, `.invalid`, `.test` | RFC 2606, RFC 6761 |
| IPv4 | `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24` | RFC 5737 |
| IPv6 | `2001:db8::/32` | RFC 3849 |
| ASN | `64496–64511`, `65536–65551` | RFC 5398 |
| Hashes | Placeholders marked `SYNTHETIC` | — |

Tool output shown in the cases is **reconstructed to match real tool formats**. It was not
captured from live queries against real infrastructure.

## What was not done

No phishing email was sent. No credentials were harvested. No victim was contacted. No system
was exploited or accessed without authorisation. No authentication was bypassed. No phishing
infrastructure was deployed. No private information about any real person was collected or
published. No usable malicious URL appears in this repository.

## Defanging

All indicators in human-readable material are defanged (`hxxp://`, `[.]`, `[@]`). Values in
the machine-readable exports under `iocs/` are synthetic and cannot resolve; they must not be
loaded into production enforcement systems.

## Methodology, applied to real cases

The methodology here is intended for **defensive use**: investigating phishing that targets
your own organisation, or that you are authorised to investigate. If you apply it:

- Prefer passive collection. Active interaction tips off the operator and can destroy the
  evidence you are collecting.
- Never submit victim-unique tokens to public sandboxes.
- Never submit credentials to phishing infrastructure, real or fabricated.
- Minimise and pseudonymise personal data before retention or sharing. Under GDPR this is an
  obligation, not a courtesy.
- Investigate only within your authorisation. Unauthorised access to systems is a criminal
  offence across the EU regardless of intent.

## Analytical honesty

Observations, evidence, hypotheses, assessments and intelligence gaps are labelled distinctly
throughout. Confidence levels are stated with justification. **No attribution to any named
threat actor is made**, because the evidence in an OSINT-only phishing investigation rarely
supports it.

## Licence

Content is MIT licensed (see `LICENSE`). Attribution is appreciated. The synthetic data
carries no warranty of any kind and must not be treated as real threat intelligence.
