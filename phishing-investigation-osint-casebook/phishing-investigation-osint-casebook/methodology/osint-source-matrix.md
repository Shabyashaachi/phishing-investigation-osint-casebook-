# OSINT Source Matrix

Sources used or recommended in this casebook, grouped by investigative stage.

> **Verify before relying on the access column.** Free tiers, rate limits and registration
> requirements change frequently and without notice. The capability descriptions below are
> stable properties of each service; the access terms are indicative and should be checked
> against the provider's own documentation at time of use. Nothing here describes an API
> endpoint or feature the author has not seen documented by the provider.

Priority is given to free and freely accessible resources.

---

## Email analysis

| Source | Purpose | What it reveals | Example use | Limitations | Free | Registration |
|---|---|---|---|---|---|---|
| Raw message source | Primary evidence | Full header chain, MIME structure, true link targets | Read `Received` hops bottom-up to find the injection point | Headers above the first trusted hop are attacker-controllable | Yes | No |
| `dig TXT <domain>` | SPF record retrieval | Authorised senders, include chains | Verify whether the sending IP is within the domain's SPF | SPF pass ≠ benign; attacker domains publish valid SPF | Yes | No |
| `dig TXT _dmarc.<domain>` | DMARC policy | Policy (`none`/`quarantine`/`reject`), alignment mode, `rua` | Establish whether a spoof would have been rejected | Reveals victim posture, not attacker intent | Yes | No |
| `dig TXT <selector>._domainkey.<domain>` | DKIM public key | Selector existence, key | Check whether a claimed DKIM selector exists for the domain | Absent selector may mean rotation, not forgery | Yes | No |
| MHA / public header analysers | Header parsing convenience | Structured hop table, delay per hop | Rapid first pass on a long chain | Pasting headers into third-party sites discloses recipient data — pseudonymise first | Yes | No |
| Mail gateway search | Scope determination | Recipient count, delivery status, similar messages | Establish whether this is one message or a wave | Organisation-internal, not OSINT | n/a | n/a |

## URL analysis

| Source | Purpose | What it reveals | Example use | Limitations | Free | Registration |
|---|---|---|---|---|---|---|
| urlscan.io | Sandboxed URL rendering | Redirect chain, DOM, resources, screenshot, IP/ASN of each request | Submit the landing URL privately, read the request chain | Public submissions are visible to anyone, including the operator; kits cloak against known scanners | Yes (tiered) | For private scans |
| VirusTotal (URL) | Multi-engine reputation + relations | Engine verdicts, first/last analysis, related domains and files | Sanity-check reputation and pivot on relations tab | Engines are not independent; low detection early in a campaign is normal | Yes (tiered) | For API |
| Hybrid Analysis / any sandbox | Behavioural detonation | Network calls, dropped content | Detonate an attachment-linked payload | Anti-analysis and geofencing are common | Yes (tiered) | Yes |
| CyberChef | Decoding | Base64/URL/hex layers, obfuscated redirect parameters | Decode a `?u=` parameter carrying the real destination | Client-side only; no intelligence of its own | Yes | No |
| Manual decomposition | Ground truth | True registrable domain vs. brand string in path/subdomain | Distinguish `bank.example.attacker[.]test` from `bank[.]example` | Requires knowing the public suffix list | Yes | No |

## Domain intelligence

| Source | Purpose | What it reveals | Example use | Limitations | Free | Registration |
|---|---|---|---|---|---|---|
| RDAP (registry/registrar endpoints, `rdap.org` bootstrap) | Structured registration data | Creation/updated/expiry, registrar, status codes, nameservers, abuse contact | Retrieve creation date as a lower bound on campaign preparation | EU registrant data is redacted by default under GDPR; redaction carries little evidential weight | Yes | No |
| `whois` (CLI) | Legacy registration data | Same fields, unstructured; some TLDs still richer than RDAP | Cross-check RDAP where registry RDAP is thin | Inconsistent formats; rate limited | Yes | No |
| ICANN Lookup | Registrar verification | Registrar identity, IANA ID, abuse contact | Identify where to send a takedown request | No historical data | Yes | No |
| Registry-specific WHOIS (e.g. national ccTLD registries) | Authoritative ccTLD data | Fields not exposed via generic RDAP | European ccTLD investigations | Varies by registry; some require CAPTCHA | Yes | Varies |

## DNS

| Source | Purpose | What it reveals | Example use | Limitations | Free | Registration |
|---|---|---|---|---|---|---|
| `dig` / `nslookup` | Live resolution | A/AAAA/MX/NS/TXT/CNAME, TTL, authoritative server | Establish current hosting and whether MX exists | Snapshot only; timestamp every query in UTC | Yes | No |
| DNSDumpster | Quick DNS overview | Record set, discovered hosts, simple map | Fast orientation on an unfamiliar domain | Limited depth; coverage varies | Yes | Sometimes |
| SecurityTrails | Historical DNS and WHOIS | Record history, reverse lookups, associated domains | Establish when hosting changed | Free tier is heavily limited; absence ≠ absence | Tiered | Yes |
| Mnemonic Passive DNS / Validin / other pDNS | Resolution history | First-seen / last-seen pairs of name↔IP | Determine when a domain began resolving | Sensor coverage bias; first-seen is a lower bound | Tiered | Varies |
| DNS Dumpster / crt.sh cross-use for subdomains | Subdomain discovery | Names observed in CT rather than brute-forced | Enumerate siblings without touching the target | Misses names never given a logged certificate | Yes | No |

