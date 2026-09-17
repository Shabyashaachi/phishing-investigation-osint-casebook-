# Diagrams

Mermaid sources. GitHub renders these inline, so they are kept as text rather than images —
diffable, editable, and reviewable.

| Diagram | Location |
|---|---|
| Investigation workflow | [`../README.md`](../README.md) |
| Repository architecture | [`../README.md`](../README.md) |
| Wave 1 domain relationships | [Case 003 §9](../cases/case-003-phishing-domain-investigation/investigation.md) |
| Correlation graph with rejected cluster | [Case 004 §5](../cases/case-004-infrastructure-correlation/investigation.md) |
| Cross-wave campaign graph | [Case 005 §11](../cases/case-005-eu-fin-001-credential-phishing-campaign/investigation.md) |
| CTI-to-SOC pipeline | [`../soc-integration/README.md`](../soc-integration/README.md) |

## Convention

Edge style carries meaning and is consistent across the casebook:

- **Solid** — directly observed record (DNS, header, HTTP response)
- **Dotted** — certificate relationship
- **Double/thick** — inferred or correlational link, always labelled with its confidence
- **Greyed with dashed border** — explicitly rejected relationship, drawn so the exclusion is
  visible

A graph that shows only accepted edges hides the judgement that produced it.
