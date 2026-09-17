# Threat Intelligence Report — EU-FIN-001

**Credential Phishing Campaign Targeting Meridian Financial Group**
Technical Analyst Report · TLP:CLEAR · Version 1.0 · 2026-02-18

| | |
|---|---|
| Report ID | CTI-EU-FIN-001-TR |
| Author | [Author], independent security research |
| Scope | Wave 2, 2026-02-11; references Wave 1, 2026-01-14 |
| Audience | SOC analysts, detection engineers, mail and IAM teams |
| Data | Synthetic scenario — see [DISCLAIMER](../../DISCLAIMER.md) |

> Working record with full reasoning, alternative explanations and rejected hypotheses:
> [`investigation.md`](investigation.md). Executive version:
> [`executive-brief.md`](executive-brief.md).

---

## 1. Executive summary

On 2026-02-11, eleven Meridian Financial Group staff — nine of them in payment-authorising
roles — received a spearphishing email impersonating the organisation's IT Security function.
The message passed SPF and DKIM for the adversary's own sending domain and failed DMARC
alignment for the spoofed Meridian domain, but was delivered because a mail-gateway
allow-list overrode the quarantine policy.

The linked site was **not a credential-harvest clone**. It operated as an
adversary-in-the-middle reverse proxy in front of the genuine identity provider, capable of
relaying MFA challenges and capturing post-authentication session tokens. Four users
connected; two submitted credentials and completed MFA. Both sessions were revoked within 34
minutes of the first interaction.

The campaign is assessed with Medium-High confidence to be the same operator as a January
2026 wave against the same organisation, with materially improved tradecraft. No attribution
to a named threat actor is offered.

The single most important defensive conclusion: **password resets and indicator blocking do
not address this technique.** Phishing-resistant, origin-bound authentication (FIDO2/WebAuthn)
for payment-authorising roles is the only recommended control that removes it.

---

## 2. Investigation objective

Determine the origin, infrastructure, capability and objective of a reported phishing message;
extract indicators; identify techniques; assess confidence and gaps; and produce defensive
recommendations that outlive the campaign infrastructure.

---

## 3. Initial evidence

| Item | Value |
|---|---|
| Artefact | One reported `.eml`, gateway export |
| Subject | `[Action] Re-authenticate your Meridian SSO session` |
| Delivered | 2026-02-11T08:31:09Z – 08:34Z |
| Recipients | 11 (9 Treasury/Payments, 2 Finance leadership) |
| Interaction | 4 connections, 2 POSTs (proxy logs) |
| Reported | 08:52Z by recipient |

Evidence manifest and hashes: [`evidence/MANIFEST.md`](evidence/MANIFEST.md).

---

## 4. Methodology

Seventeen-stage process:
[`methodology/phishing-investigation-methodology.md`](../../methodology/phishing-investigation-methodology.md).
Findings are structured as observation → evidence → interpretation → alternative explanation →
assessment → confidence → gap
([analytical framework](../../methodology/analytical-framework.md)). Source reliability is
graded A–F and information credibility 1–6
([source evaluation](../../methodology/osint-source-evaluation.md)).

All collection was passive or sandboxed. No interaction with adversary infrastructure beyond
a single sandboxed retrieval with a substituted token. No credentials were submitted at any
point.

---

## 5. Technical findings

### 5.1 Email authentication

```
spf=pass    smtp.mailfrom=mail-relay-eu.invalid
dkim=pass   header.d=mail-relay-eu.invalid header.s=s1
dmarc=fail  (p=quarantine sp=quarantine dis=none) header.from=meridian-fin.example
```

The adversary controls `mail-relay-eu[.]invalid` and configured valid SPF (`-all`) and DKIM
signing for it. Authentication therefore **passes for the adversary's domain** while DMARC
fails for the spoofed display domain, because neither authenticated identifier aligns with
`meridian-fin.example`.

`dis=none` records that the receiving gateway did **not** apply the published quarantine
policy. Cause: gateway allow-listing. This is the proximate cause of delivery.

### 5.2 Routing

