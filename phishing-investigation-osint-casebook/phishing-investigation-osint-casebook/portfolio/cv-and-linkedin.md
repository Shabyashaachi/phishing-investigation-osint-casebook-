# CV Entries and LinkedIn Content

All wording describes **independent security research / a portfolio project**. Nothing here
implies employment, client work, or handling of real incidents. That restraint is itself
assessed positively by most European hiring managers, and its absence is a common reason
portfolio claims get probed hard in interview.

---

## CV — Version 1 (one-line ATS)

> **Phishing Investigation & OSINT Casebook** — Independent security research project: five
> documented phishing investigations covering email header analysis, URL and domain OSINT,
> DNS/IP/ASN and Certificate Transparency pivoting, infrastructure correlation, IOC
> management, MITRE ATT&CK mapping and CTI reporting. [github.com/…]

*Keyword-dense for ATS parsing while remaining accurate.*

---

## CV — Version 2 (two-bullet professional)

> **Phishing Investigation & OSINT Casebook** — *Independent security research* · [link]
>
> - Designed and documented five progressively complex phishing investigations against
>   synthetic European scenarios, covering email authentication analysis (SPF/DKIM/DMARC
>   alignment), URL and redirect analysis, RDAP/DNS/Certificate Transparency intelligence,
>   and infrastructure correlation with explicit confidence and alternative explanations.
> - Produced CTI deliverables for two audiences — technical analyst reports and one-page
>   executive briefs — with structured IOC exports, evidence-constrained MITRE ATT&CK
>   mappings, and defensive recommendations translated into synthetic SIEM detections and
>   threat-hunting questions.

---

## CV — Version 3 (three-bullet technical)

> **Phishing Investigation & OSINT Casebook** — *Independent security research* · [link]
>
> - Built a seventeen-stage phishing investigation methodology (triage → evidence
>   preservation → email/URL/domain/DNS/IP/certificate analysis → correlation → TTP mapping →
>   confidence assessment → reporting), applied consistently across five case studies with
>   synthetic, defanged data suitable for public release.
> - Investigated a flagship credential-phishing campaign against a fictional EU
>   financial-services firm, identifying adversary-in-the-middle reverse-proxy infrastructure
>   capable of session-token capture and MFA relay, and demonstrating that the proximate cause
>   of delivery was a mail-gateway policy override rather than attacker sophistication.
> - Applied structured analytic technique throughout — selectivity-based infrastructure
>   correlation with documented rejected hypotheses, confidence levels with justification,
>   prioritised intelligence gaps, and IOC actions weighted by collateral cost (shared-hosting
>   and spoofed victim-owned indicators explicitly excluded from blocking).

---

## LinkedIn Post 1 — Introducing the casebook

> I've published a **Phishing Investigation & OSINT Casebook** — five documented
> investigations taking a phishing artefact from initial triage through to actionable
> intelligence.
>
> What I wanted to build was not another tool walkthrough. Running a WHOIS query is easy. The
> part that's hard, and the part nobody can show you in a certificate, is deciding what an
> observation actually licenses you to conclude.
>
> So every significant finding in the repository is written as: observation → evidence →
> interpretation → **alternative explanation** → assessment → confidence → intelligence gap.
>
> That fourth field changed how I work. A domain registered four days ago, behind privacy
> protection, on a cheap registrar — that describes a huge population of entirely legitimate
> domains. It's a weak prior, not a finding. Writing down the innocent explanation forces you
> to go find something that actually discriminates.
>
> The casebook covers email header and SPF/DKIM/DMARC analysis, URL and redirect
> investigation, RDAP and DNS intelligence, Certificate Transparency as a discovery source,
> and infrastructure correlation. Each case ends with IOCs, evidence-constrained ATT&CK
> mappings, confidence-rated judgements, intelligence gaps, and defensive recommendations.
>
> All scenarios are synthetic, all indicators defanged, all domains and IPs in reserved
> ranges. Independent research — not client work.
>
> I'd particularly welcome disagreement from practising CTI and SOC analysts about the
> confidence levels. That's the most useful review this kind of work can get.
>
> [link]
>
> #ThreatIntelligence #OSINT #Phishing

