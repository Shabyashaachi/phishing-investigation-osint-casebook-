# Phishing Investigation Methodology

A seventeen-stage process for taking a phishing artefact from arrival to actionable
intelligence. Every stage states its objective, the questions it answers, the evidence it
collects, suitable OSINT sources, its limitations, and its expected output.

Stages are ordered but not strictly linear. Stages 6–11 loop: a certificate discovered at
stage 10 frequently sends you back to stage 7 with a new domain. The loop terminates when
new pivots stop producing new indicators, not when the analyst runs out of time.

---

## Stage 1 — Preparation

**Objective.** Establish a safe, repeatable working environment before touching the artefact.

**Questions.** What am I permitted to do with this sample? What is my analysis environment?
What is my note-taking and naming convention? Who consumes the output?

**Evidence collected.** None yet — this stage produces the case record: case ID, opening
timestamp (UTC), requester, scope, authorisation.

**Sources / tooling.** Isolated VM or container, no corporate credentials, no default
browser profile. Egress through a connection not attributable to the organisation when
active interaction is contemplated.

**Limitations.** Preparation cannot be retrofitted. An analyst who clicks first and thinks
second has already tipped off the adversary and contaminated the evidence.

**Output.** Case folder, case ID, scope statement.

---

## Stage 2 — Initial triage

**Objective.** Decide in minutes whether this is a false alarm, a commodity phish, or
something warranting deep investigation.

**Questions.** What is the artefact? Who received it and how many? Did anyone interact? Is
credential compromise plausible already? Is this a bulk campaign or targeted?

**Evidence collected.** Reporter account, delivery timestamp, recipient count, subject line,
interaction status.

**Sources.** Mail gateway search, abuse mailbox, initial visual inspection of the raw source.

**Limitations.** Triage is deliberately shallow and will occasionally be wrong. Its output is
a *priority*, not a *verdict*. Never let a triage label harden into a conclusion — that is
the most common route to anchoring bias in phishing work.

**Output.** Severity/priority assignment, decision to escalate or close.

---

## Stage 3 — Evidence preservation

**Objective.** Freeze the artefact in a state that can be re-examined and defended later.

**Questions.** Do I have the original, unmodified source? Is its integrity verifiable? Is
provenance recorded?

**Evidence collected.** Raw `.eml` with full headers, SHA-256 of every file, acquisition
timestamp, acquisition method, custodian.

**Sources.** Mail client "show original" / gateway export; `sha256sum`; write-once storage.

**Limitations.** Forwarded copies are not originals — forwarding rewrites headers and
destroys the routing chain. If only a forwarded copy exists, that limitation must be stated
in the report, because the entire header analysis inherits it.

**Output.** Evidence manifest with hashes. See [evidence-handling.md](evidence-handling.md).

---

## Stage 4 — IOC extraction (first pass)

**Objective.** Pull every atomic indicator out of the artefact before interpreting any of it.

**Questions.** What addresses, domains, URLs, IPs, hashes, and identifiers exist in this
sample?

**Evidence collected.** Sender addresses, `Reply-To`, `Return-Path`, `Message-ID` domain,
all `Received` hop hosts and IPs, every URL in body and attachments, attachment hashes,
tracking identifiers.

**Sources.** Raw source inspection; a parser is convenient but manual reading catches what
parsers normalise away.