Injection point `203.0.113[.]140` (`mx3.mail-relay-eu[.]invalid`), first trusted hop at
08:31:09Z. Hops below are sender-asserted; hop 1 claims an RFC 1918 address, indicating
submission from behind the sender's own NAT.

### 5.3 Message construction

Directory-quality personalisation (correct first names and job titles). Inverted-urgency
pretext: a claimed unrecognised device registration, where the security-conscious response is
to click. No `Reply-To`, no `X-Mailer` — both present in Wave 1 and both removed, reducing
signature surface.

### 5.4 Landing infrastructure — adversary-in-the-middle

Sandbox capture recorded server-side requests from `198.51.100[.]44` to the genuine Meridian
IdP, a completed MFA push challenge relayed back to the client, and a valid session cookie
returned to the phishing host.

This excludes a static clone, which cannot complete an MFA challenge and whose asset requests
originate client-side. The infrastructure is positioned to capture session tokens, defeating
push-notification and OTP MFA.

**Operational consequence:** a stolen session token survives a password reset. Session
revocation is mandatory.

### 5.5 Domain and DNS

| | Landing | Sending |
|---|---|---|
| Domain | `sso-meridian-fin[.]example` | `mail-relay-eu[.]invalid` |
| Registered | 2026-02-08T22:14Z (3 days pre-delivery) | 2025-09-19 (5 months pre-delivery) |
| A record | `198.51.100[.]44`, TTL 120 | `203.0.113[.]140`, TTL 3600 |
| MX | none | `mx3.mail-relay-eu[.]invalid` |
| SPF/DKIM | none | valid, `-all`, selector `s1` |
| Nameservers | `ns1/ns2.dnspark-lite[.]test` | same |

The lifecycle split — disposable landing infrastructure, long-lived reputation-bearing
sending infrastructure — is a durable behavioural observation and more useful than either
domain as an indicator.

### 5.6 Certificates

| Cert | SANs | Issued |
|---|---|---|
| C3 | `sso-meridian-fin[.]example` | 2026-02-09T06:41Z |
| C4 | `sso-meridian-group[.]example`, `login-meridian-fin[.]example` | 2026-02-09T06:44Z |

C4 covers domains never observed in delivery — staged, unused infrastructure discovered via a
CT string search. Both waves showed a used/unused certificate pair issued ~3 minutes apart
(Low-Medium confidence as a pattern; see §10).

---

## 6. Infrastructure analysis

| Edge | Basis | Confidence |
|---|---|---|
| Sending domain ↔ landing domain | Observed causal link in the artefact | High |
| C3 ↔ C4 | 3-min CT gap, same CA, same naming scheme | Medium |
| Wave 1 ↔ Wave 2 | Shared landing IP, shared NS provider, shared registrar, identical target — four independent dimensions | Medium-High |
| Landing IP ↔ ~900 co-tenants | Shared mass-hosting address only | **Rejected** |
| Either domain ↔ registrar portfolio | Shared registrar only | **Rejected** |

`198.51.100[.]44` is mass shared hosting (PTR `srv-shared-118`, >900 tenants). It is campaign
infrastructure but **must not be blocked**. `203.0.113[.]140` has three tenants and is a safe
blocking candidate.

Graph and per-edge reasoning: [`investigation.md` §11](investigation.md).

---

## 7. IOC table

Defanged. Machine-readable export: [`../../iocs/`](../../iocs/).

| ID | Type | Indicator | Confidence | Status | Action |
|---|---|---|---|---|---|
| IOC-005-01 | domain | `sso-meridian-fin[.]example` | High | retired (ceased resolving 2026-02-13) | block |
| IOC-005-02 | domain | `mail-relay-eu[.]invalid` | High | active | block |
| IOC-005-03 | domain | `sso-meridian-group[.]example` | Medium | active (staged, unused) | block |
| IOC-005-04 | domain | `login-meridian-fin[.]example` | Medium | active (staged, unused) | block |
| IOC-005-05 | ipv4 | `203.0.113[.]140` | High | active | block |
| IOC-005-06 | ipv4 | `198.51.100[.]44` | High | shared_infrastructure | **monitor — do not block** |
| IOC-005-07 | url | `hxxps://sso-meridian-fin[.]example/auth/realms/meridian/login` | High | retired | block |
| IOC-005-08 | email | `no-reply[@]meridian-fin[.]example` | High (spoofed) | — | **enrich_only — victim's own domain** |
| IOC-005-09 | email | `bounce[@]mx3.mail-relay-eu[.]invalid` | High | active | block |
| IOC-005-10 | cert_sha256 | `SYNTHETIC-CERT-C3` | High | retired | monitor |
| IOC-005-11 | cert_sha256 | `SYNTHETIC-CERT-C4` | Medium | active | monitor |
| IOC-005-12 | asn | `AS64502` | — | context only | enrich_only |