## IP / ASN

| Source | Purpose | What it reveals | Example use | Limitations | Free | Registration |
|---|---|---|---|---|---|---|
| RIPEstat | European allocation context | Prefix, holder, allocation history, routing, abuse contact | Primary for RIPE-region address space | Strongest for RIPE; other RIRs via their own services | Yes | No |
| RDAP for IP / RIR WHOIS | Netblock ownership | Allocation holder, country, abuse contact | Identify who to notify | Holder ≠ current user on resold space | Yes | No |
| bgp.he.net | Routing view | ASN, announced prefixes, peers, adjacent networks | Understand whether hosting is a mainstream cloud or a small reseller | Routing data, not tenancy data | Yes | No |
| Shodan | Host exposure | Open ports, banners, TLS certificate, historical observations | Characterise a server hosting a phishing kit | Free tier is limited; scanning data can be stale | Tiered | Yes |
| Censys | Host and certificate search | Host profiles, certificate-centric search | Pivot from certificate to other hosts presenting it | Free tier query limits | Tiered | Yes |
| Reverse-IP services | Co-hosting count | Other domains on the same address | Test whether an IP is shared hosting before inferring co-ownership | Coverage incomplete; **this is a false-positive minefield** | Tiered | Varies |

## Certificate intelligence

| Source | Purpose | What it reveals | Example use | Limitations | Free | Registration |
|---|---|---|---|---|---|---|
| crt.sh | CT log search | All logged certificates for a name or string, issuance timestamps, SAN lists | Search a brand string to discover lookalike domains before they are used | Only logged certificates; string search returns unrelated matches | Yes | No |
| Censys certificates | Certificate-centric pivoting | Fingerprint, issuer, SANs, hosts presenting it | Find every host presenting a specific certificate | Free tier limits | Tiered | Yes |
| `openssl s_client -connect` | Live certificate retrieval | Presented chain, SANs, validity, fingerprint | Confirm what a host actually serves now | Active connection — touches the target | Yes | No |

## Reputation

| Source | Purpose | What it reveals | Example use | Limitations | Free | Registration |
|---|---|---|---|---|---|---|
| VirusTotal | Aggregated verdicts | Engine results, first submission, relations | Cross-check an indicator | Engines share upstreams; not independent corroboration | Tiered | For API |
| PhishTank / OpenPhish | Phishing URL feeds | Community/automated phishing reports | Check whether a URL is already reported | Coverage skewed to high-volume campaigns | Tiered | Varies |
| Spamhaus / abuse blocklists | Sending-infrastructure reputation | Listing status of IPs/domains | Assess whether the sending IP has a history | Listing lag; delisting happens | Tiered | Varies |
| Google Safe Browsing status | Browser-level blocking | Whether major browsers warn on the URL | Gauge user-facing protection | Binary and opaque | Yes | For API |

## Threat intelligence and community reporting

| Source | Purpose | What it reveals | Example use | Limitations | Free | Registration |
|---|---|---|---|---|---|---|
| CERT-EU / national CSIRT advisories | Regional context | Campaigns targeting European entities, sector advisories | Contextualise a campaign against EU financial services | Publication lag; strategic more than atomic | Yes | No |
| ENISA Threat Landscape | Strategic framing | Sector-level phishing trends | Frame an executive brief | Annual cadence; not operational | Yes | No |
| MISP communities | Structured indicator sharing | Events, attributes, galaxies | Consume and contribute IOCs in a standard format | Membership required; trust-group dependent | Yes | Yes |
| Vendor blogs / public reporting | TTP detail | Kit analyses, campaign write-ups | Compare observed kit behaviour to documented families | Marketing incentives; variable rigour | Yes | No |

## Passive DNS (summary)

Passive DNS deserves its own note because it is the highest-value and most
misread source in infrastructure work. It records *observed* resolutions from sensor
networks. Therefore:

- **First-seen is a lower bound**, never a creation date.
- **Absence means no sensor observed it** — not that it never resolved.
- Free tiers truncate aggressively; a thin result set is a tooling artefact as often as a
  fact about the world.

## Search engines and archives

| Source | Purpose | Limitations |
|---|---|---|
| Google/Bing advanced operators | Locate references to a domain, kit strings, reused page text | Indexing lag; phishing sites are rarely indexed |
| Wayback Machine | Historical page content | Sparse coverage of short-lived phishing sites |
| GitHub code search | Locate leaked or published kit source | Legal and ethical care required; do not redistribute kits |

## MITRE ATT&CK

| Source | Purpose | Limitations |
|---|---|---|
| ATT&CK Enterprise matrix | Common vocabulary for adversary behaviour | Describes behaviour, not attribution; resist mapping unobserved techniques |
| ATT&CK Navigator | Visualising technique coverage | A colourful layer is not evidence |

## Infrastructure discovery

| Source | Purpose | Limitations |
|---|---|---|
| Favicon hash / HTML structure hashing | Find hosts serving identical kit front-ends | Requires a scanning dataset (Shodan/Censys); common frameworks collide |
| JARM / TLS fingerprinting | Cluster servers by TLS stack configuration | Identifies software stack, not operator — many unrelated hosts match |
| Nameserver reuse | Cluster domains under a niche DNS provider | Worthless for large providers; only selective for small/self-hosted NS |
