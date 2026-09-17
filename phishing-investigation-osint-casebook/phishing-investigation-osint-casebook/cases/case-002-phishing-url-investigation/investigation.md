# Case 002 — Phishing URL Investigation

| | |
|---|---|
| **Case ID** | CASE-002 |
| **Input** | URL extracted in CASE-001 |
| **Opened** | 2026-01-14T10:05Z |
| **Data** | Synthetic |
| **Classification** | TLP:CLEAR |

---

## 1. Objective

Establish where the link actually leads, how it conceals that, what hosts it, and whether the
terminal page collects credentials — producing pivot points for domain and infrastructure
analysis.

## 2. Input

```
hxxps://mailbox-verify.relay-ns2[.]invalid/auth/login?u=[TOKEN-REDACTED]
```

The `u=` parameter is a victim-unique token. It was **substituted with a control value**
before any sandbox submission. Submitting the original would publish the recipient identifier
and signal to the operator that the message was reported.

## 3. URL decomposition

| Component | Value | Analytic note |
|---|---|---|
| Scheme | `https` | TLS is ubiquitous on phishing sites; the padlock proves encryption, not identity |
| Subdomain | `mailbox-verify` | Carries the pretext; free to create, no registration required |
| Registrable domain | `relay-ns2[.]invalid` | **The only part that required registration** — the true identity of the site |
| Path | `/auth/login` | Mimics an authentication endpoint |
| Query | `u=<base64 recipient id>` | Personalisation / tracking; enables the kit to pre-fill the address field |

The discipline here is trivial but frequently skipped: identify the registrable domain using
the public suffix list before reading anything else. Brand strings in subdomains and paths
are free and therefore evidentially worthless as identity claims.

## 4. Redirect chain

Sandboxed retrieval (control token), 2026-01-14T10:12Z:

| Hop | URL (defanged) | Status | Host IP | Note |
|---|---|---|---|---|
| 1 | `hxxps://mailbox-verify.relay-ns2[.]invalid/auth/login?u=CONTROL` | 302 | `203.0.113.77` | Initial contact; same IP as the mail sender |
| 2 | `hxxps://cdn-assets.trk-shorten[.]test/r/4417` | 302 | `198.51.100.12` | Third-party redirect service |
| 3 | `hxxps://secure-login.meridian-verify[.]example/session/auth` | 200 | `198.51.100.44` | Terminal page |

**Observations.**

- Three distinct IPs across three hops. The delivery host, the redirector and the landing
  host are separate — a deliberate compartmentalisation that raises takedown cost and means
  blocking any one of them does not break the chain.
- Hop 2 is a link-shortening/tracking service. This is the hop most likely to be *abused
  legitimate infrastructure* rather than attacker-owned, and must be treated differently: it
  is a takedown/abuse-report target, not a blocklist entry.
- Only at hop 3 does a domain appear that impersonates the target organisation's name.

**Cloaking check.** A second retrieval from a residential-egress browser with a mobile user
agent returned the same terminal page. A third retrieval from a datacentre IP with a
known-scanner user agent returned an HTTP 404. This asymmetry is itself a finding: the site
serves different content to suspected analysts, which means **any automated reputation
verdict on this URL is unreliable**, and a "clean" result must not be read as benign.

## 5. Terminal page characterisation

| Observation | Evidence |
|---|---|
| Page presents a single-field email prompt, then a password prompt on submission | Sandbox DOM capture |
| Email field pre-populated from the `u=` parameter | Confirmed by varying the control token |
| Branding assets visually imitate a generic mail provider login | Screenshot (not published — see `screenshots/README.md`) |
| Form POSTs to `/session/collect` on the same host | DOM inspection — **the POST target was read, not executed** |
| No credentials were submitted at any point | Analyst attestation |

The two-step email-then-password flow is a recognisable pattern in modern credential kits: it
allows the kit to validate the address and select branding before asking for the secret. It
is a behavioural observation worth more than any of the atomic indicators, because it
survives infrastructure rotation.

## 6. Hosting and network context

| Attribute | Hop 1 (`203.0.113.77`) | Hop 3 (`198.51.100.44`) |
|---|---|---|
| PTR | `mx1.relay-ns2.invalid` | `srv-shared-118.host-example.test` |
| ASN | `AS64501` | `AS64510` |
| Holder (RDAP) | Small VPS reseller | Mass shared-hosting provider |
| Allocation region | RIPE | RIPE |
| Co-hosted domains (reverse IP) | 2 | **>900** |
| Assessment | Likely dedicated to this operation | Shared hosting — co-location is near-meaningless |

