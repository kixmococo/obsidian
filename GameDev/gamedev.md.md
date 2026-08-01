# Enterprise Game Development Playbook
### From Concept to Live, Published Product — A Team-Based Standard Operating Guide

**Scope assumptions:** mid-size team (10–30 people), cross-platform PC/console target, engine-agnostic (Unity/Unreal notes called out where relevant), 18–24 month cycle. Scale phases up/down for your actual team size.

---

## Table of Contents
1. Team Structure & Roles
2. Phase 0 — Concept & Greenlight
3. Phase 1 — Pre-Production
4. Phase 2 — Production
5. Phase 3 — Alpha
6. Phase 4 — Beta
7. Phase 5 — Certification, Localization & Polish
8. Phase 6 — Launch
9. Phase 7 — Live Ops / Post-Launch
10. Tooling & Pipeline Stack
11. Documentation Standards
12. Milestone & Approval Gates (Quick Reference)
13. Risk Management
14. Some Recommendations & Opinions

---

## 1. Team Structure & Roles

| Role                                                                     | Responsibility                                                | Reports To          |
| ------------------------------------------------------------------------ | ------------------------------------------------------------- | ------------------- |
| Executive Producer                                                       | Budget, timeline, publisher/exec relations                    | Studio leadership   |
| Producer(s)                                                              | Day-to-day scheduling, cross-team coordination, risk tracking | Exec Producer       |
| Creative Director                                                        | Vision ownership, final creative call                         | Exec Producer       |
| Game Director                                                            | Systems, pillars, scope decisions                             | Creative Director   |
| Design Lead + Designers                                                  | Systems, levels, economy, narrative, UX flow                  | Game Director       |
| Art Director + Artists (concept, 3D, 2D, VFX, animation)                 | Visual identity, asset production                             | Creative Director   |
| Engineering Lead + Engineers (gameplay, tools, systems, engine, network) | All code, tools, performance                                  | Game Director / CTO |
| Audio Lead (sound design, music, VO direction)                           | All sound                                                     | Creative Director   |
| Narrative Lead / Writers                                                 | Story, dialogue, lore                                         | Creative Director   |
| QA Lead + Testers                                                        | Bug finding, regression, certification prep                   | Producer            |
| Live Ops / Community Manager                                             | Post-launch content, player comms                             | Exec Producer       |
| Marketing/PR Lead                                                        | Store presence, trailers, press, influencers                  | Exec Producer       |
| UX/UI Designer                                                           | Menus, HUD, accessibility, onboarding                         | Design Lead         |
| DevOps/Build Engineer                                                    | CI/CD, build pipeline, source control hygiene                 | Engineering Lead    |

**side note:** even at 10 people, keep these as *hats*, not headcount — one person can wear Producer + DevOps early on. What matters is that every hat is explicitly owned by someone; unowned responsibilities are where projects die.

---

## 2. Phase 0 — Concept & Greenlight (2–6 weeks)

**Goal:** prove the idea is worth funding before anyone builds real assets.

**Deliverables:**
- **One-pager pitch**: genre, hook, target audience, platform, comparable titles, why now.
- **Core pillars** (3–5 words each) — the non-negotiable identity of the game. Every future scope decision gets checked against these.
- **Market/competitive analysis** — comps, pricing tier, TAM, differentiation.
- **Risk assessment** — technical, market, scope risks with mitigation notes.
- **Rough budget & timeline band** (order-of-magnitude, not final).
- **Vertical slice proposal** — what will be built in pre-production to prove the fun.

**Gate:** Greenlight review with leadership/publisher. Go / no-go / revise.

---

## 3. Phase 1 — Pre-Production (2–4 months)

**Goal:** de-risk the game. Prove the core loop is fun, prove the tech works, lock the plan.

**Deliverables:**
- **Game Design Document (GDD)** — living doc covering core loop, systems, progression, narrative outline, UX flow, monetization model if any.
- **Technical Design Document (TDD)** — engine choice, architecture, target platforms, performance budgets, tools needed, third-party middleware (physics, netcode, analytics, IAP SDKs).
- **Art bible** — style pillars, color palette, reference boards, target fidelity.
- **Vertical slice** — one small, fully polished piece of the game representing final quality bar. This is the single most important artifact of pre-production.
- **Production plan** — full schedule broken into milestones (Alpha/Beta/Gold), staffing ramp plan, budget breakdown.
- **Risk register** — living document, reviewed weekly.
- **Pipeline setup** — source control (Perforce/Git+LFS), project management tool (Jira/Shortcut/Hansoft), build automation, asset naming conventions.