---

## LinkedIn Post 2 — The flagship investigation

> The flagship case in my phishing casebook is the one where the obvious conclusion was
> wrong, and I've kept the near-miss in the write-up.
>
> Scenario: eleven staff at a fictional EU financial-services firm receive an SSO
> re-authentication email. Nine work in Treasury or Payments. Two authenticate.
>
> The first surprise was in the headers: `spf=pass`, `dkim=pass`, `dmarc=fail`. The attacker
> owned their sending domain and had configured it properly — valid SPF, working DKIM. They
> pass authentication for *their own* domain. Only DMARC alignment catches it, and the mail
> gateway didn't enforce the quarantine policy because of an allow-list entry.
>
> Which means the proximate cause of delivery was a configuration override on our side, not
> the attacker's skill. That's the finding I'd want on a CISO's desk.
>
> The second surprise mattered more. I nearly characterised the landing page from the
> screenshot: a login clone, collect credentials, done. Reading the sandbox network capture
> instead showed **server-side** requests from the phishing host to the genuine identity
> provider, a relayed MFA push, and a session cookie coming back to the proxy.
>
> Adversary-in-the-middle. Not a clone — a reverse proxy. The target was the session token.
>
> That single observation changes the response entirely. A password reset does not invalidate
> a stolen session token. If I'd trusted the screenshot, the recommendation would have been a
> password reset, and the compromise would have stayed live.
>
> It also changes what you recommend long-term. Blocking domains raises cost. Push-notification
> MFA gets relayed. Only origin-bound authentication — FIDO2/WebAuthn — actually removes the
> technique, because a credential relayed through a proxy can't be replayed against a
> different origin.
>
> Synthetic scenario, defanged indicators, independent research.
>
> [link]
>
> #ThreatIntelligence #IncidentResponse #Phishing

---

## LinkedIn Post 3 — What I learned about infrastructure correlation

> The most useful thing I learned building my phishing casebook: **infrastructure correlation
> is mostly about deciding what *not* to link.**
>
> Pivoting is cheap. Domain → IP → ASN → certificate → nameserver, and within twenty minutes
> you have an impressive-looking graph. Most of the edges in that graph are wrong.
>
> The property that decides an edge's weight is **selectivity** — how many unrelated parties
> could produce the same observation by coincidence?
>
> - Same certificate: only entities that proved control of every name at issuance. Strong.
> - Same dedicated IP with two tenants: few. Moderate.
> - Same shared IP with 900 tenants: hundreds of unrelated customers. **Worthless.**
> - Same registrar: millions. Worthless.
> - Same large cloud ASN: millions. Worthless.
>
> In one of my cases the strongest-*looking* signal was 900+ domains sharing an IP with the
> phishing site. I threw all of it out. What I kept instead was a three-minute gap between two
> certificate issuances in the CT logs — corroborated by shared nameservers and an identical
> naming convention.
>
> Two further things I'd tell my earlier self:
>
> **Independence matters as much as selectivity.** Four weak links only combine if the
> adversary chose them separately. "Same IP, same /24, same ASN" is one link counted three
> times.
>
> **Write down what would break your own hypothesis.** In one case I claimed two domains were
> related partly because they shared an unusual nameserver — then checked whether that
> nameserver was simply the registrar's default. It wasn't, so the link held. If I hadn't
> checked, I'd have published a coincidence as evidence.
>
> My correlation diagrams now draw the rejected cluster explicitly. A graph that shows only
> the accepted edges hides the judgement that produced it.
>
> Synthetic data throughout. Independent research.
>
> [link]
>
> #CTI #OSINT #ThreatIntelligence
