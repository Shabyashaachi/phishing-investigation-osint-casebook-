# Interview Preparation

Twenty questions a European SOC or CTI interviewer could reasonably ask about this
repository. Each entry gives what is being tested, a strong answer, a likely follow-up, and
the mistake that costs candidates the point.

---

### 1. Walk me through what SPF, DKIM and DMARC each actually check.

**Testing:** whether you understand the mechanisms or just recite the acronyms.

**Strong answer.** SPF checks whether the sending IP is authorised for the **envelope**
sender (`MAIL FROM` / `Return-Path`) — not the `From:` header the user sees. DKIM verifies a
cryptographic signature over selected headers and body, proving the signing domain
authorised the message and it wasn't modified in transit. DMARC ties either mechanism to the
**visible `From:` domain** through alignment, and publishes a policy for what the receiver
should do on failure. The essential point is that SPF and DKIM can both pass for a domain the
attacker owns. Only DMARC alignment connects authentication to the identity the user reads.

**Follow-up:** *"So how did a message in your flagship case pass SPF and DKIM but still get
flagged?"* — It authenticated for `mail-relay-eu[.]invalid`, the attacker's own domain, which
had valid SPF and a working DKIM selector. Neither aligned with the spoofed `From:` domain, so
DMARC failed.

**Common mistake:** saying "SPF checks the sender" without distinguishing envelope from
header, or treating an SPF pass as evidence a message is legitimate.

---

### 2. A message shows `dmarc=fail (p=quarantine dis=none)`. What does that tell you?

**Testing:** whether you read disposition, not just result.

**Strong answer.** The domain published a quarantine policy, DMARC evaluation failed, and the
receiving gateway **did not apply** the policy — `dis=none` means the disposition taken was
none. Something overrode it: typically an allow-list entry, a local policy exception, or a
forwarding exemption. In my flagship case that override was the proximate cause of delivery.
It's the finding I'd put in front of leadership, because it's the one thing entirely within
the organisation's control.

**Follow-up:** *"How would you detect this systematically?"* — Alert on DMARC failure for
own-domain `From:` where disposition is neither quarantine nor reject. Alerting on DMARC
failure alone is noise; alerting on unenforced failure is a short, actionable list.

**Common mistake:** reading `p=quarantine` and assuming the message was quarantined.

---

### 3. How do you read a `Received` header chain?

**Testing:** header fundamentals and trust boundaries.

**Strong answer.** Bottom-up — the oldest hop is last in the file. The critical task is
identifying the **first hop written by infrastructure you control or trust**. Everything below
that is asserted by the sender and can be entirely fabricated; attackers routinely prepend
fake hops to suggest an innocuous origin. In my cases the injection point is the sending IP
recorded by the organisation's own MX. I also check timestamp progression for unexplained
delays and look for whether the authorised mail provider appears anywhere in the chain at all.

**Follow-up:** *"What if you only have a forwarded copy?"* — Then the original chain is gone;
forwarding rewrites it. I'd state that limitation explicitly in the report, because the entire
header analysis inherits it, and request the original from the gateway.

**Common mistake:** treating the lowest `Received` hop as the true origin.

---

### 4. Is a newly registered domain evidence of malicious intent?

**Testing:** indicator discipline — this is the single most reliable discriminator between
junior candidates.

**Strong answer.** No. Several hundred thousand domains are registered daily, overwhelmingly
legitimate. Age is a weak prior that shifts probability slightly and proves nothing. What was
informative in my cases was the **sequence and spacing**: registration, then certificate
issuance hours later, then DNS, then delivery within seventy-two hours, with no content
development in between. That pattern is hard to produce innocently. The date alone isn't.

**Follow-up:** *"Same question for privacy protection."* — Even weaker. Post-GDPR, redaction
is the default for EU registrants and is applied automatically by most registrars. Reading
intent into it is close to reading nothing at all.

**Common mistake:** stacking weak indicators — new + private + cheap TLD + shared host — and
treating the pile as strong. That description fits an enormous benign population.

---

### 5. Two domains resolve to the same IP. Same operator?

**Testing:** the correlation trap.

**Strong answer.** It depends entirely on the hosting context, which I check before drawing
the edge. On a dedicated server with two tenants it's meaningful. On mass shared hosting with
900+ tenants it's the expected observation for hundreds of unrelated customers and carries no
weight. In Case 004 I rejected exactly that edge. What I kept instead were links with higher
selectivity — a shared certificate, which requires demonstrated control of every name at
issuance.