**Limitations.** First-pass extraction is inventory, not assessment. Marking indicators
malicious at this stage guarantees false positives — the sample will contain legitimate
infrastructure (the recipient's own gateway, CDN links, unsubscribe footers).

**Output.** Raw indicator list, unscored.

---

## Stage 5 — Email analysis

**Objective.** Determine how the message was sent, by what infrastructure, and whether that
infrastructure was authorised.

**Questions.** Does the routing chain make sense read bottom-up? Where did the message enter
infrastructure the recipient trusts? Do SPF, DKIM and DMARC pass, and — critically — do they
*align* with the displayed domain? Do `From`, `Reply-To` and `Return-Path` diverge? Is the
`Message-ID` consistent with the claimed sender? Does the content show pretexting, urgency,
authority impersonation?

**Evidence collected.** Full header block, `Authentication-Results`, each `Received` hop with
timestamps, MIME structure, body text and HTML link targets.

**Sources.** Raw headers; sender-domain SPF/DMARC records via `dig TXT`; DKIM selector
lookup; RDAP for hop IPs.

**Limitations.** Headers above the first trusted hop are attacker-controllable and can be
entirely fabricated. SPF pass means the envelope sender was authorised — it does **not** mean
the message is benign; attacker-owned domains routinely publish valid SPF. Absence of DMARC
on the spoofed domain tells you about the victim's posture, not the attacker's intent.

**Output.** Header findings table, authentication verdict, sender-infrastructure hypothesis.

---

## Stage 6 — URL analysis

**Objective.** Understand where a link actually leads and how it disguises that.

**Questions.** What is the true registrable domain? Is the brand name in the path,
subdomain, or userinfo section rather than the domain? Are there redirect hops, and are any
of them legitimate services being abused as open redirectors? Does the final page solicit
credentials? Is there victim-specific encoding in the URL?

**Evidence collected.** Full URL, decomposed components, decoded parameters, redirect chain
with status codes, final landing URL, page structure.

**Sources.** Manual decomposition; URL-unfurling and sandbox services (urlscan.io, Hybrid
Analysis, VirusTotal URL view); CyberChef for encoding layers.

**Limitations.** Phishing kits cloak — they serve benign content to datacentre IPs, known
scanner user agents and repeat visitors. A clean sandbox verdict is weak evidence of
benignity. Conversely, submitting a URL containing a victim-unique token to a public sandbox
publishes that token and can alert the operator. Strip or substitute unique tokens first.

**Output.** Redirect chain, landing-page characterisation, terminal domain for pivoting.

---

## Stage 7 — Domain intelligence

**Objective.** Establish what is publicly known about the registration of the domain.

**Questions.** When was it registered? Which registrar? What nameservers? Is registrant data
redacted? Has it changed recently? What TLD, and does that TLD have known abuse economics?

**Evidence collected.** RDAP/WHOIS response, creation/updated/expiry dates, registrar and
IANA ID, nameserver set, status codes (EPP), abuse contact.

**Sources.** RDAP (`rdap.org` bootstrap, registry RDAP endpoints), `whois`, registry lookup,
ICANN Lookup.

**Limitations.** This is the single most over-read stage in junior phishing work. A recent
creation date is a *risk signal*, not a verdict — millions of legitimate domains are
registered every week. Privacy/proxy redaction is the GDPR-era default for EU registrants,
so redaction carries almost no evidential weight. Registrar identity indicates cost and
friction, not intent. Treat all three as weak priors that require corroboration.

**Output.** Registration profile, registration-timing observation, pivot candidates
(nameserver, registrar, registrant email if genuinely disclosed).

---

## Stage 8 — DNS analysis

**Objective.** Map the domain's current and historical resolution and mail posture.

**Questions.** What do A/AAAA return, and with what TTL? Is there an MX record — and is it
configured to *receive* mail, which distinguishes a credential-harvest page from a
mail-sending domain? What does TXT reveal (SPF, verification tokens, DMARC at `_dmarc`)? Are
nameservers self-hosted or a shared provider? Are there wildcard responses?

**Evidence collected.** Full record set per type, TTLs, authoritative server, query
timestamp, passive-DNS history where available.

**Sources.** `dig`, `nslookup`, DNSDumpster, SecurityTrails (free tier), Validin, Mnemonic
passive DNS, `_dmarc` and selector lookups.

**Limitations.** Live DNS is a snapshot; fast-flux and rapid re-pointing mean your A record
may be stale within hours — always timestamp queries in UTC. Free passive-DNS tiers have
thin historical coverage, and absence of history is not evidence of absence. Wildcard DNS
makes subdomain enumeration by brute force produce garbage.

**Output.** DNS record table with query timestamps, hosting IP(s), infrastructure pivots.

---

## Stage 9 — IP / ASN analysis

**Objective.** Place the hosting in context: who operates this address space and how it is
typically used.

**Questions.** Which ASN announces the prefix? Who is the operator — bulletproof host,
mainstream cloud, shared hosting, CDN? Is the IP dedicated or shared with thousands of
sites? Is it behind a reverse proxy that conceals the origin? What is the abuse contact?

**Evidence collected.** IP, PTR, prefix, ASN and holder, netblock allocation, geolocation
(country-level only), co-hosted domain count.

**Sources.** RDAP for IP, BGP looking glasses, bgp.he.net, RIPEstat (strong for European
allocations), Shodan/Censys host views, reverse-IP services.

**Limitations.** Shared hosting and CDN fronting destroy the "same IP = same operator"
inference; on a large shared host, co-location is nearly meaningless. Geolocation resolves
to allocation, not to a human's location, and city-level claims are unreliable. Behind
Cloudflare or similar, the observed IP is the proxy, and the origin remains unknown — say so
rather than reporting the proxy IP as the adversary's host.

**Output.** Hosting profile, ASN context, co-hosting caveat, confidence-weighted pivots.

---

## Stage 10 — Certificate intelligence

**Objective.** Use TLS certificates and Certificate Transparency as a discovery and
timeline source.

**Questions.** When was the first certificate issued for this name — does that predate or
follow the campaign? Which CA and validation level? What other names are in the SAN list?
Does a CT search for the brand string reveal sibling lookalike domains? Do multiple domains
share a single certificate?

**Evidence collected.** Certificate serial, SHA-256 fingerprint, issuer, validity window,
full SAN list, CT log entry timestamps.

**Sources.** crt.sh, Censys, `openssl s_client`, CT log search.

**Limitations.** Free automated CAs issue certificates in seconds to anyone controlling a
name, so issuance says nothing about intent. A shared certificate is a *strong* relational
signal (someone controlled every SAN name at issuance time) — but shared *hosting-provider*
certificates, where a platform bundles unrelated customers into one SAN list, are a known
false-positive source and must be checked for. First CT issuance is a useful lower bound on
operational readiness, not a precise campaign start.

**Output.** Certificate table, SAN-derived domain candidates, infrastructure-readiness
timestamp.

---

## Stage 11 — Infrastructure correlation

**Objective.** Determine which discovered elements genuinely belong to the same operation.

**Questions.** For each candidate relationship: what exactly links A and B? How selective is
that link? What innocent explanation produces the same observation? Does any second,
independent link corroborate it?

**Evidence collected.** For every edge in the graph: indicator A, indicator B, relationship
type, evidence, source, timestamp, confidence, alternative explanation.

**Sources.** Reverse DNS/IP, passive DNS, CT SAN overlap, nameserver reuse, registrant
email, favicon/HTML hashes, kit artefacts, TLS JARM.

**Limitations.** Correlation is where phishing investigations most often go wrong. Selectivity
is the discriminating property: a link that millions of unrelated hosts also share (same
large cloud ASN, same popular registrar, same public nameserver) carries almost no weight,
while a link few could coincidentally share (identical certificate, unique registrant
address, identical kit hash) carries a great deal. Never assert common ownership from a
single non-selective link. Full treatment in [Case 004](../cases/case-004-infrastructure-correlation/).

**Output.** Infrastructure graph with per-edge confidence; explicitly rejected edges.

---

## Stage 12 — TTP identification

**Objective.** Describe adversary behaviour in transferable terms, so defenders can act
after the indicators expire.

**Questions.** What did the adversary actually *do*? Which behaviours are durable (pretext
type, delivery method, kit family) versus disposable (this domain, this IP)? Which map to
ATT&CK techniques *supported by observation*?

**Evidence collected.** Behaviour statements paired with the specific observation evidencing
each.

**Sources.** MITRE ATT&CK Enterprise, ATT&CK for Phishing-adjacent techniques, kit
documentation, public reporting of similar TTPs.

**Limitations.** ATT&CK inflation is a recognisable junior-analyst tell. Mapping `T1056.003`
(Web Portal Capture) because a credential page *presumably* stores input, without having
observed the POST, is unsupported. Map what you evidenced; list the rest as hypotheses.

**Output.** TTP table with technique ID, observed behaviour, evidence reference, confidence.

---

## Stage 13 — Timeline construction

**Objective.** Order events in UTC to expose preparation lead time and campaign tempo.

**Questions.** When was the domain registered, the certificate issued, DNS first observed,
the mail sent, the first click, the site taken down? What is the gap between registration
and weaponisation?

**Evidence collected.** Every dated artefact, normalised to UTC with its source noted.

**Sources.** RDAP dates, CT timestamps, `Received` header times, passive-DNS first-seen,
gateway logs.

**Limitations.** Timestamp sources have different granularity and trustworthiness: attacker-
controlled `Date` headers can be arbitrary, CT timestamps are reliable, passive-DNS
first-seen depends on sensor coverage and is a *lower bound* on existence. Never merge
sources of different reliability into one line without annotation.

**Output.** UTC timeline table with per-entry source and reliability.

---

## Stage 14 — Confidence assessment

**Objective.** State how strongly the evidence supports each judgement, and why.

**Questions.** How many independent sources support this? Is the supporting evidence direct
or circumstantial? Does a plausible alternative remain unexcluded? Would the judgement
survive one piece of evidence being wrong?

**Sources / method.** See [confidence-assessment.md](confidence-assessment.md).

**Limitations.** Confidence describes the evidential basis, not the analyst's feelings, and
it is not a probability of being right. High confidence on a well-corroborated judgement can
still be wrong; that is why the reasoning is published alongside it.

**Output.** Confidence label with a one-sentence justification per key judgement.

---

## Stage 15 — Intelligence-gap identification

**Objective.** State plainly what is not known, and what would resolve it.

**Questions.** What did I fail to establish? Which gaps materially change the assessment if
filled? Who holds the missing data — the organisation, the hosting provider, a commercial
dataset, law enforcement?

**Limitations.** A report without gaps is a report that has not been examined critically.
Conversely, listing gaps that cannot change any decision is padding — prioritise gaps by
decision impact.

**Output.** Gap table: gap, why it matters, how it could be closed, feasibility.

---

## Stage 16 — Reporting

**Objective.** Deliver the findings in the form each audience can act on.

**Questions.** Who decides what, based on this? What does a SOC analyst need at 03:00? What
does a CISO need in one page?

**Limitations.** Technical completeness and executive clarity are different products, not
the same product at different lengths. Compressing the technical report does not yield an
executive brief; the brief answers different questions. See
[templates/](../templates/).

**Output.** Technical report; executive one-page brief; machine-readable IOC export.

---

## Stage 17 — Defensive recommendations

**Objective.** Convert intelligence into changes in defensive posture.

**Questions.** What blocks now? What detections persist after the infrastructure dies? What
hunt should run across historical data? What control gap did this expose?

**Evidence collected.** Recommendation, rationale linked to a specific finding, owner,
expected false-positive cost.

**Limitations.** Recommendations that ignore operational cost get ignored. Blocking an entire
ASN because one phishing site lived there is technically simple and organisationally
reckless — state the collateral impact honestly. Indicator blocks decay quickly; behavioural
detections and control changes are the durable deliverable.

**Output.** Prioritised recommendation table; detection logic; hunt queries. See
[soc-integration/](../soc-integration/).
