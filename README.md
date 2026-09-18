# SidelineReel Product Assessment — Wanja Naomi Mugambi

**Video:** [PASTE VIDEO LINK HERE]
**Prototype:** [PASTE PUBLISHED PROTOTYPE LINK HERE]
**Other work / portfolio:** [optional — leave blank if none]

This assessment references Ajaia ([https://ajaia.ai](https://ajaia.ai)) and its product SidelineReel.

---

## Task 1: The Roadmap Call

**Reading each source for what it actually is:**

| Material | What it is | Evidence weight |
|---|---|---|
| Carla's email + Jamal's Slack | One client's leadership + our own sales team, pushing the same ask | Organizational/commercial pressure — real, but no product evidence behind the specific feature |
| Survey (58% want editing controls) | Stated preference, 210 respondents | Weak until checked against behavior |
| Usage data (Reel Editor: 6% open, <1% finish) | Actual behavior | Strong — and it contradicts the survey |
| Support tickets (3 separate wrong-kid mix-ups, 3 different clubs, same pattern: confusable digits) | Unprompted, independent, recurring failures | Strong |
| Health dashboard (92% share rate, footnoted) | Metric leadership is citing as proof of health | Looks strong, is hollow — the footnote excludes the 30% of reels never opened, some due to wrong-kid thumbnails |

**Resolving the survey-vs-usage conflict.** "More editing controls" tops the survey, but almost nobody uses the editor that already exists, and of those who try, almost none finish. That's not suppressed demand for a better editor — a stated wish that doesn't convert to behavior usually means the ask is a proxy for something else. Read against the support tickets, "I want to edit my kid's reel" most plausibly means "I want to be able to fix it when it's wrong," not "I want creative control."

**What the flattering metric is hiding.** 92% share rate sounds like a healthy product. The footnote is the real story: it's calculated only over opened reels, excluding the 30% that are never opened at all — a share of which go unopened because the thumbnail shows the wrong kid. The team is measuring enthusiasm only among people the bug hasn't hit yet.

**Three independent signals, one root cause.** The Ridgeline, Brightwater, and Cobblestone tickets are three unconnected clubs reporting the same failure shape — visually similar jersey digits (14/4, 1/11) get cross-matched. That's a pattern in the matching logic, not noise, and it's quietly costing the product on the survey axis and the metric axis at the same time.

### Ranking

1. **Fix roster tagging trust** — hold low-confidence jersey matches for a quick human confirmation before they ship to the wrong kid's reel. Evidence: three independent tickets sharing a failure pattern, the footnoted 30% non-open rate, and the likely real meaning behind the survey's top ask. This is the one thing actively eroding the core promise — "the right highlight lands in the right kid's reel" — for every club, not just one.
2. *(Backlog, next cycle)* A lightweight "flag this clip" correction control for parents/coaches — a cheap human-in-the-loop safety net, worth revisiting once the confidence-gate work ships and we can see what still slips through.
3. *(Backlog, low priority)* Watermark-missing bug, app icon complaint — real but minor, no urgency signal behind either.

### What I'm NOT building, and why

**Livestreaming, including for Ridgeline.** Every signal pointing at it is commercial, not product: one client's board ask, escalated through sales, tied to a renewal deadline. Nothing in the survey, usage data, tickets, or health metrics independently supports it, and it would be a major build — real-time video infrastructure — outside SidelineReel's actual job, which is finding and personalizing highlights, not live broadcast. More pointedly: the Ridgeline parent who filed the wrong-kid ticket is Carla's own customer. The thing actively damaging trust at her club right now isn't the absence of livestreaming — it's a highlight reel showing the wrong kid. I'd tell Carla exactly that, concretely: not "we hear you," but "here's the specific thing we're fixing that's hurting your families this season, and here's why livestreaming isn't it — yet. If it becomes a broader pattern across clubs, we'll revisit it with real scoping, not a renewal-deadline commitment."

**Reel Editor expansion.** Survey-topped, but usage says otherwise — 6% open it, <1% finish. Building more editing power onto a feature people already abandon solves the wrong layer of the problem.

---

## Task 2: The Spec

**Feature: Confidence Gate for Roster Tagging**

**Outcome it's meant to produce.** No highlight clip reaches a family's reel under a kid's name unless the system, or a human, is actually confident it's that kid. Wrong-kid clips drop toward zero, without slowing down the clips already tagged correctly.

**In scope:**
- Every detected highlight is tagged with a jersey number and a match-confidence score at the point roster tagging happens.
- Clips below a confidence threshold are held out of the auto-generated reel and routed to a coach review queue instead of being auto-published.
- The coach sees the clip, the system's best guess, and the roster list, and confirms the correct kid or reassigns in one tap.
- Once confirmed, the clip flows into that kid's reel and the weekly recap as normal.
- Any confirmed correction is logged (predicted jersey number vs. corrected number) to improve the matching model over time.

**Explicitly out of scope (this cycle):**
- No changes to the underlying computer-vision/jersey-recognition model itself — we don't yet know if the failure is the model, camera angle, or genuinely ambiguous digits, and rebuilding it blind is a multi-month bet, not a three-week one.
- No parent-facing "report a mistake" flow yet — that's the fallback safety net for whatever still slips through, sequenced after this.
- No change to the Reel Editor.
- No livestreaming.

**Where a human stays in the loop.** The coach — already the person uploading footage and closest to the roster — reviews only the clips the system itself flags as uncertain. High-confidence clips still ship automatically; this doesn't add friction to the majority of tagging that already works.

**Acceptance criteria:**
1. A clip below the confidence threshold never appears in an auto-published reel without coach confirmation.
2. The coach review queue shows: video clip, system's guessed jersey number, full roster list, and a one-tap confirm/reassign control.
3. A confirmed or reassigned clip appears correctly in the right kid's reel and that week's recap.
4. If a coach doesn't act on a flagged clip before the weekly recap sends, that clip is excluded from the recap rather than guessed — never shipped wrong.
5. Every prediction and correction is logged with a timestamp for measuring match accuracy over time.

**Top two failure modes post-ship, and what catches each:**
1. **Threshold is miscalibrated** — too loose and wrong-kid clips still slip through; too strict and too many clips get flagged, making coach review a chore coaches start ignoring. *Catch:* track weekly wrong-kid ticket volume (should trend down) alongside queue-completion rate by coaches (should stay high); tune the threshold against both signals together, not just one.
2. **Coaches ignore the review queue** — busy schedules, queue goes unchecked before the recap sends. *Catch:* track queue completion rate per coach. A low rate doesn't compromise safety, since criterion 4 excludes rather than guesses — but it becomes the signal to redesign the nudge (e.g., a reminder tied to recap send time) rather than to loosen the safety net.

---

## Task 3: Prototype

**Link:** [PASTE PUBLISHED PROTOTYPE / GITHUB PAGES LINK HERE]

The prototype demonstrates the core coach-facing flow for the confidence gate: a flagged clip with the system's ambiguous guess (jersey #14 vs. #4 — the exact pattern seen in the support tickets), the coach selecting the correct player from the roster, and a confirmation state showing the clip routed to the right kid's reel. A "skip" path shows the clip being excluded from the recap rather than guessed, and an empty-queue state shows that high-confidence clips ship automatically without ever entering the review flow.

What's real: the interaction pattern an engineer would build against — flag, review, confirm or skip, route correctly. What's mocked: the actual confidence-scoring model, the roster database, and the recap-send pipeline.

**What's inside**

```
sidelinereel-project/
├── index.html   ← the entire app (markup, styles, and logic in one file)
└── README.md    ← this file
```

There is no build step and no package manager involved. The app is plain HTML/CSS/JavaScript with no external dependencies. Everything works fully offline.

**How to run it locally**

You don't need Node, npm, or any dev server. Pick whichever is easiest:

- **Open the live link** — [PASTE PUBLISHED PROTOTYPE / GITHUB PAGES LINK HERE]
- **Open the file directly** — download `index.html`, then double-click it (or right-click → Open with → your browser). It loads immediately, no server required.

---

## Task 4: AI Workflow Note

I used AI throughout this assessment: to sort and cross-reference the five materials for the pattern connecting the survey/usage conflict, the support tickets, and the footnoted health metric; to draft the spec structure; and to build the prototype's interface and interaction logic. I kept the actual roadmap call and the reasoning behind it human — deciding that livestreaming gets a clear no rather than a scoped concession, and that the survey's "editing controls" ask was more plausibly a proxy for correction than a request for creative control. One thing I checked rather than accepted: the initial pass treated the three support tickets as separate, unrelated bugs. I pushed to check whether they shared a pattern — visually similar jersey digits — since that's what turns three anecdotes into one piece of real evidence.