**Gate:** Pre-production review — vertical slice must demonstrate "is this fun / does this look/feel like the target product." This is where most cancelled projects die, and that's correct — it's cheap to cancel here, expensive later.

---

## 4. Phase 2 — Production (10–16 months)

**Goal:** build the full game to feature-complete.

### 4.1 Engineering
- Core systems architecture (gameplay framework, save system, UI framework, networking if multiplayer)
- Tools development (level editor extensions, custom inspectors, automation scripts)
- Feature implementation against GDD, tracked in sprints
- Performance budgeting per platform (frame time, memory, load times) set early, tracked every milestone
- Automated testing: unit tests for core systems, smoke tests for builds
- Nightly/CI builds on all target platforms

### 4.2 Design
- Level/mission design following pillars
- Systems tuning (economy, difficulty curves, progression)
- Scripting/flowcharting for narrative beats
- Playtesting cadence (internal weekly, external at milestones)
- Accessibility design (colorblind modes, remappable controls, subtitle options, difficulty options) — **build in from the start**, not bolted on later

### 4.3 Art & Audio
- Asset production pipeline: concept → model/rig → texture → in-engine integration → lighting pass
- Animation (gameplay, cinematic, UI motion)
- VFX pass
- Music composition, SFX library, VO recording/direction, audio mixing pass
- Consistent art review cadence against the art bible

### 4.4 Narrative
- Full script lock
- Dialogue implementation and localization prep (see Phase 5)
- Cinematics/cutscene production

### 4.5 Production/Process
- Sprint cadence (1–2 week sprints typical), standups, sprint review/retro
- Milestone builds delivered to stakeholders every 4–8 weeks
- Scope management — cut list maintained actively against pillars, not reactively at deadline
- Cross-discipline sync (design↔engineering↔art) at least weekly

**Gate:** Feature Complete milestone — every planned feature exists in some playable form (may be rough), no new features added after this point except by exception process.

---

## 5. Phase 3 — Alpha

**Definition:** Feature complete. All systems in, may be buggy/unbalanced/unpolished.

- Full game playable start to finish
- QA ramps to full regression testing
- Bug database triage process begins (severity/priority matrix, daily bug council)
- Performance profiling begins in earnest on all target hardware
- First full internal/external playtests with structured feedback capture
- Content lock discussions begin — what's truly in vs. cut

**Gate:** Alpha review — is the full game here, even if rough?

---

## 6. Phase 4 — Beta

**Definition:** Content complete. No new content, only bug fixing, balancing, polish.

- **Content lock** — no new assets, levels, or systems
- Full regression QA pass across all platforms
- Balance passes based on playtest data/telemetry
- Performance optimization sprints (frame rate, memory, load times, thermal on console)
- Accessibility QA pass
- Localization integration and linguistic QA (LQA) if shipping multiple languages
- Platform certification prep begins (see Phase 5)
- Marketing ramps: trailers, store page, press builds

**Gate:** Beta exit review — is the game stable, balanced, and hitting performance targets?

---

## 7. Phase 5 — Certification, Localization & Polish

- **Platform certification/TRC/TCR compliance** (Sony/Microsoft/Nintendo/Steam requirements — save data handling, achievements/trophies, controller conventions, age rating submission via ESRB/PEGI/etc.)
- **Localization** — text and audio for target markets, LQA pass in each language
- **Final polish pass** — juice, VFX timing, audio mix, UI feel, transition smoothness, loading screen tips, first-time-user-experience (FTUE)/tutorial refinement
- **Day-one patch planning** — decide what ships in the disc/initial build vs. day-one patch
- **Store page finalization** — screenshots, trailer, description, pricing, regional availability
- **Submission to platform cert** (allow 2–6 weeks buffer per platform for review/resubmission cycles)

**Gate:** Release Candidate (RC) build approved — this is the "gold master."

---

## 8. Phase 6 — Launch