**Follow-up:** *"What if they're behind Cloudflare?"* — Then I'm looking at the proxy, not the
origin. The observed IP tells me about the CDN's infrastructure. I'd say so in the report
rather than presenting the proxy IP as the adversary's host.

**Common mistake:** "shared IP therefore same actor", which is how innocent websites end up in
published threat reports.

---

### 6. What is Certificate Transparency and how have you used it?

**Testing:** whether CT is understood as a discovery source or just a lookup.

**Strong answer.** CT is a set of append-only public logs of issued certificates. For
investigation it does three things: it gives a reliable issuance timestamp, which is an
excellent timeline anchor; it exposes SAN lists, which reveal sibling hostnames; and — most
valuably — a **string search** on a brand token finds lookalike domains that have been staged
but never used in any message. In my flagship case that found two prepared domains with no
prior sighting anywhere, two days before delivery.

**Follow-up:** *"Limitations?"* — Only publicly logged certificates appear; a substring match
is not a relationship, and my brand search returned legitimate organisations using the same
word, which I excluded by inspection. Issuance from a free automated CA also says nothing
about intent, since that's the default for the whole modern web.

**Common mistake:** presenting every crt.sh substring hit as related infrastructure.

---

### 7. Explain the difference between RDAP and WHOIS.

**Testing:** current practice.

**Strong answer.** RDAP is the structured, JSON-based successor to WHOIS, with standardised
fields, proper internationalisation, bootstrapping to the authoritative server, and
differentiated access. WHOIS returns unstructured text whose format varies by registry. I
default to RDAP because it parses reliably and points me at the authoritative source, but I
still fall back to WHOIS for some ccTLDs where the registry exposes richer data that way.

**Follow-up:** *"What can't either give you now?"* — Registrant identity, in most European
cases. GDPR redaction is the default, and that's a genuine intelligence gap I record rather
than speculate about. Closing it requires legal process, not OSINT.

**Common mistake:** treating redaction as suspicious rather than as the norm.

---

### 8. Why does ASN matter in a phishing investigation, and how far does it get you?

**Testing:** proportionate use of network context.

**Strong answer.** ASN tells me the class of hosting and who to contact for abuse — a
bulletproof host, a mainstream cloud, a small VPS reseller, a CDN. That's operationally
useful. What it is *not* is a relational link: millions of unrelated entities share a large
provider's ASN. In my casebook, ASN is recorded as context and explicitly carries zero
correlation weight.

**Follow-up:** *"Would you ever block at ASN level?"* — Very rarely, and only with the
collateral impact stated explicitly to whoever signs it off. Blocking a mainstream cloud ASN
because one phishing site lived there is technically trivial and organisationally reckless.

**Common mistake:** presenting a shared ASN as evidence of a shared actor.

---

### 9. How do you decide when infrastructure correlation is strong enough to assert?

**Testing:** structured analytic technique.

**Strong answer.** Two tests. **Selectivity** — how many unrelated parties could produce this
observation by coincidence? A shared certificate is highly selective; a shared registrar is
not. **Independence** — do multiple weak links represent separate choices by the operator?
Four independent weak links can combine; "same IP, same /24, same ASN" is one link counted
three times. I won't assert common operation from a single non-selective link, and for each
accepted edge I record what would falsify it.

**Follow-up:** *"Give me an example where you actually ran the falsification check."* — I
linked two domains partly on a shared unusual nameserver, then checked whether that
nameserver was simply the registrar's default. It wasn't, so the link held. If it had been,
the edge would have collapsed to negligible.

**Common mistake:** presenting a big graph with no per-edge reasoning.

---

### 10. How do you express confidence, and what does "high confidence" mean to you?

**Testing:** analytic rigour and self-awareness.

**Strong answer.** Confidence describes the **evidential basis for a judgement**, not my
feelings and not a probability of being right. High means multiple independent sources or
direct primary evidence, plausible alternatives tested and excluded, and the judgement
surviving any single item being wrong. Medium means credible but partly circumstantial with a
surviving alternative. Low means single-source or fragmentary — included because it may
matter, flagged as weak. I grade source reliability A–F and information credibility 1–6
separately from analytic confidence, because they're different axes.