IOC-005-06 and IOC-005-08 carry deliberate non-blocking actions. Both would cause operational
damage if handled by confidence alone.

---

## 8. TTP analysis

| Durable behaviour | Defensive implication |
|---|---|
| Aged, reputation-bearing sending domain with valid SPF/DKIM | Reputation and authentication filtering will not catch this; DMARC alignment enforcement will |
| Reverse-proxy AiTM landing infrastructure | Only origin-bound authentication defeats it |
| Disposable landing / persistent sending split | Block sending infrastructure aggressively; treat landing indicators as short-lived |
| CT-visible staging 2–3 days pre-delivery | CT monitoring provides advance warning |
| Inverted-urgency security pretext | Awareness content must address this specifically; generic "spot the phish" training is counter-productive here |
| Directory-quality targeting of payment roles | Role-based control hardening is justified |

---

## 9. MITRE ATT&CK mapping

| ID | Technique | Evidence | Confidence |
|---|---|---|---|
| T1566.002 | Spearphishing Link | Message artefact; 11-recipient wave | High |
| T1598.003 | Phishing for Information: Spearphishing Link | Landing form solicits credentials | High |
| T1656 | Impersonation | IT Security and SSO portal impersonation | High |
| T1583.001 | Acquire Infrastructure: Domains | RDAP creation dates | High |
| T1608.005 | Stage Capabilities: Link Target | CT issuance 48h pre-delivery | High |
| T1557 | Adversary-in-the-Middle | Sandbox network capture, completed MFA relay | High |
| T1539 | Steal Web Session Cookie | Session cookie returned to proxy host | Medium-High |
| T1585.002 | Establish Accounts: Email Accounts | 5-month-old sending domain with valid auth | Medium |

**Not mapped:** T1078 (Valid Accounts) — capture position observed, use not observed. T1114,
T1586, T1534 — no supporting evidence. Rationale: [`investigation.md` §12](investigation.md).

---

## 10. Timeline

| UTC | Event | Reliability |
|---|---|---|
| 2025-09-19T11:02Z | Sending domain registered | A |
| 2026-01-11 → 01-14 | Wave 1 | A |
| 2026-02-08T22:14Z | Landing domain registered | A |
| 2026-02-09T06:41Z / 06:44Z | Certificates C3 / C4 issued | A |
| 2026-02-10 | First passive-DNS resolution (lower bound) | B |
| 2026-02-11T08:31–08:34Z | 11 messages delivered | A |
| 2026-02-11T08:39Z | First connection | A |
| 2026-02-11T08:43Z / 08:47Z | Two POSTs | A |
| 2026-02-11T08:52Z | Reported | A |
| 2026-02-11T09:05Z | Sessions revoked | A |
| 2026-02-13T14:00Z | Landing domain ceases resolving | A |

Delivery to first interaction: **8 minutes**. Delivery to report: **21 minutes**. Interaction
to containment: **26 minutes**.

---

## 11. Assessment and confidence

| ID | Key judgement | Confidence |
|---|---|---|
| KJ-1 | Targeted credential phishing from adversary-controlled infrastructure impersonating Meridian SSO | High |
| KJ-2 | Landing infrastructure was an AiTM reverse proxy capable of session-token capture | High |
| KJ-3 | Recipient selection was deliberate and concentrated on payment-authorising roles | Medium-High |
| KJ-4 | Same operator as Wave 1 | Medium-High |
| KJ-5 | Two sessions were exposed to token capture; no evidence tokens were used | High (exposure) / Low (use) |
| KJ-6 | Probable objective is payment fraud rather than credential resale | Medium |

