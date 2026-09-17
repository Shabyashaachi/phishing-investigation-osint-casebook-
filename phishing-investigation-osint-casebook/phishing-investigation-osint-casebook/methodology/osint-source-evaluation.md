# OSINT Source Evaluation

Not all open sources deserve equal weight. This casebook grades every source on two
independent axes, following the admiralty-style convention used widely in European
intelligence and law-enforcement reporting.

## Source reliability (A–F)

| Grade | Meaning | Typical examples in phishing OSINT |
|---|---|---|
| **A** | Completely reliable — authoritative, primary, verifiable | Certificate Transparency logs, RDAP from the authoritative registry, DNS from the authoritative nameserver |
| **B** | Usually reliable — minor doubts, good track record | Established registries (RIPE NCC), major CERT advisories, well-maintained passive-DNS providers |
| **C** | Fairly reliable — some inconsistency | Aggregated reputation services, crowd-contributed blocklists with moderation |
| **D** | Not usually reliable — significant doubt | Unmoderated crowd submissions, scraped aggregator sites of unknown provenance |
| **E** | Unreliable — history of inaccuracy | Low-quality "threat lookup" sites that re-publish stale data without dates |
| **F** | Reliability cannot be judged | New or opaque source with no methodology statement |

## Information credibility (1–6)

| Grade | Meaning |
|---|---|
| **1** | Confirmed by independent sources |
| **2** | Probably true — consistent with other known information |
| **3** | Possibly true — plausible, not corroborated |
| **4** | Doubtful — not corroborated and somewhat inconsistent |
| **5** | Improbable — contradicted by other information |
| **6** | Truth cannot be judged |

A rating of **B2** means a usually-reliable source reporting something probably true. **A1**
is reserved for primary evidence corroborated independently.

## Questions to ask of any source

1. **What is the methodology?** Does the provider explain how data is collected? Opaque
   scoring cannot be reasoned about.
2. **When was this collected?** Undated OSINT is near-worthless in phishing work, where
   infrastructure lives for days. A record without a first-seen/last-seen is a red flag.
3. **Is it primary or derived?** Many services re-publish the same upstream feed. Derived
   sources do not corroborate each other.
4. **What is its coverage bias?** Passive-DNS sensor placement, CT log participation and
   scanner geography all create blind spots. Absence in the dataset ≠ absence in reality.
5. **Does querying it cost me operational security?** Some lookups are passive; others notify
   the operator or publish my query.
6. **Is the free tier truncating results?** A "no results" from a rate-limited free tier is
   not a negative finding.

## Independence check

Before treating two sources as corroborating, ask whether they share an upstream. In
practice:

- Multiple reputation engines inside one aggregator are **not** independent.
- A domain-intelligence site displaying WHOIS is showing you the registry's data — the
  registry is the source, not the site.
- Two passive-DNS providers with overlapping sensor networks are partially dependent.

Corroboration means *different observation paths reaching the same conclusion*.

## Negative findings

"No results" is a finding and must be recorded with its own reliability caveat:

> No Certificate Transparency entries were found for `x[.]example` as of 2026-02-11T10:03Z
> (crt.sh, identity search). This indicates no publicly-logged certificate was issued for
> this exact name; it does not exclude a certificate for a parent wildcard, a
> non-logged certificate, or use of the domain without TLS. **Source A, credibility 2.**

Stating what a negative finding does *not* exclude is as important as the finding itself.
