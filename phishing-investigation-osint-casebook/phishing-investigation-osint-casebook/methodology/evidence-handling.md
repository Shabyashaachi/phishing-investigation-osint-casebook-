# Evidence Handling

Phishing investigations produce evidence that may later support a takedown request, an
insurance claim, a regulatory notification under GDPR Art. 33, or a criminal referral. It is
cheaper to handle everything properly than to decide afterwards which items mattered.

## Principles

1. **Originals are never modified.** Work on copies. The original `.eml` is stored read-only
   with its hash.
2. **Provenance is recorded at acquisition**, not reconstructed later: who obtained it, from
   where, by what method, at what UTC time.
3. **Integrity is verifiable.** SHA-256 for every file, recorded in a manifest that travels
   with the evidence.
4. **Derived artefacts are labelled as derived.** A parsed header table is analysis output,
   not evidence.
5. **Minimise personal data.** Recipient identities, internal hostnames and employee names
   are pseudonymised before anything is published or shared externally. Under GDPR this is
   not optional courtesy; it is data minimisation.

## Acquisition

| Artefact | Correct method | Wrong method |
|---|---|---|
| Phishing email | Export original message (`.eml`/`.msg`) from gateway or client "show original" | Forwarding the message — rewrites headers, destroys routing chain |
| Headers | Copy the complete raw block including all `Received` hops | Screenshot of the client's simplified header view |
| Attachment | Extract to isolated storage, hash before opening | Double-clicking to "see what it is" |
| Web page | Sandbox capture (urlscan.io, isolated VM) with DOM and response chain | Browsing to it from a corporate workstation |
| Tool output | Save raw text output with the exact command and UTC timestamp | Screenshot only, undated |

## Manifest format

Each case carries `evidence/MANIFEST.md`:

| File | SHA-256 | Acquired (UTC) | Method | Custodian | Notes |
|---|---|---|---|---|---|
| `eu-fin-001-sample.eml` | `SYNTHETIC-…` | 2026-02-11T08:52Z | Gateway export | Analyst | Original, unmodified |
| `dns-queries.txt` | `SYNTHETIC-…` | 2026-02-11T09:14Z | `dig` capture | Analyst | Derived — live query record |

In this repository all hashes are marked `SYNTHETIC` because the artefacts are fabricated.
A real case would carry genuine digests.

## Timestamps

Everything in UTC, ISO 8601, with the source of the timestamp noted. A `Received` header
timestamp written by an attacker-controlled host is not the same class of evidence as a
Certificate Transparency log entry, and the timeline must say so.

## Handling live infrastructure safely

- Prefer **passive** collection. Passive DNS, CT logs and RDAP do not touch the adversary.
- Active interaction (resolving, fetching, sandboxing) tips off an operator who monitors
  their logs, and may cause a campaign to rotate infrastructure — destroying the evidence you
  were about to collect. Decide deliberately.
- Never submit a URL containing a **victim-unique token** to a public sandbox. Public
  submissions are visible to others, including the operator, and the token may identify the
  recipient. Substitute the token or use a private submission.
- Never authenticate to a phishing page, not even with fabricated credentials. It is
  unnecessary for the analysis and it pollutes the operator's dataset in ways that can
  interfere with a later investigation.

## Retention and publication

| Class | Internal retention | Publishable |
|---|---|---|
| Original message with real recipient data | Per organisational policy | No |
| Pseudonymised headers | Case lifetime | Yes, with defanging |
| Infrastructure IOCs | Per IOC lifecycle | Yes, defanged |
| Malware samples | Controlled storage | No — reference by hash only |
| Screenshots containing PII | Per policy | Only after redaction |

## Defanging convention

Applied to **all** human-readable output in this repository:

```
http://bad.example/login   →   hxxp://bad[.]example/login
198.51.100.44              →   198.51.100[.]44
user@bad.example           →   user[@]bad[.]example
```

Machine-readable exports under [`iocs/`](../iocs/) keep a `defanged` column so that a
consumer parses fanged values only when they intend to.
