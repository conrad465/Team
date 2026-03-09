# Whiteboard AI — Product Spec

> **Owner:** Product Dev Agent (Conrad reviews and approves)
> **Last updated:** 2026-03-09
> **Version:** 0.1 (PoC → v1 transition)

---

## 1. Product Vision

Whiteboard AI is a rendering framework that allows any LLM to produce
Khan-Academy-style animated whiteboard presentations — without slow, expensive
video generation. An LLM writes a structured JSON scene; the engine plays it back
with synchronized TTS audio and canvas animations.

**Core value proposition:**
> Any concept, explained visually and audibly, in seconds, at near-zero cost.

---

## 2. Target Users

### Primary (now)
- **Conrad** — sole user, business school student, building the tool as a learning
  and productivity aid. Uses it to get visual explanations of complex concepts from LLMs.

### Target (next phase)
- Knowledge workers and students who want richer answers from LLMs
- Educators who want quick illustrative explainers without video production overhead
- Developers building LLM-powered apps who want a visual output layer

---

## 3. User Stories (prioritized)

### Must Have (v1)
- [ ] **US-01:** As a user, I can type a question/topic and receive a complete animated
  whiteboard explanation without leaving the app (no copy/paste to external LLM)
- [ ] **US-02:** As a user, the scene starts playing within a reasonable time after I submit
  my question (target: <15s end-to-end)
- [ ] **US-03:** As a user, animations stay in sync with the spoken words
- [ ] **US-04:** As a user, I can pause, rewind, and replay at any time
- [ ] **US-05:** As a user, I can see the transcript with the current spoken phrase highlighted

### Should Have (v1)
- [ ] **US-06:** As a user, I can give feedback on a scene (thumbs up/down + comment)
  and it persists across sessions
- [ ] **US-07:** As a user, I can choose from multiple TTS voices
- [ ] **US-08:** As a user, I can see a history of my recent scenes

### Nice to Have (post-v1)
- [ ] **US-09:** As a user, I can share a scene with a link
- [ ] **US-10:** As a user, the app works on mobile
- [ ] **US-11:** As a user, I can export a scene as video

---

## 4. Technical Architecture

### Core rendering pipeline
```
User query
    → LLM (Claude API) → Scene JSON
    → SceneLoader (validate + parse)
    → SceneGraph (live element store)
    → AnimationController + Renderer (rAF draw loop)
    → TTS Engine (Web Speech API)
    → BoundaryTracker (onboundary → action triggers)
```

### Key constraints
- **Canvas 2D only** — no WebGL, no DOM elements on canvas
- **Web Speech API** — `onboundary` word events; Chrome/Edge reliable, Firefox limited
- **No server** — pure static site (GitHub Pages); LLM calls go direct from browser
- **TypeScript strict** — `erasableSyntaxOnly: true`
- **Schema version:** `"1.0"` — backwards compatibility required for saved scenes

### Tech stack
- TypeScript 5 + Vite (vanilla-ts)
- HTML5 Canvas 2D
- Web Speech API
- Anthropic SDK (browser build)

---

## 5. Scene Language (v1.0 summary)

The LLM contract. Full spec in `public/system-prompt.md`.

- **Elements:** `shape` (rectangle, triangle, circle, line, arrow) | `text`
- **Colors:** 8 named colors (white black red blue green yellow orange purple)
- **Positions:** `canvas` (absolute %) | `relative` (anchored to another element) | `connected` (arrow between elements)
- **Actions:** `create` | `edit` | `delete`
- **Animations:** `fade_in` | `pop_in` | `pop_highlight` | `draw_in` | `typewriter`
- **Timing:** `trigger_phrase` — exact substring of `transcript`; action fires when TTS reaches that phrase

---

## 6. Known Issues / Limitations

| # | Issue | Severity | Fix status |
|---|-------|----------|-----------|
| KI-01 | Firefox `onboundary` events unreliable | Medium | Mitigated (handleEnd flush) |
| KI-02 | No in-app LLM integration — user must copy/paste scene JSON | High | Planned (US-01) |
| KI-03 | Feedback data stored in browser localStorage only — lost on clear | Medium | Planned |
| KI-04 | No mobile layout | Low | Post-v1 |
| KI-05 | `connected` position type not validated against existing element IDs at load time | Low | Planned |

---

## 7. Success Metrics

### PoC → v1 gate (Conrad's personal bar)
- [ ] Can generate a useful whiteboard explanation for a real business school concept
  end-to-end in <15 seconds without leaving the app
- [ ] TTS + animation sync feels natural (no obvious lag or drift)
- [ ] Used it 10+ times in a real study/work context

### v1 → scale gate (not yet)
- TBD — depends on feedback from additional users

---

## 8. Out of Scope (explicit)

- Video export (post-v1)
- Multi-user collaboration
- Custom fonts or imported images
- Audio/video element types
- Any server-side rendering or state

---

## 9. Roadmap

### v1 — "Self-contained tool" (current target)
- In-app LLM integration (US-01)
- Persistent feedback (US-06)
- Voice selector (US-07)
- Basic scene history (US-08)
- Improved error messages for bad scene JSON

### v1.1 — "Shareable"
- Scene sharing via URL (encoded JSON or API)
- Improved mobile layout

### v2 — "Platform"
- Scene library / gallery
- Multi-user support
- Analytics dashboard
