# Whiteboard AI — Team Document

> **Purpose:** This document is the shared operating system for the Whiteboard AI team.
> It defines strategy, roles, state, skills, and provides a place for anyone to post
> suggestions, questions, and tool requests.
> **Owners:** Conrad (human) + Product Dev Agent (Claude Code). Updated each sprint.

---

## 1. Vision

Whiteboard AI makes it effortless to turn any LLM-generated explanation into a
Khan-Academy-style animated whiteboard presentation — no video generation required.
The framework combines cheap, fast text-based generation with human preference for
animated visual learning.

**North star:** A user can ask a question and receive a polished, synchronized
audio+visual explanation in under 10 seconds, at near-zero marginal cost.

---

## 2. Strategy (The Flywheel)

```
Strong Product Vision
        ↓
  System that delivers low-latency,
  engaging LLM→human visual interface
        ↓
  Users attracted → Feedback collected
        ↓
  Stronger Product Vision
```

**Current flywheel phase:** PoC → first real user (Conrad) → refine core experience.

---

## 3. Team

| Role | Who | Responsibilities |
|------|-----|-----------------|
| Product owner / user | Conrad | Strategy, user testing, final approvals, suggestion board |
| Product Dev Agent | Claude Code | Codebase ownership, product spec maintenance, implementation, sprint execution |
| Research Agent | Claude (ad hoc) | Deep research on request (skills, patterns, competitive landscape) |

**Decision authority:**
- Technical implementation details → Dev Agent proposes, Conrad approves
- Product direction → Conrad decides with Dev Agent input
- Urgent bugfixes → Dev Agent can act, notes in suggestion board

---

## 4. Systems of Record

| System | Owner | Format | Location |
|--------|-------|--------|----------|
| Codebase | Dev Agent + Conrad | TypeScript / JSON | `src/`, `public/` |
| Product Spec | Dev Agent (Conrad reviews) | Markdown | `docs/PRODUCT_SPEC.md` |
| Team Document | Co-owned | Markdown | `docs/TEAM.md` (this file) |
| User Feedback | System collects, Conrad reviews | JSON export | Test Mode → Export |

---

## 5. Skills Registry

Skills are Claude Code custom slash commands stored in `.claude/commands/`.
To use a skill, type `/skill-name` in a Claude Code session.

### Product Management Skills

| Skill | Command | Description | Status |
|-------|---------|-------------|--------|
| Sprint review | `/sprint-review` | Audit current tasks, update spec, propose next sprint | Planned |
| Feedback analysis | `/analyze-feedback` | Read exported feedback JSON, extract patterns, update spec | Planned |
| Spec update | `/update-spec` | Guided product spec update after user input | Planned |

### Development Skills

| Skill | Command | Description | Status |
|-------|---------|-------------|--------|
| Scene validator | `/validate-scene` | Validate a scene JSON against schema/types.ts rules | Planned |
| Simplify | `/simplify` | Review changed code for quality and reuse | Active |
| Test runner | `/run-tests` | Execute test suite and report results | Planned |
| Deploy check | `/deploy-check` | Verify GitHub Pages build is clean before push | Planned |

> **To add a skill:** Post a request in section 8 (Suggestion Board) with the skill name,
> what it does, and why it would help. Dev Agent will implement and update this table.

---

## 6. Current State

- **Live URL:** https://[username].github.io/Whiteboard-AI (GitHub Pages)
- **Users:** 1 (Conrad, solo)
- **Stage:** PoC — not ready for external users
- **Feedback system:** Test Mode panel with JSON export (basic, functional)
- **TTS:** Web Speech API — reliable on Chrome/Edge desktop, limited on Firefox
- **Known limitations:** See Product Spec §6 (Known Issues)

---

## 7. Active Sprint

> Updated each sprint. Dev Agent updates status; Conrad approves done items.

| # | Task | Owner | Status | Notes |
|---|------|-------|--------|-------|
| 1 | Define Team document format | Dev Agent + Conrad | ✅ Done | This file |
| 2 | Define Product Spec format | Dev Agent + Conrad | ✅ Done | `docs/PRODUCT_SPEC.md` |
| 3 | Create PM skills Deep Research prompt | Dev Agent | ✅ Done | `docs/research/pm-skills-prompt.md` |
| 4 | Create Dev skills Deep Research prompt | Dev Agent | ✅ Done | `docs/research/dev-skills-prompt.md` |
| 5 | First release under new working approach | Dev Agent + Conrad | 🔄 In Progress | Completing setup |

**Next sprint candidates** (not yet committed):
- Improve LLM-to-scene generation UX (in-app prompt → scene, no copy/paste)
- Improve scene validation error messages
- Add voice selector to UI
- Mobile layout improvements

---

## 8. Suggestion Board

> Post suggestions, questions, tool requests, or blockers here.
> Include your name/role, date, and tag: [SUGGESTION] [QUESTION] [TOOL REQUEST] [BLOCKER]

---

**[TOOL REQUEST] Dev Agent — 2026-03-09**
Request: `/validate-scene` skill — validates a pasted scene JSON against the full
type schema and reports specific errors with line numbers. Would catch trigger_phrase
mismatches before runtime. High value, low cost.

---

**[TOOL REQUEST] Dev Agent — 2026-03-09**
Request: `/analyze-feedback` skill — reads the exported feedback JSON from Test Mode,
summarizes patterns (common failure types, prompts that worked/failed), and drafts
product spec update suggestions. Critical for closing the feedback flywheel loop.

---

**[QUESTION] Dev Agent — 2026-03-09**
What is the target response latency for scene generation? (Time from user query to
first TTS word playing.) This defines whether we need streaming JSON parsing or
can wait for full response. Need Conrad's answer to inform architecture decisions.
