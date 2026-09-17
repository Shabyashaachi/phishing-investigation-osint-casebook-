# Hiring-Manager Review

*Written in the voice of a European cybersecurity hiring manager assessing this repository as
part of an application for a junior CTI / SOC analyst role. No numerical score is given — a
number would compress judgements that need to stay separate.*

---

## Clearly demonstrated skills

**Analytical discipline.** This is the strongest thing in the repository, and it is the thing
most portfolios lack entirely. The observation → evidence → interpretation → alternative
explanation → assessment → confidence → gap structure is applied consistently, and — crucially
— the alternative explanations are *tested*, not decorative. Case 003 rejects the
strongest-looking signal in the case (900+ domains on a shared IP) and carries the conclusion
on a different piece of evidence. Case 004 records what would falsify each accepted edge, and
one entry documents a falsification check that was actually performed. I would read that and
believe it.

**Indicator restraint.** The explicit treatment of newly-registered, privacy-protected and
cheap-registrar domains as weak priors rather than findings tells me the candidate has
internalised something most juniors take a year of real casework to learn. The "discriminating
power" column in Case 003 §3 is the single best page in the repository.

**Email authentication fluency.** The SPF/DKIM/DMARC treatment is correct and precise,
including the distinction between envelope and header identity, the meaning of alignment
modes, and — the detail that convinced me — reading `dis=none` and identifying the gateway
allow-list override as the proximate cause of delivery. That is a finding about the *defender*,
produced by an investigation aimed at the attacker. It shows the candidate understands what a
report is *for*.

**Operational judgement on IOCs.** Separating `recommended_action` from `confidence` is a
production-grade instinct. The two deliberate non-blocks — shared hosting, and the victim
organisation's own spoofed address — are exactly the two mistakes that generate incidents in
real SOCs. Most candidates hand me a list of everything they saw.

**Communication across audiences.** The executive brief is genuinely a different document, not
a shortened technical report. Explaining adversary-in-the-middle as "the fake site sat between
the employee and the real login system" without using the term, and then drawing the two
consequences the reader actually needs, is better executive writing than I see from some
mid-level analysts. Naming the organisation's own misconfiguration in the brief is the right
call and shows political maturity.

**Ethical handling.** Reserved ranges throughout, defanging convention, victim-token
substitution before sandbox submission, explicit refusal to submit credentials, pseudonymised
recipients, and a clear statement that this is not professional experience. The
synthetic-data conventions table pre-empts the exact objection I would otherwise raise.

**Self-critique.** Case 005 §18 records an anchoring near-miss — the candidate nearly
characterised an AiTM proxy as a static clone from a screenshot — and notes that a faster
analyst would have got it wrong and recommended a password reset that left the compromise
live. Including that is a deliberate choice against self-presentation, and it is the thing
that moved my overall impression most.

---

## Skills that need stronger evidence

**Working with messy, contradictory data.** Every case resolves cleanly. Real investigations
stall: the passive DNS is empty, the headers are truncated because someone forwarded the
message, two sources disagree and neither is obviously wrong, and the answer is "insufficient
evidence" rather than a confidence-rated judgement. I cannot tell from this repository how the
candidate behaves when the evidence does not cooperate. **Suggested fix:** one case built on a
deliberately degraded artefact — forwarded copy, no original, a dead domain, an empty pDNS
result — that terminates in an honest "cannot determine" with a stated collection plan. That
case would be more persuasive than the flagship.

**Tool fluency in practice.** The methodology names the right queries, but every output is
reconstructed. I have no evidence the candidate has actually run `dig`, parsed an RDAP JSON
response, or navigated crt.sh's result noise. This is a reasonable consequence of the ethical
decision to use synthetic data, but it leaves a gap. **Suggested fix:** one short appendix
applying the identical methodology to a *public, already-reported, already-dead* phishing
domain from a published advisory, with real tool output. That is safe, legal, and closes the
gap entirely.

**Detection engineering depth.** The SIEM content is well-reasoned but thin on the realities:
no discussion of tuning, of the field-mapping work between vendors, of the volume these
queries would actually produce, or of what the false-positive rate does to an analyst's
evening. The Sigma rule is structurally correct; I would want to see one rule taken through
iteration. **Suggested fix:** a short note on tuning one detection — what it fired on, what was
excluded, what the residual FP rate was.