This table is the reason IP indicators in this casebook carry explicit action guidance.
`203.0.113.77` is a plausible block candidate. `198.51.100.44` is not, at any confidence
level, because blocking it would break hundreds of unrelated sites.

## 7. Certificate

| Field | Value |
|---|---|
| Subject CN | `secure-login.meridian-verify[.]example` |
| SANs | `meridian-verify[.]example`, `secure-login.meridian-verify[.]example`, `mail.meridian-verify[.]example` |
| Issuer | Free automated CA, DV |
| Validity from | 2026-01-11T14:02Z |
| SHA-256 | `SYNTHETIC-CERT-002` |

**Interpretation.** DV issuance from a free CA is the default for the whole modern web and
carries no signal about intent. What *is* informative: the certificate predates delivery by
roughly 72 hours, giving a lower bound on staging time, and the SAN list discloses two
sibling hostnames not present in the original email — including `mail.`, which is a pivot
worth following in Case 003.

## 8. Registration snapshot

| Domain | Created | Registrar | Registrant | Note |
|---|---|---|---|---|
| `relay-ns2[.]invalid` | 2025-11-02 | Registrar A | Redacted (privacy) | ~10 weeks old at delivery |
| `meridian-verify[.]example` | 2026-01-10 | Registrar B | Redacted (privacy) | 4 days old at delivery |

Recency is a risk *signal*, not a verdict — see Case 003 §3 for why this is treated
carefully. The relevant point here is the **relationship between dates**: domain registered
2026-01-10, certificate issued 2026-01-11, mail delivered 2026-01-14. A coherent three-day
preparation sequence is more informative than any single date.

## 9. Reputation

| Source | Result at 2026-01-14T10:30Z | Weight |
|---|---|---|
| Multi-engine URL aggregator | 2/90 engines flag | Low — early-campaign under-detection is normal, and cloaking (§4) undermines automated scanning entirely |
| Public phishing feed | No entry | Low — absence indicates no prior report, not benignity |
| Browser safe-browsing status | Not flagged | Low |

Reputation contributed nothing to the assessment in this case, and the report says so
explicitly rather than quietly omitting it. A clean reputation result in the first hours of a
campaign is the expected observation, not a counter-indication.

## 10. Timeline

| UTC | Event | Source | Reliability |
|---|---|---|---|
| 2025-11-02 | `relay-ns2[.]invalid` registered | RDAP | A |
| 2026-01-10 | `meridian-verify[.]example` registered | RDAP | A |
| 2026-01-11T14:02Z | Certificate issued for landing domain | CT log | A |
| 2026-01-12 | First passive-DNS resolution of landing domain | pDNS | B — lower bound |
| 2026-01-14T08:40Z | Message delivered | `Received` header, first trusted hop | A |
| 2026-01-14T10:12Z | Landing page observed live | Sandbox | A |

## 11. Assessment

The URL leads, via a two-hop redirect chain including an abused third-party redirector, to a
**credential-collection page impersonating a mail login and personalised to the recipient**.
**Confidence: High** — the terminal page's form structure, the pre-population behaviour, the
impersonating domain name and the staging timeline are mutually consistent, and the
alternative explanation (a legitimate but oddly-configured service) is excluded by the
cloaking behaviour, which has no benign purpose.

The three hops are assessed as **separately controlled**: hop 2 is likely abused legitimate
infrastructure rather than adversary-owned. **Confidence: Medium** — based on the service's
general-purpose nature, without access to its account data.

## 12. Intelligence gaps

| Gap | Impact | How to close |
|---|---|---|
| Where collected credentials are exfiltrated | Determines whether a mailbox, Telegram bot or remote host is involved; changes hunt scope | Kit source recovery (not attempted — would require unauthorised access) |
| Whether other organisations are targeted by the same kit | Sharing value; campaign scope | CT search on the kit's naming pattern; CSIRT enquiry |
| Whether hop 2 is abused or attacker-registered | Determines abuse-report vs. block | Abuse report to the redirect provider |
| Identity of `mail.meridian-verify[.]example` role | May indicate mail-receiving capability for credential exfiltration | DNS MX check — Case 003 |

## 13. Pivots into Case 003

- `meridian-verify[.]example` — full domain investigation
- `mail.meridian-verify[.]example` — MX/exfiltration hypothesis
- Certificate SAN list — sibling discovery
- `AS64510` / `198.51.100.44` — hosting context, with co-hosting caveat carried forward
