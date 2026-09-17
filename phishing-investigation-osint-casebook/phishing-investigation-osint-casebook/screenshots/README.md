# Screenshots — deliberately empty

This directory contains no images, and that is a decision rather than an omission.

Every scenario in this casebook is synthetic. Fabricating screenshots of tool output would
mean manufacturing evidence that looks like it came from a real query, which is exactly the
habit this repository argues against. Reconstructed *text* output is labelled as such and is
readable as reconstruction; a fabricated screenshot is not.

## What would be captured in a real investigation

| Stage | Capture | Why |
|---|---|---|
| Triage | Message as rendered in the client | Shows what the user actually saw, including display-name spoofing |
| Header analysis | Raw header block (text, not image) | Images are not searchable or quotable |
| URL analysis | Sandbox screenshot of the landing page | Visual imitation quality; branding fidelity |
| URL analysis | Sandbox request-chain view | The redirect chain and the server-side requests |
| Domain intelligence | RDAP JSON response | Structured, timestamped |
| Certificate | CT log result set | Issuance timestamps and SAN lists |
| Correlation | Graph view with per-edge annotation | The reasoning, not just the shape |

## Capture rules that would apply

- UTC timestamp and the exact query visible in frame.
- Redact recipient identifiers, internal hostnames and any personal data **before** saving,
  not before publishing — redaction after the fact tends not to happen.
- Never capture a credential entry field with anything typed into it.
- Store alongside the text output; a screenshot is a supplement to searchable evidence, never
  a replacement.