**No attribution to a named actor.** The observed tradecraft is available in commodity and
open-source form and is used by many financially motivated actors; nothing observed narrows
it.

---

## 12. Intelligence gaps

| # | Gap | Impact | Closure | Feasible |
|---|---|---|---|---|
| G-1 | Whether captured tokens were used | Attempted vs. actual compromise | IdP sign-in review, 72h | Yes |
| G-2 | Kit family | Cross-campaign correlation | Artefact recovery / public reporting | Partial |
| G-3 | Source of recipient names and titles | Public scraping vs. prior compromise | Exposure review + access hunt | Yes |
| G-4 | Other targeted organisations | Campaign scale, sharing value | CSIRT/ISAC enquiry | Yes |
| G-5 | Exfiltration destination | Infrastructure picture beyond this campaign | — | No |
| G-6 | Cause of DMARC override | **Root cause of delivery** | Gateway policy review | Yes |
| G-7 | Validity of CT pairing pattern | Predictive blocking | Third wave / broader study | Partial |

---

## 13. Defensive recommendations

**Immediate**

1. Revoke all sessions for affected users (password reset alone is insufficient — KJ-2).
2. Review IdP sign-in logs, 72h window, for impossible travel and new device registrations (G-1).

**High priority**

3. Block domains IOC-005-01 through -04 and IP IOC-005-05 at DNS, proxy and mail gateway.
4. **Do not block** `198.51.100[.]44` (900+ unrelated tenants).
5. Remediate the gateway allow-list that overrode DMARC quarantine (G-6) — this is the root cause.
6. Move own DMARC policy to `p=reject` after alignment monitoring is clean.
7. Deploy FIDO2/WebAuthn for payment-authorising roles — **the only control that removes the AiTM technique**.

**Medium priority**

8. Alert on IdP authentication from network paths or device fingerprints inconsistent with user baseline.
9. Continuous CT monitoring for brand-lookalike registrations (both waves were visible 2–3 days early).
10. 90-day retrospective hunt across proxy, DNS and mail logs for both waves' indicators.
11. Targeted briefing for Treasury/Payments on the inverted-urgency pretext.
12. Share defanged IOCs and TTPs with the sector CSIRT/ISAC.

Items 3–4 expire within days. Items 5–7 and 9 are permanent.

Detection logic and queries: [`../../soc-integration/`](../../soc-integration/).

---

## 14. Appendix

- A1 — Raw headers: [`evidence/eu-fin-001-headers.txt`](evidence/eu-fin-001-headers.txt)
- A2 — DNS record captures: [`evidence/eu-fin-001-dns.txt`](evidence/eu-fin-001-dns.txt)
- A3 — Redirect and network capture: [`evidence/eu-fin-001-redirect-chain.txt`](evidence/eu-fin-001-redirect-chain.txt)
- A4 — Certificate extracts: [`evidence/eu-fin-001-certificates.txt`](evidence/eu-fin-001-certificates.txt)
- A5 — Evidence manifest: [`evidence/MANIFEST.md`](evidence/MANIFEST.md)
- A6 — Full working record: [`investigation.md`](investigation.md)

## 15. Sources

| Source | Used for | Reliability |
|---|---|---|
| Primary artefact (`.eml`) | Headers, content, URL | A |
| Authoritative DNS (`dig`) | Record set, SPF/DKIM/DMARC | A |
| Registry RDAP | Registration dates, registrar, nameservers | A |
| Certificate Transparency (crt.sh) | Certificate issuance, SAN discovery | A |
| Sandbox retrieval | Redirect chain, AiTM behaviour | A |
| RIR RDAP / routing data | ASN, prefix, allocation | A |
| Reverse-IP service | Co-tenancy density | C |
| Internal proxy and gateway logs | Interaction, wave size | A (internal) |
| Public reporting on reverse-proxy phishing kits | TTP context only | C |

All external collection was passive or sandboxed. Reliability grades per
[osint-source-evaluation.md](../../methodology/osint-source-evaluation.md).
