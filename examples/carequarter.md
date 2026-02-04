# Worked Example: CareQuarter (3-Phase Startup Validation)

A complete end-to-end example showing how Ditto was used to validate a
startup concept across three iterative research phases in ~4 hours.

**Public study links (viewable by anyone):**
- [Phase 1: Pain Discovery](https://app.askditto.io/organization/studies/shared/UlXcv4cjValQu0qJFJrSfL5eKB1zHfc5e2cFZiLYjrA)
- [Phase 2: Deep Dive](https://app.askditto.io/organization/studies/shared/x52Mu1QOwow6fbhornqjY51ug4QI-jY7daoj37ndfAw)
- [Phase 3: Concept Test](https://app.askditto.io/organization/studies/shared/IQiBzKN_q2M3-vSISd_1C7zr2XB66tV6QvVlfy8DkMo)

**Full article:** https://askditto.io/news/ai-founders-use-synthetic-research-to-launch-startup-in-4-hours

---

## Context

CareQuarter is an elder care coordination service for the "sandwich
generation" — working adults aged 45-65 managing healthcare for aging
parents. The concept was validated entirely through Ditto synthetic
research before any product was built.

## Phase 1: Pain Discovery

**Goal:** Identify the core customer problem and unmet needs.

**Research Group:**
```json
{
  "name": "US Adults Managing Elder Care",
  "group_size": 12,
  "filters": {
    "country": "USA",
    "age_min": 45,
    "age_max": 65
  }
}
```

**Questions asked (7):**
1. Walk me through what healthcare administration looks like for your aging parent. What tasks fall on you?
2. What's the most frustrating part of managing their healthcare? What makes you lose sleep?
3. How much time per week do you spend on healthcare admin for your parent?
4. What tools, people, or workarounds do you currently rely on?
5. Have you tried any services or apps to help? What happened?
6. If you could fix ONE thing about managing your parent's healthcare, what would it be?
7. What would make you hesitant to let someone else handle some of this, even if it freed up your time?

**Key Finding:**

The dominant theme was NOT "I need an app" or "I want better technology."
It was:

> "I'm responsible without real authority in a system that's chopped
> into pieces."

Every persona described being the unpaid case manager — the "human API"
stitching together a healthcare system that refuses to talk to itself.

**Specific pain points surfaced:**
- Portal fragmentation (different login for every provider, pharmacy, lab)
- Prior authorisation ping-pong (insurer blames provider, provider blames insurer)
- HIPAA purgatory (legal auth signed but not visible in provider systems)
- Friday 4pm fires (hospital discharge with nothing arranged)
- 2am worry spiral (persistent anxiety about what's being missed)

**Decision:** The opportunity is "care coordination with authority" — a
human coordinator with legal standing to act, not a software platform.

---

## Phase 2: Deep Dive

**Goal:** Understand what "authority" means to customers. Define trust
requirements, triggers, and deal breakers.

**Research Group:** 10 new personas, same demographic filters.

**Questions asked (7):**
1. What does "authority to act" mean to you in the context of your parent's healthcare?
2. What would a third party need to demonstrate before you'd grant them HIPAA access?
3. What specific tasks would you trust a coordinator to handle? What's off-limits?
4. How do you want to be kept informed? What level of detail? How often?
5. What trigger moment would make you pick up the phone and hire someone?
6. What's your biggest fear about handing over any control?
7. What would make you fire the coordinator immediately? What's the line?

**Key Findings:**

Trust architecture emerged as the critical design constraint:

- **Start with HIPAA only** — power of attorney is too much trust too fast.
  Customers want to see competence before granting broader authority.
- **Named person, not a team** — every persona rejected rotating care
  teams. They'd been burned by anonymous call centres.
- **Phone and paper first** — the target demographic actively rejected
  app-based solutions. They have too many portals already.
- **Guardrails are non-negotiable** — spending caps, defined scope, easy
  exit, clear documentation of what the coordinator can and cannot do.

**Trigger moments identified:**
- Hospital discharge (especially Friday afternoon)
- New diagnosis requiring specialist coordination
- Medication change across multiple providers
- Parent transitioning between living arrangements
- Caregiver experiencing burnout or their own health crisis

**Deal breakers:**
- Rotating or anonymous staff
- No spending caps
- Data resale
- Complicated cancellation

**Decision:** The product is a named human coordinator, HIPAA-authorised,
operating by phone, with tiered authority and customer-controlled guardrails.

---

## Phase 3: Concept Test

**Goal:** Validate positioning, pricing, and purchase intent.

**Research Group:** 10 new personas, same demographic filters.

**Questions asked (7):**
1. I'm going to show you four descriptions. Which grabs you most?
   A) "Stop being the unpaid case manager."
   B) "A named coordinator who actually gets things done."
   C) "The family member you wish you had nearby."
   D) "Healthcare is broken. We're the fixer."
2. What's your reaction to a monthly service at $175 for routine coordination?
3. What about $325/month for complex needs (discharges, billing disputes, extended hours)?
4. Would you use a $125 per-event crisis add-on for same-day emergencies?
5. What moment would trigger you to sign up? What pushes you from "interested" to "take my money"?
6. What would make you recommend this to a friend or sibling in the same situation?
7. What would kill the deal for you, even if everything else sounds perfect?

**Key Results:**

**Positioning winner:** "Stop being the unpaid case manager."
- Hit hardest because it validates the customer's experience without
  patronising them. Names the role they've been forced into.
- Runner-up: "A named coordinator who actually gets things done."
- Loser: "The family member you wish you had nearby" — described as
  "manipulative" and "too personal."

**Pricing validated:**

| Tier | Price | Persona Response |
|------|-------|------------------|
| Core | $175/month | 100% within acceptable range |
| Full | $325/month | 100% within acceptable range |
| Crisis | $125/event | Unanimous interest |

Not "probably reasonable." Every single persona confirmed these prices
were acceptable for the service described.

**Purchase trigger:** Hospital discharge — specifically the Friday 4pm
call. Crisis converts.

---

## What Was Produced

Using findings from all three phases:

1. **Landing page** — complete, responsive, conversion-optimised.
   Copy derived directly from persona responses.
2. **Pitch deck** — 14 slides including market sizing ($470B TAM),
   validated pricing, and competitive positioning.
3. **Messaging guide** — brand voice, words to use/avoid, phrases
   that tested well vs. poorly.
4. **Design specification** — full production-ready design spec.

**Total time:** ~4 hours
**Total personas:** 32 (across 3 phases)
**Total questions:** 21
**Human intervention:** Zero (beyond the initial "go")

---

## Patterns to Reuse

1. **Phase 1 surfaces the real problem** — which is often different from
   what you expected. The hypothesis was "time burden." The finding was
   "structural authority gap."

2. **Phase 2 converts insight into design constraints** — don't jump
   from pain to solution. Understand trust boundaries, triggers, and
   deal breakers first.

3. **Phase 3 validates the commercial model** — positioning, pricing,
   purchase triggers. The qualitative depth (not just "would you pay X?"
   but "why?" and "what would kill it?") makes the data defensible.

4. **Each phase's questions could not have been designed without the
   previous phase's findings.** This iterative approach produces
   qualitatively different results than a single-pass study.

5. **Negative findings are as valuable as positive ones.** The rejection
   of app-based solutions, emotional positioning, and rotating care teams
   each saved months of building the wrong thing.
