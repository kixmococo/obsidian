# Discord-Ready Version — Enterprise Game Dev Playbook
Each section below is separated by `=== MESSAGE BREAK ===`. Copy everything between breaks into one Discord message (all are under 2,000 characters). Tables converted to bullets since Discord doesn't render Markdown tables. Best posted as a thread so it reads top to bottom.

=== MESSAGE BREAK ===

# 🎮 Enterprise Game Dev Playbook — Concept to Live Game

**Scope:** mid-size team (10–30 people), cross-platform PC/console, engine-agnostic, 18–24 month cycle.

**Phases covered in this thread:**
0️⃣ Concept & Greenlight
1️⃣ Pre-Production
2️⃣ Production
3️⃣ Alpha
4️⃣ Beta
5️⃣ Cert, Localization & Polish
6️⃣ Launch
7️⃣ Live Ops
Plus: team roles, tooling stack, and gate checklist.

Reply in thread with questions on any phase 👇

=== MESSAGE BREAK ===

## 👥 Team Structure & Roles

- **Executive Producer** — budget, timeline, publisher relations
- **Producer(s)** — scheduling, cross-team coordination, risk tracking
- **Creative Director** — vision ownership, final creative call
- **Game Director** — systems, pillars, scope decisions
- **Design Lead + Designers** — systems, levels, economy, narrative, UX
- **Art Director + Artists** — concept, 3D, 2D, VFX, animation
- **Engineering Lead + Engineers** — gameplay, tools, systems, engine, netcode
- **Audio Lead** — sound design, music, VO direction
- **Narrative Lead / Writers** — story, dialogue, lore
- **QA Lead + Testers** — bug finding, regression, cert prep
- **Live Ops / Community Manager** — post-launch content, player comms
- **Marketing/PR Lead** — store presence, trailers, press
- **UX/UI Designer** — menus, HUD, accessibility, onboarding
- **DevOps/Build Engineer** — CI/CD, build pipeline

💡 Even at small team size, keep these as *hats* not headcount — one person can wear multiple. What matters is every hat is explicitly owned.

=== MESSAGE BREAK ===

## 0️⃣ Phase 0 — Concept & Greenlight (2–6 weeks)

**Goal:** prove the idea is worth funding before building real assets.

**Deliverables:**
- One-pager pitch (genre, hook, audience, platform, comps, why now)
- Core pillars (3–5 words each) — non-negotiable identity, every future scope call gets checked against these
- Market/competitive analysis
- Risk assessment (technical, market, scope)
- Rough budget & timeline band
- Vertical slice proposal

**Gate:** Greenlight review with leadership/publisher — go / no-go / revise.

=== MESSAGE BREAK ===

## 1️⃣ Phase 1 — Pre-Production (2–4 months)

**Goal:** de-risk the game. Prove the core loop is fun and the tech works.

**Deliverables:**
- Game Design Document (GDD) — core loop, systems, progression, narrative outline, UX flow, monetization
- Technical Design Document (TDD) — engine, architecture, platforms, perf budgets, middleware
- Art bible — style pillars, palette, reference boards, fidelity target
- **Vertical slice** — small, fully polished piece at final quality bar (the most important artifact of this phase)
- Production plan — full schedule, staffing ramp, budget breakdown
- Risk register (living doc, reviewed weekly)
- Pipeline setup — source control, PM tool, build automation, naming conventions

**Gate:** Vertical slice must prove "is this fun / does it hit the target bar." Cheap to cancel here, expensive later.

=== MESSAGE BREAK ===

## 2️⃣ Phase 2 — Production (10–16 months)

**Goal:** build the full game to feature-complete.

**Engineering:** core architecture, tools dev, feature implementation, performance budgeting per platform, automated testing, nightly CI builds

**Design:** level/mission design, systems tuning, narrative scripting, weekly internal playtests, accessibility built in from the start (not bolted on later)

**Art & Audio:** full asset pipeline, animation, VFX, music/SFX/VO, consistent art review vs. the art bible

**Narrative:** script lock, dialogue implementation, localization prep, cinematics

**Process:** 1–2 week sprints, milestone builds every 4–8 weeks, active cut-list management against pillars, weekly cross-discipline sync

**Gate:** Feature Complete — every planned feature exists in some playable form. No new features after this except by exception process.