**Scale.** Everything here is one artefact at a time. Real phishing work involves triaging
forty reports a shift and deciding which three deserve this treatment. The triage stage exists
in the methodology but is never stress-tested. **Suggested fix:** a one-page triage decision
framework — what makes a report worth an hour rather than four minutes.

---

## Missing evidence

- **No sharing output.** There is a MISP/STIX mapping intention in the IOC schema but no
  actual MISP event or STIX bundle. For a European CTI role where sector CSIRT and ISAC
  participation is routine, an exported MISP event JSON would be a concrete, cheap addition.
- **No screenshots.** Understandable given synthetic data, but the `screenshots/` directory
  documenting what *would* be captured is an unusual choice that needs its explanation to be
  prominent, or a reviewer will read it as an empty folder.
- **No PDF renders.** The original structure anticipated `report.pdf`. Markdown is fine for
  GitHub, but a rendered PDF of the flagship technical report and executive brief would show
  the candidate can produce a document that survives leaving the repository.
- **No peer review.** The repository invites disagreement about confidence levels but shows no
  evidence any has been received or incorporated. One issue thread with a substantive
  disagreement and a revised judgement would be worth more than another case study.
- **No treatment of legal/regulatory triggers.** For an EU financial-services scenario I would
  expect at least a paragraph on when this becomes a GDPR Art. 33 notification question or a
  DORA incident-reporting matter. The candidate mentions GDPR in evidence handling but does not
  connect the incident to reporting obligations — which is a real part of the job in this
  sector.

---

## What an interviewer is likely to ask

Based on the claims this repository makes, I would probe:

1. *"Show me a judgement in here you're least confident about, and why you published it
   anyway."* — Testing whether the confidence labels are real or decorative. KJ-6 is the honest
   answer.
2. *"Your flagship says the gateway allow-list was the root cause. Defend that against someone
   who says the root cause was the user clicking."* — Testing causal reasoning and whether the
   candidate will hold a position.
3. *"Walk me through the AiTM identification without using the word 'proxy'."* — Testing whether
   the understanding is mechanical or memorised.
4. *"You rejected the shared-IP link. What would have made you accept it?"* — Testing whether
   selectivity is a principle or a phrase.
5. *"How many of these detections have you actually run?"* — The gap identified above. The
   candidate needs a prepared, non-defensive answer.
6. *"This is all synthetic. Convince me it transfers."* — The core objection. The honest answer
   — the method transfers and the reasoning is inspectable — is available in the repository and
   the candidate should give it without flinching.
7. *"What would you do in your first week if the evidence in a real case looked nothing like
   this?"*

---

## What should be improved before publication

In priority order, and none of these is expensive:

1. **Add the degraded-evidence case.** Highest return of anything on this list. It closes the
   largest gap and demonstrates a quality — comfort with "insufficient evidence" — that is
   rarer than technical skill.
2. **Add one real-tool appendix** against a public, dead, already-reported phishing domain.
   Closes the tool-fluency gap without any ethical cost.
3. **Make the screenshots policy prominent**, or remove the directory. An empty folder reads as
   an unfinished repository regardless of the explanation inside it.
4. **Export one MISP event or STIX bundle.** Concrete, sector-relevant, half a day's work.
5. **Add a short legal/regulatory note** connecting the flagship scenario to EU incident
   notification considerations.
6. **Render the flagship reports to PDF.**
7. **Fill in the author section properly.** The placeholders are currently visible and undercut
   an otherwise polished repository.
8. **Consider trimming.** The methodology and the flagship are both long. A reviewer with
   fifteen minutes will read the README, the flagship README and the executive brief — those
   three should carry the impression on their own, and currently they do, which is good. But
   watch that further additions don't dilute the entry points.

---

## Overall impression

This is not a beginner project and it is not padded. The analytical content is the substance
rather than the framing, the ethical handling is thought through rather than boilerplate, and
the candidate demonstrably knows the difference between what they observed and what they
concluded — which is the specific thing I am hiring for at junior level, because the technical
knowledge can be taught on the job and that habit largely cannot.

The gaps are real and they are gaps of *exposure*, not of understanding: no messy data, no
live tooling, no operational scale. Those are precisely the gaps a first role closes.

I would interview this candidate, and I would spend the interview on the material above rather
than on whether they know what DMARC is.

**No portfolio guarantees employment, and this one does not either.** Hiring depends on the
role, the competition, timing, and factors entirely outside a candidate's control. What this
repository does is ensure the conversation starts at a more interesting place than most.