- **Day-one patch deployment** if applicable
- **Launch monitoring war room** — engineering, QA, community, and production on-call for first 48–72 hours
- **Live telemetry dashboards** watched for crash rates, server load (if online), funnel drop-off
- **Community management active** — social channels, Discord, review monitoring, PR response plan for issues
- **Press/influencer embargo lift coordination**
- **Hotfix pipeline ready** — fast-track cert process for critical patches (most platforms have expedited cert for crash-level fixes)

---

## 9. Phase 7 — Live Ops / Post-Launch

- Post-mortem within 2–4 weeks of launch (what worked, what didn't, process improvements)
- Ongoing patch cadence (balance, bug fixes)
- Content roadmap for DLC/seasons/updates if planned
- Community feedback loop feeding back into design backlog
- Analytics-driven iteration (retention curves, funnel analysis, monetization tuning if applicable)
- Long-tail marketing (sales events, platform features, anniversary content)

---

## 10. Tooling & Pipeline Stack (typical enterprise choices)

| Category | Common Tools |
|---|---|
| Engine | Unreal Engine, Unity, or proprietary |
| Source Control | Perforce (art-heavy projects) or Git + LFS |
| Project Management | Jira, Shortcut, Hansoft, or Azure DevOps |
| Build Automation/CI | Jenkins, TeamCity, or Unity Cloud Build / Unreal's build tools |
| Bug Tracking | Jira, or integrated engine bug trackers |
| Communication | Slack/Teams + Confluence/Notion for docs |
| Analytics | GameAnalytics, Unity Analytics, or custom telemetry pipeline |
| Localization | memoQ, Crowdin, or in-house LQA pipeline |
| QA/Test Management | TestRail, Zephyr |
| Voice/Audio | Wwise or FMOD for adaptive audio |

---

## 11. Documentation Standards

Every enterprise team should maintain, as living documents (not write-once):
- GDD (Game Design Document)
- TDD (Technical Design Document)
- Art Bible
- Production Schedule / Roadmap
- Risk Register
- Bug Triage Rubric (severity/priority definitions)
- Style Guide (naming conventions, folder structure, code standards)
- Onboarding doc for new hires

---

## 12. Milestone & Approval Gates (Quick Reference)

```
Concept → Greenlight → Pre-Production → Vertical Slice Approval
→ Production → Feature Complete (Alpha entry)
→ Alpha → Content Lock (Beta entry)
→ Beta → Release Candidate
→ Certification/Localization → Gold Master
→ Launch → Live Ops
```

Each arrow is a gate requiring sign-off from Producer + relevant discipline leads + Exec Producer/publisher.

---

## 13. Risk Management

Maintain a living risk register scored on **likelihood × impact**, reviewed weekly in production meetings. Standard categories:
- **Scope risk** — feature creep vs. pillars
- **Technical risk** — unproven tech, engine limitations, platform constraints
- **Schedule risk** — dependency bottlenecks, understaffed disciplines
- **Market risk** — competitive shifts, platform policy changes
- **Personnel risk** — key-person dependency, burnout (crunch is a risk, not a plan)

---

## 14. Some Recommendations & Opinions

A few things I'd push you toward, beyond the standard template above:

1. **Protect the vertical slice like it's the whole budget.** Teams that skip a real pre-production vertical slice almost always discover "is this fun" too late — usually at Alpha, when it's expensive to fix. Spend the extra 4 weeks here.
2. **Build accessibility and localization hooks from day one of Production**, not as a Beta bolt-on. Retrofitting text systems for localization or input remapping for accessibility later is a real engineering tax — I'd bake it into the TDD from the start.
3. **Cap crunch explicitly in the production plan.** It's tempting to treat "crunch buffer" as free schedule slack, but it degrades quality and burns out your best people right before cert — the phase where you need them sharpest.
4. **Telemetry from Alpha onward, not just post-launch.** Instrument playtests early so Beta balance passes are data-driven, not vibes-driven.
5. **Treat the risk register as a weekly agenda item, not a document that gets written once and forgotten.** Most "surprise" schedule slips were visible in the risk register three months earlier — nobody was reading it.

If you want, I can turn any single phase above into its own deep-dive doc (e.g., a full GDD template, a sprint/scrum cadence guide, or a certification checklist per platform), or build you a project tracker artifact to actually run this. Just point me at what you want to build first.
