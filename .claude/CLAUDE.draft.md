# Team Document — WhiteBoard AI

This file is the authoritative guide for everyone on the team — human and agent alike. Read it before doing any work. It defines the project, the strategy, how the team operates, and where everything lives.

---

## The Project

WhiteBoard AI is a framework that lets LLMs generate illustrative presentations for knowledge workers without expensive video generation. It consists of a JSON-oriented language and a rendering engine. An LLM defines a presentation using the language; the engine animates it using text-to-speech and SceneGraph.

**The core insight**: LLMs think faster than people can read. WhiteBoard AI bridges that gap — combining cheap, fast text generation with human preference for animated, visual communication.

**The flywheel**: Strong product vision → system that delivers low-latency, engaging visual experiences → attracts users → yields feedback → stronger product vision. Every decision should be traced back to whether it accelerates or slows this loop.

**Current state**: PoC on GitHub Pages. Solo operation — one human (a business school student) and AI agents. Not yet ready to scale to more users. A basic user feedback framework exists. We should build as if for business school users for the time being.

---

## Systems of Record

There are four systems of record. Each has a clear owner. Project state, decisions, and history live here — not in skills.

| System | Location | Owner |
|---|---|---|
| Codebase | `Codebase/` | Dev skill |
| System Prompt | `Codebase/public/system-prompt.md` | PM skill + Dev skill |
| Product Spec | `Product Spec/` | PM skill + human |
| User Feedback | `User Feedback/` | PM skill |

**Agents should read the relevant system of record before starting any task.** Don't rely on memory or skill files for current project state — go to the source.

The **system prompt** is the LLM contract — it defines what language constructs exist and how an LLM should use them to generate scenes. It is a joint PM/Dev artifact: PM owns the design intent; Dev owns technical accuracy relative to the rendering engine. Changes to either the engine or the product vision may require updating the system prompt.

---

## Agent Session Kickoff

At the start of every session, read the following to get up to speed:

| Agent | Read on startup |
|---|---|
| PM | All files in `Product Spec/`, all files in `User Feedback/`, recent git log |
| Dev | `Codebase/README.md`, recent git log |
| Both | `CLAUDE.md` (auto-loaded) |

If `Product Spec/` is empty, the PM should flag it and create a spec before starting any feature work.

---

## How Skills Work

Skills encode **best practices for a type of task** — mental models, frameworks, and workflows that apply regardless of what the project looks like at any given moment.

Skills do NOT encode project state. The current state of the codebase, open tasks, or recent decisions belong in the systems of record above, not in a skill file.

| Skill | Domain |
|---|---|
| `pm` | Product management — specs, roadmap, prioritization, user feedback analysis |
| `dev` | Development — architecture decisions, code review, implementation, refactoring |
| `skill-creator` | Creating and improving skills |

---

## Branch Rules

Never checkout or switch to `main` or `master`. All agent work happens on the `agents` branch. If you need changes on main, create a PR — do not switch to it directly.

---

## Skill and CLAUDE.md Update Rules

Never edit skill files (`.claude/skills/*/SKILL.md`) or `CLAUDE.md` directly. All changes go through the draft → PR → GitHub Actions pipeline:

1. Write the updated version to the draft location (`.claude/skills-draft/<skill>/SKILL.md` for skills, `.claude/CLAUDE.draft.md` for CLAUDE.md)
2. Commit to the `agents` branch and open a PR to `main`
3. GitHub Actions will copy the draft to the live location on merge
4. The human must approve the PR before anything goes live

Note: A file named `CLAUDE.draft.md` may exist in this repository. It represents the previously promoted version of this file and is retained for diffing purposes only. Do not use it as guidance — this file (`CLAUDE.md`) is always the authoritative source.

## Feedback Operating Procedure

This is the current process for collecting and acting on user feedback. It will evolve as the team grows.

**Step 1 — Test**: Human tests the app using the TestingPanel. Submits observations per test case using the in-app feedback form.

**Step 2 — Export**: Export feedback to Excel (button in TestingPanel). Save the file to `User Feedback/`.

**Step 3 — Analyze**: PM reads the raw feedback and writes a brief analysis (`User Feedback/analysis-YYYY-MM-DD.md`) that categorizes issues and recommends the next action.

**Step 4 — Act**: Dev or PM acts on the highest-priority finding. Before fixing a content quality issue, regenerate scenes with the current system prompt first — the issue may already be resolved.

**Step 5 — Iterate**: Regenerate affected test scenes, re-test, repeat.

---

## How to work together
Working norms of this project are evolving. The human, PM and Dev skill should iteratively manage how to work together using this document. The AI should take a driving seat in determining the norms.

## Human suggestions
If you need something to help you iterate on this project more effectively, request it here. Tools, permissions, actions taken in the real world. Anything! Only request when it's clearly necessary and would meaningfully unblock the team.