**Follow-up:** *"Can a high-confidence judgement be wrong?"* — Yes, and that's why I publish
the reasoning alongside it, and why my confidence assessments are versioned. In one case I
recorded lowering a judgement from High to Medium after new hosting-tenancy data reinstated an
alternative explanation I'd thought excluded.

**Common mistake:** using high confidence to mean "I'm sure", or inflating confidence to sound
authoritative.

---

### 11. Why does your flagship case offer no attribution?

**Testing:** discipline under the temptation to overclaim.

**Strong answer.** Because the evidence doesn't support it. The observed tradecraft —
reverse-proxy kit, aged sending domain, brand-lookalike naming — is available in commodity and
open-source form and used by a wide range of financially motivated actors. Nothing I observed
was distinctive enough to narrow it. An OSINT-only phishing investigation very rarely produces
attributable evidence, and claiming otherwise would undermine everything else in the report.

**Follow-up:** *"What would you need to attribute?"* — Distinctive tooling with a limited user
base, operational-security errors linking to known personas, victimology matching a documented
campaign, or corroboration from a source with visibility I don't have. Even then I'd attribute
to a cluster with a confidence level, not name a group.

**Common mistake:** naming an APT because the target sector matches something in a vendor blog.

---

### 12. How do you avoid false positives in IOC handling?

**Testing:** operational judgement.

**Strong answer.** Three habits. First, enrich before enforcing — an IP never reaches a
blocklist without a tenancy figure attached. Second, separate `recommended_action` from
`confidence`: action depends on confidence *and* collateral cost. Third, age indicators
aggressively and retire them, because an expired phishing domain re-registered by an innocent
party becomes a permanent false positive if it's left in a blocklist.

**Follow-up:** *"Give me two indicators from your casebook that you deliberately didn't
block."* — A shared-hosting IP with 900+ tenants, marked `shared_infrastructure`, monitor
only. And the spoofed sender address, which is the **victim organisation's own** address —
blocking it would block internal mail. That second one catches people who export "all observed
indicators" without thinking.

**Common mistake:** treating an IOC list as a blocklist.

---

### 13. How would you operationalise these findings in a SIEM?

**Testing:** whether CTI output connects to defensive reality.

**Strong answer.** Atomic indicators go to enforcement with expiry dates and exclusions. The
more valuable output is behavioural: in my flagship case the highest-value detection is DMARC
failure on an internal `From:` domain **that wasn't enforced**, which surfaces both the attack
and the misconfiguration. I'd also correlate mail delivery against proxy connections to
recently-registered domains, and — for the AiTM element — alert on successful MFA sign-ins
from hosting-provider ASNs inconsistent with the user's baseline.

**Follow-up:** *"Why that last one?"* — In an AiTM relay the authentication reaches the IdP
from the proxy host, not the user's device. So you see a successful MFA result originating
from datacentre address space. That's the signature, and it survives every infrastructure
change the operator makes.

**Common mistake:** producing detections that are just the IOC list restated as queries.

---

### 14. Which threat-hunting questions would you run after this campaign?

**Testing:** hunting as hypothesis-driven work.

**Strong answer.** I'd start with hypotheses, not indicators. Did any earlier wave land
undetected — any connection to campaign domains in the preceding ninety days? Which messages
spoofed an internal domain, failed DMARC, and were delivered anyway? Which successful MFA
sign-ins came from hosting ASNs? Were new authentication methods or mailbox rules created
within an hour of an unusual sign-in? And, to close an intelligence gap: do the recipients
share a role attribute that's inferable from public sources?

**Follow-up:** *"Which of those survives the infrastructure being taken down?"* — All of them
except the first. That's the point of framing hunts as questions rather than indicator
searches.

**Common mistake:** "hunting" that is retrospective IOC matching.

---

### 15. Your landing page turned out to be an AiTM proxy. How did you establish that, and why does it matter?

**Testing:** the flagship's central technical finding.

**Strong answer.** The sandbox network capture showed requests to the genuine identity
provider originating **server-side** from the phishing host, an MFA push challenge relayed
back to the client, and a session cookie returned to the proxy. A static clone can't do that:
its asset requests originate client-side and it can't complete an MFA challenge. It matters
because the target is the session token, not the password — so push-notification and OTP MFA
are relayed, and a password reset alone leaves the compromise live. Session revocation is
mandatory.

**Follow-up:** *"What control actually stops it?"* — Origin-bound authentication, FIDO2 or
WebAuthn. The credential is cryptographically tied to the origin, so a relay through a
different origin can't replay it. Everything else raises cost; this removes the technique.

