# Executive Brief — EU-FIN-001

**Phishing attack against Treasury and Payments staff**
For: CISO · Security Manager · SOC Manager · IT leadership
2026-02-18 · TLP:CLEAR · One page

*Synthetic scenario for a portfolio project — not a real incident.*

---

### What happened

On the morning of 11 February, eleven employees received an email that looked like it came
from our own IT Security team. Nine of them work in Treasury or Payments. The email claimed
an unknown device had signed in to their account and asked them to log in to check.

Four people clicked. Two entered their password and approved the multi-factor prompt on their
phone. Their sessions were shut down within half an hour of the first click.

### Why it matters

The fake login page was not a simple copy of our sign-in screen. It sat **between the
employee and our real login system**, passing everything through in real time. That design
means the attacker could capture not just the password but the **session token** — the thing
our systems accept as proof that someone has already logged in.

Two consequences follow, and both are uncomfortable:

- **Approving the phone prompt did not protect the user.** Our current multi-factor method
  can be passed straight through to the attacker.
- **Changing the password does not end the attacker's access.** Only cancelling the session
  does. We did that, but the window was open for roughly half an hour.

The people targeted were chosen. Nine of eleven can authorise payments. The email used their
correct names and job titles.

### What we found

- The email was sent from infrastructure the attacker set up and had been maintaining for
  five months, which is why it looked reputable to our filters.
- Our own email policy **should have quarantined it**. A setting on our mail gateway
  overrode that policy and let it through. This is the reason it reached anyone.
- The fake website was registered three days before the attack. We can see the preparation in
  public records — which means we could have seen it coming.
- The same attacker very likely targeted us once before, in January, with a simpler version.
  They have improved.
- Three further fake websites have been prepared and not yet used.

### What we know and what we don't

**We know** how the email got in, what the fake site was capable of, and that two accounts
were exposed.

**We don't yet know** whether the attacker actually used the stolen sessions before we closed
them. We are reviewing sign-in records now. We also don't know where the attacker obtained
our staff names and titles — most likely public sources, but a previous undetected intrusion
has not been ruled out.

We are **not** claiming to know who the attacker is. The methods used are widely available
and are not distinctive enough to identify anyone.

### What we should do next

| Priority | Action | Why |
|---|---|---|
| **Now** | Confirm whether the stolen sessions were used | Decides whether this is an attempted or actual breach |
| **Now** | Fix the mail gateway setting that ignored our own policy | This one setting is why the email was delivered |
| **This quarter** | Move payment-authorising staff to security keys (FIDO2/WebAuthn) | **The only measure that would have stopped this attack.** Phone-approval MFA does not |
| **This quarter** | Tighten our email policy so spoofed messages are rejected outright | Removes this whole category of impersonation |
| **Ongoing** | Monitor public certificate records for fake domains using our name | Both attacks were visible 2–3 days in advance |
| **Ongoing** | Brief Treasury and Payments on this specific trick | The email worked by making *checking* feel like the safe thing to do |

### The one-sentence version

Our mail filter let through a well-made impersonation because of a configuration override,
and our current multi-factor method could not have stopped what came next — so the two fixes
that matter are the gateway setting and security keys for staff who can move money.

---

*Technical detail: [report-technical.md](report-technical.md). Full analytical record:
[investigation.md](investigation.md).*
