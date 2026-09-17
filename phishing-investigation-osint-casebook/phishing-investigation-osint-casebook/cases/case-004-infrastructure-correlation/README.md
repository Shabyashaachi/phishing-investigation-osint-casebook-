# Case 004 — Infrastructure Correlation

**Focus:** deciding which infrastructure elements genuinely belong to one operation — and
documenting the ones that do not.

**What this case demonstrates**

- A **selectivity table** ranking link types by how many unrelated entities could produce the
  same observation by coincidence
- An **edge register** where every candidate relationship — accepted *and* rejected — records
  indicator A, indicator B, relationship, evidence, source, selectivity, alternative
  explanation and confidence
- Explicit falsification: each accepted edge states what would break it, and one edge records
  the check that was actually performed
- The distinction between *independent* weak links (which combine) and *dependent* ones
  (same IP and same ASN are one link counted twice)
- Correct classification of **abused third-party infrastructure**, which is an abuse-report
  target and must never enter a blocklist

**Rejected edges are drawn in the graph.** A correlation diagram showing only accepted links
hides the analyst's judgement; showing the exclusions is what makes it reviewable.

**Files:** [`investigation.md`](investigation.md) · [`evidence/`](evidence/)

All data is synthetic and defanged. See [DISCLAIMER](../../DISCLAIMER.md).