**Common mistake:** characterising the page from a screenshot. I nearly did, and I kept that
near-miss in the write-up.

---

### 16. Walk me through your investigation methodology.

**Testing:** whether there's a repeatable process or ad-hoc tool use.

**Strong answer.** Seventeen stages: preparation, triage, evidence preservation, first-pass
IOC extraction, then email, URL, domain, DNS, IP/ASN and certificate analysis, then
correlation, TTP identification, timeline, confidence assessment, gap identification,
reporting and defensive recommendations. Stages six to eleven loop — a certificate found at
stage ten often sends me back to stage seven with a new domain. I stop when new pivots stop
producing new indicators. Every stage documents its objective, limitations and expected
output, and the limitations section is the one I'd point an interviewer at.

**Follow-up:** *"Which stage do juniors skip?"* — Evidence preservation, and it can't be
retrofitted. If someone forwards you the email instead of exporting it, the routing chain is
already destroyed.

---

### 17. What are the risks of interacting with phishing infrastructure during an investigation?

**Testing:** operational security and ethics.

**Strong answer.** Three. You tip off an operator who monitors their logs, which can trigger
infrastructure rotation and destroy the evidence you were about to collect. You may expose
attributable infrastructure — corporate IP space in their logs. And public sandbox submissions
are visible to anyone, including the operator. In my casebook I default to passive collection
— RDAP, DNS, CT — and where I do sandbox, I substitute victim-unique tokens first, because
submitting them would publish a recipient identifier and signal that the message was reported.

**Follow-up:** *"Would you ever enter credentials into a phishing page?"* — No. It's
unnecessary for the analysis, it pollutes the operator's dataset in ways that can interfere
with a later investigation, and depending on jurisdiction it may not be lawful.

---

### 18. How do you decide which ATT&CK techniques to map?

**Testing:** resistance to framework inflation.

**Strong answer.** Only where a specific observation supports the technique, with the
observation and evidence recorded beside each mapping. In my flagship case I mapped eight and
explicitly listed what I **didn't** map and why — most notably T1078 Valid Accounts, because I
observed the capture position but never observed the credentials being used. That's the one a
less disciplined report includes because it fits the narrative and makes the incident sound
worse.

**Follow-up:** *"Why does over-mapping matter?"* — It's a credibility problem. If a reviewer
finds one unsupported mapping, they'll reasonably doubt the other seven. And on the defensive
side it distorts coverage analysis — you end up building detections for behaviour that never
occurred.

---

### 19. How do you write for a CISO versus a SOC analyst?

**Testing:** communication, and whether you understand these are different products.

**Strong answer.** They answer different questions, so compressing the technical report
doesn't produce an executive brief. The CISO version explains what happened in plain language,
what the business consequence is, what we know and don't know, and what to do — and it names
our own failures first, because leadership should hear that from security rather than discover
it. My executive brief explains AiTM without using the term: the fake site sat between the
employee and the real login system, which means approving the phone prompt didn't protect them
and changing the password doesn't end the attacker's access. The analyst version keeps the
header blocks, the per-edge correlation reasoning and the query logic.

**Follow-up:** *"What's the one-sentence version for leadership?"* — A configuration override
let the message through, and our current MFA couldn't have stopped what came next — so the two
fixes that matter are the gateway setting and security keys for staff who can move money.

---

### 20. What are the weaknesses of this project?

**Testing:** self-awareness. Interviewers ask this deliberately, and a defensive answer costs
more than any technical gap.

**Strong answer.** The data is synthetic, which is the honest limitation: constructed scenarios
smooth over the contradictory evidence, decaying infrastructure and incomplete headers you get
in real telemetry. I've had no access to commercial passive DNS, so historical resolution
analysis is thin and I flag that as a gap rather than working around it. And it demonstrates
method rather than a track record — I haven't handled a live incident under time pressure with
a stakeholder asking for an answer every ten minutes.

**Follow-up:** *"So why should I value it?"* — Because the method transfers and the reasoning
is inspectable. You can read any judgement in the repository, see the evidence it rests on, see
the alternative I rejected and why, and disagree with me specifically. That's harder to fake
than tool output, and it's the thing I'd actually be doing on day one.

**Common mistake:** claiming there are no weaknesses, or conceding so much that the work sounds
worthless.