=== MESSAGE BREAK ===

## 3️⃣ Phase 3 — Alpha

**Definition:** Feature complete. All systems in, may be buggy/unbalanced/unpolished.

- Full game playable start to finish
- QA ramps to full regression testing
- Bug triage process begins (severity/priority matrix, daily bug council)
- Performance profiling starts on all target hardware
- First full internal/external playtests w/ structured feedback
- Content lock discussions begin

**Gate:** Is the full game here, even if rough?

=== MESSAGE BREAK ===

## 4️⃣ Phase 4 — Beta

**Definition:** Content complete. No new content — only bug fixing, balancing, polish.

- **Content lock** — no new assets, levels, systems
- Full regression QA across all platforms
- Balance passes from playtest data/telemetry
- Performance optimization sprints (frame rate, memory, load times, thermals)
- Accessibility QA pass
- Localization integration + linguistic QA
- Platform cert prep begins
- Marketing ramps: trailers, store page, press builds

**Gate:** Is the game stable, balanced, and hitting performance targets?

=== MESSAGE BREAK ===

## 5️⃣ Phase 5 — Certification, Localization & Polish

- Platform cert/TRC/TCR compliance (save data, achievements/trophies, controller conventions, age ratings)
- Localization + LQA pass per language
- Final polish pass — juice, VFX timing, audio mix, UI feel, FTUE/tutorial refinement
- Day-one patch planning — what ships on disc vs. day-one patch
- Store page finalization
- Submission to platform cert (buffer 2–6 weeks per platform for review/resubmit cycles)

**Gate:** Release Candidate (RC) build approved — "gold master."

=== MESSAGE BREAK ===

## 6️⃣ Phase 6 — Launch

- Day-one patch deployment if applicable
- Launch war room — eng/QA/community/production on-call 48–72 hrs
- Live telemetry dashboards (crash rates, server load, funnel drop-off)
- Active community management — socials, Discord, review monitoring
- Press/influencer embargo coordination
- Hotfix pipeline ready — fast-track cert for critical patches

## 7️⃣ Phase 7 — Live Ops

- Post-mortem within 2–4 weeks of launch
- Ongoing patch cadence
- Content roadmap (DLC/seasons) if planned
- Community feedback → design backlog loop
- Analytics-driven iteration (retention, funnel, monetization tuning)
- Long-tail marketing (sales, platform features, anniversaries)

=== MESSAGE BREAK ===

## 🛠️ Tooling & Pipeline Stack

- **Engine:** Unreal, Unity, or proprietary
- **Source Control:** Perforce (art-heavy) or Git + LFS
- **Project Management:** Jira, Shortcut, Hansoft, Azure DevOps
- **CI/Build:** Jenkins, TeamCity, Unity Cloud Build / Unreal build tools
- **Bug Tracking:** Jira or engine-integrated trackers
- **Comms/Docs:** Slack/Teams + Confluence/Notion
- **Analytics:** GameAnalytics, Unity Analytics, custom telemetry
- **Localization:** memoQ, Crowdin, in-house LQA
- **QA/Test Management:** TestRail, Zephyr
- **Audio:** Wwise or FMOD

=== MESSAGE BREAK ===

## ✅ Milestone Gate Checklist (pin this one)

```
Concept → Greenlight
→ Pre-Production → Vertical Slice Approval
→ Production → Feature Complete (Alpha entry)
→ Alpha → Content Lock (Beta entry)
→ Beta → Release Candidate
→ Cert/Localization → Gold Master
→ Launch → Live Ops
```
Each arrow = sign-off required from Producer + relevant discipline leads + Exec Producer/publisher.

=== MESSAGE BREAK ===

## 💬 u2's Take

1. **Protect the vertical slice like it's the whole budget.** Skipping it means discovering "is this fun" too late, usually at Alpha when it's expensive to fix.
2. **Bake accessibility + localization into Production from day one**, not Beta. Retrofitting is a real engineering tax.
3. **Cap crunch explicitly in the schedule.** It's not free slack — it burns out your best people right before cert.
4. **Instrument telemetry from Alpha on**, so Beta balance passes are data-driven, not vibes-driven.
5. **Treat the risk register as a weekly agenda item.** Most "surprise" slips were visible in it three months earlier — nobody was reading it.
