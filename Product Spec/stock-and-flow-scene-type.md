# Stock & Flow Scene Type — Spec

*Last updated: 2026-03-11. Owner: PM skill + human.*

---

## Why This Scene Type First

Stock & Flow diagrams (Donna Meadows, *Thinking in Systems*) are the ideal first scene type because:

1. **Business school relevance**: Stocks and flows model inventory, cash, headcount, market share — core MBA concepts.
2. **Constrained visual vocabulary**: Only 5 element types (stock, flow, converter, connector, cloud). The LLM can't go wrong.
3. **Natural teaching sequence**: Introduce stock → add flows → add converters → reveal feedback loops. Maps perfectly to narrated animation.
4. **Proves the architecture**: If we can render this well with engine-controlled layout, the same pattern extends to flow charts, cycles, hierarchies.

---

## Core Insight (for the dev)

The LLM describes the **system** — not the visual. It says "there is a stock called Population, with an inflow called Births." The engine decides where to draw the rectangle, how to route the pipe, where to place the valve symbol, and how to animate the entrance. The LLM never touches coordinates, sizes, or animation types.

---

## Domain Model

Five element types, derived from Meadows/Forrester notation:

| Element | What it represents | Visual form | Max count |
|---|---|---|---|
| **Stock** | An accumulation (measurable at a point in time) | Rectangle with thick border, label centered inside | 3 |
| **Flow** | A rate that changes a stock (inflow adds, outflow drains) | Pipe (double-lined arrow) with a valve (bowtie ⋈) symbol at midpoint | 6 total (max 3 in + 3 out per stock) |
| **Cloud** | Source or sink outside the system boundary | Small cloud shape at the origin/terminus of a flow | Auto-generated (1 per flow endpoint not connected to another stock) |
| **Converter** | An auxiliary variable that influences a flow's rate | Circle with label | 4 |
| **Connector** | An information link showing influence | Thin curved arrow | 6 |

**Structural rules** (the engine enforces these):
- A flow must connect to at least one stock (as inflow or outflow).
- A flow between two stocks has no clouds (stock-to-stock flow).
- A flow from outside the system into a stock gets an auto-generated source cloud.
- A flow from a stock to outside the system gets an auto-generated sink cloud.
- Connectors can go from any element to any flow or converter (but never directly into a stock — per Meadows convention, only flows change stocks).

---

## LLM Input Schema

This is the exact JSON the LLM produces. The engine renders it. No other format is accepted.

```json
{
  "scene_type": "stock_and_flow",
  "version": "0.1",

  "title": "Population Dynamics",

  "colors": {
    "stock": "blue",
    "inflow": "green",
    "outflow": "red",
    "converter": "purple",
    "connector": "orange"
  },

  "transcript": "Every system has stocks. A stock is something you can measure at a point in time. In this system, the stock is Population. Population grows through Births. And it shrinks through Deaths. The Birth Rate determines how fast births happen. And notice — Population itself feeds back into Births. More people means more births. This is a reinforcing feedback loop.",

  "system": {
    "stocks": [
      { "id": "population", "label": "Population" }
    ],
    "flows": [
      {
        "id": "births",
        "label": "Births",
        "type": "inflow",
        "to_stock": "population"
      },
      {
        "id": "deaths",
        "label": "Deaths",
        "type": "outflow",
        "from_stock": "population"
      }
    ],
    "converters": [
      { "id": "birth_rate", "label": "Birth Rate" },
      { "id": "death_rate", "label": "Death Rate" }
    ],
    "connectors": [
      { "from": "birth_rate", "to": "births" },
      { "from": "death_rate", "to": "deaths" },
      { "from": "population", "to": "births", "label": "reinforcing" }
    ]
  },

  "narrative": [
    {
      "trigger_phrase": "the stock is Population",
      "action": "enter",
      "targets": ["population"]
    },
    {
      "trigger_phrase": "grows through Births",
      "action": "enter",
      "targets": ["births"]
    },
    {
      "trigger_phrase": "shrinks through Deaths",
      "action": "enter",
      "targets": ["deaths"]
    },
    {
      "trigger_phrase": "Birth Rate determines",
      "action": "enter",
      "targets": ["birth_rate"]
    },
    {
      "trigger_phrase": "Population itself feeds back",
      "action": "enter",
      "targets": ["population", "births"]
    },
    {
      "trigger_phrase": "reinforcing feedback loop",
      "action": "emphasize",
      "targets": ["population", "births"]
    }
  ]
}
```

### Schema Reference

#### Top level

| Field | Type | Required | Notes |
|---|---|---|---|
| `scene_type` | `"stock_and_flow"` | yes | Tells the engine which renderer to use |
| `version` | `"0.1"` | yes | Schema version for this scene type |
| `title` | string | yes | Displayed at top of canvas; typewrite animation on enter |
| `colors` | ColorScheme | no | Override default colors. Any omitted field uses the default. |
| `transcript` | string | yes | Full narration for TTS. Narrative trigger phrases must be exact substrings. |
| `system` | SystemDefinition | yes | The stock & flow model |
| `narrative` | NarrativeBeat[] | yes | Ordered sequence of visual events, synced to transcript |

#### ColorScheme

All fields optional. Values must be one of: `white`, `black`, `red`, `blue`, `green`, `yellow`, `orange`, `purple`.

| Field | Default | Controls |
|---|---|---|
| `stock` | `blue` | Stock rectangle fill |
| `inflow` | `green` | Inflow pipe, valve, and cloud |
| `outflow` | `red` | Outflow pipe, valve, and cloud |
| `converter` | `purple` | Converter circle fill |
| `connector` | `orange` | Connector arrow stroke |

The engine derives additional colors automatically:
- Stock border: darker shade of stock color
- Stock label text: white (on dark fills) or black (on light fills) — auto-contrast
- Cloud fill: 20% opacity version of the flow's color
- Background: white (not configurable in v0.1)

#### SystemDefinition

| Field | Type | Required | Constraints |
|---|---|---|---|
| `stocks` | Stock[] | yes | 1–3 items |
| `flows` | Flow[] | yes | 1–6 items |
| `converters` | Converter[] | no | 0–4 items |
| `connectors` | Connector[] | no | 0–6 items |

#### Stock

| Field | Type | Required |
|---|---|---|
| `id` | string | yes |
| `label` | string (1–25 chars) | yes |

#### Flow

| Field | Type | Required | Notes |
|---|---|---|---|
| `id` | string | yes | |
| `label` | string (1–25 chars) | yes | |
| `type` | `"inflow"` or `"outflow"` | yes | |
| `to_stock` | string (stock id) | required for inflow | The stock this flow feeds into |
| `from_stock` | string (stock id) | required for outflow | The stock this flow drains from |
| `from_stock` | string (stock id) | optional for inflow | If set, this is a stock-to-stock flow (no source cloud) |
| `to_stock` | string (stock id) | optional for outflow | If set, this is a stock-to-stock flow (no sink cloud) |

**Stock-to-stock flows**: If a flow has both `from_stock` and `to_stock`, it transfers between two stocks. No clouds are generated. Example: "Migration" flowing from "Region A" to "Region B".

#### Converter

| Field | Type | Required |
|---|---|---|
| `id` | string | yes |
| `label` | string (1–25 chars) | yes |

#### Connector

| Field | Type | Required | Notes |
|---|---|---|---|
| `from` | string (any element id) | yes | Source of influence |
| `to` | string (flow or converter id) | yes | Target of influence. Cannot point to a stock directly. |
| `label` | string (1–20 chars) | no | Optional annotation on the connector arrow |

#### NarrativeBeat

| Field | Type | Required | Notes |
|---|---|---|---|
| `trigger_phrase` | string | yes | Exact substring of `transcript`. Case-sensitive. |
| `action` | `"enter"` or `"emphasize"` or `"exit"` | yes | What to do |
| `targets` | string[] (element ids) | yes | Which elements to apply the action to. For `enter`, entering a flow also enters its cloud(s). |

**Narrative rules:**
- An element must be `enter`ed before it can be `emphasize`d or `exit`ed.
- When a flow is entered, its cloud(s) enter simultaneously (no separate beat needed).
- When a connector is entered, both its endpoints must already be visible (entered).
- Multiple targets in one beat act simultaneously.
- The engine selects the animation primitive for each target based on its element type (see Animation Mapping below).

---

## Animation Mapping

The engine assigns animation primitives based on element type and action. The LLM never specifies animations.

| Element type | Enter | Emphasize | Exit |
|---|---|---|---|
| Title | typewrite | — | fade |
| Stock | pop | pulse | fade |
| Flow (pipe + valve) | draw (along pipe direction) | pulse | fade |
| Cloud | fade (simultaneous with its flow) | — | fade |
| Converter | pop | pulse | fade |
| Connector | draw (from source toward target) | flash | fade |

Easing per the animation primitives spec: pop/pulse use spring, draw uses linear, fade uses ease-out (enter) or ease-in (exit), typewrite uses linear.

---

## Layout Algorithm (for the dev)

The engine computes all positions. The LLM provides zero layout information.

### Canvas Setup
- Aspect ratio: 16:9
- Title occupies the top ~10% of canvas height
- System diagram occupies the remaining ~90%

### Layout Strategy: Horizontal Flow Axis

The primary visual metaphor is left-to-right flow, matching how English readers scan.

```
         ┌─────────── Title (typewrite) ──────────────┐
         │                                             │
  [☁]──→─┤⋈├──→──[ STOCK A ]──→──┤⋈├──→──[ STOCK B ]──→──┤⋈├──→──[☁]
         │        ↑                                    │
         │     (Birth Rate)                            │
         │        ↑                                    │
         │     ← ← ← (connector: reinforcing) ← ← ← ←│
         └─────────────────────────────────────────────┘
```

### Step-by-step layout

1. **Place stocks** along the horizontal center line, evenly spaced.
   - Single stock: centered at (50%, 50%)
   - Two stocks: at (33%, 50%) and (67%, 50%)
   - Three stocks: at (25%, 50%), (50%, 50%), (75%, 50%)

2. **Place flows** as pipes connecting to stocks.
   - Inflows approach from the left of the stock.
   - Outflows exit to the right of the stock.
   - For stock-to-stock flows, the pipe goes between the two stocks.
   - Each pipe has a valve symbol (bowtie ⋈) at its midpoint.
   - If multiple inflows/outflows on the same side of a stock, fan them vertically (stagger by ~8% of canvas height).

3. **Place clouds** at the free end of each flow.
   - Inflow source cloud: to the left of the inflow pipe.
   - Outflow sink cloud: to the right of the outflow pipe.
   - Clouds are small (~6% of canvas width).

4. **Place converters** near the flows they influence.
   - Look at which connectors reference each converter → find the target flow → place the converter above or below that flow.
   - Alternate above/below to avoid collisions.
   - If a converter isn't connected to any flow, place it in available space above the stocks.

5. **Place connectors** as curved arrows.
   - Route from source element to target element (flow or converter).
   - Use quadratic Bezier curves to avoid overlapping other elements.
   - Connector arrows are thin and use the connector color.
   - If the connector has a label, place it at the curve's midpoint.

### Collision Avoidance
The engine should check for overlapping elements and nudge them apart. Simple heuristic: after initial placement, run one pass of overlap detection. If two elements overlap, move the smaller one outward along the axis perpendicular to the main flow axis.

---

## Title Animation

The title appears first, before any system elements. It uses the typewrite animation and occupies a horizontal band at the top of the canvas.

- Font: large or xlarge, bold
- Color: black (or darkest color in the scheme)
- Position: top-center
- Duration: first ~2 seconds of the transcript, or until the first narrative beat fires
- The title remains visible throughout the presentation (it does not exit)

If the transcript begins with introductory narration before the first beat, the title animates during that intro.

---

## Rendering Details (for the dev)

### Stock
- Rectangle with rounded corners (4px radius)
- Border width: 3px
- Fill: stock color from scheme
- Label: centered, white text (if fill is dark) or black text (if fill is light)
- Size: ~18% canvas width × ~14% canvas height (adjusts if 3 stocks need to fit)

### Flow (Pipe)
- Two parallel lines (pipe walls) with gap between them, ~3% canvas height thick
- Direction: horizontal, from source to target
- Valve: bowtie/hourglass shape at the pipe midpoint, filled with the flow's color
- Label: below the pipe, in the flow's color
- Flow line uses the flow's color (inflow = green default, outflow = red default)

### Cloud
- Organic cloud shape (3-4 overlapping circles forming a blob)
- Fill: flow color at 20% opacity
- Border: flow color at 40% opacity
- No label
- Size: ~6% × 5% of canvas

### Converter
- Circle
- Border: 2px, converter color
- Fill: converter color at 30% opacity (translucent)
- Label: centered inside, converter color at full opacity
- Size: ~8% canvas width diameter

### Connector
- Thin arrow (1.5px stroke)
- Quadratic Bezier curve from source to target
- Small arrowhead at the target end
- Color: connector color from scheme
- If a label exists, render it at the curve's midpoint in a small font, same color

---

## Example: Simple (Bathtub)

**LLM output:**
```json
{
  "scene_type": "stock_and_flow",
  "version": "0.1",
  "title": "The Bathtub",
  "transcript": "Imagine a bathtub. The water in the tub is a stock. You can measure it at any moment. The faucet is an inflow. It adds water. The drain is an outflow. It removes water. If the faucet runs faster than the drain, the water level rises. If the drain is faster, it falls.",
  "system": {
    "stocks": [
      { "id": "water", "label": "Water in Tub" }
    ],
    "flows": [
      { "id": "faucet", "label": "Faucet", "type": "inflow", "to_stock": "water" },
      { "id": "drain", "label": "Drain", "type": "outflow", "from_stock": "water" }
    ]
  },
  "narrative": [
    { "trigger_phrase": "water in the tub is a stock", "action": "enter", "targets": ["water"] },
    { "trigger_phrase": "faucet is an inflow", "action": "enter", "targets": ["faucet"] },
    { "trigger_phrase": "drain is an outflow", "action": "enter", "targets": ["drain"] },
    { "trigger_phrase": "faucet runs faster", "action": "emphasize", "targets": ["faucet"] },
    { "trigger_phrase": "drain is faster", "action": "emphasize", "targets": ["drain"] }
  ]
}
```

**What the engine renders:**
1. Title "The Bathtub" typewrite across the top.
2. "water in the tub is a stock" → blue rectangle pops in at center labeled "Water in Tub".
3. "faucet is an inflow" → green pipe draws in from left, cloud fades in at far left, valve appears at pipe midpoint. Label "Faucet" fades in below.
4. "drain is an outflow" → red pipe draws in to right, cloud fades in at far right, valve appears. Label "Drain" fades in below.
5. "faucet runs faster" → faucet pipe and valve pulse.
6. "drain is faster" → drain pipe and valve pulse.

---

## Example: Complex (Population with Feedback)

```json
{
  "scene_type": "stock_and_flow",
  "version": "0.1",
  "title": "Population Dynamics",
  "colors": {
    "stock": "blue",
    "inflow": "green",
    "outflow": "red",
    "converter": "purple",
    "connector": "orange"
  },
  "transcript": "Let us model population. Population is a stock — the total number of people alive right now. Births are an inflow. Each year, new people are born, adding to the population. Deaths are an outflow. People die, reducing the population. The birth rate controls how many births occur. The death rate controls how many deaths occur. But here is the key insight. Population itself influences births. More people means more births. This creates a reinforcing feedback loop. The system feeds on itself.",
  "system": {
    "stocks": [
      { "id": "pop", "label": "Population" }
    ],
    "flows": [
      { "id": "births", "label": "Births", "type": "inflow", "to_stock": "pop" },
      { "id": "deaths", "label": "Deaths", "type": "outflow", "from_stock": "pop" }
    ],
    "converters": [
      { "id": "birth_rate", "label": "Birth Rate" },
      { "id": "death_rate", "label": "Death Rate" }
    ],
    "connectors": [
      { "from": "birth_rate", "to": "births" },
      { "from": "death_rate", "to": "deaths" },
      { "from": "pop", "to": "births", "label": "reinforcing" }
    ]
  },
  "narrative": [
    { "trigger_phrase": "Population is a stock", "action": "enter", "targets": ["pop"] },
    { "trigger_phrase": "Births are an inflow", "action": "enter", "targets": ["births"] },
    { "trigger_phrase": "Deaths are an outflow", "action": "enter", "targets": ["deaths"] },
    { "trigger_phrase": "birth rate controls", "action": "enter", "targets": ["birth_rate"] },
    { "trigger_phrase": "death rate controls", "action": "enter", "targets": ["death_rate"] },
    { "trigger_phrase": "Population itself influences births", "action": "enter", "targets": ["pop"] },
    { "trigger_phrase": "reinforcing feedback loop", "action": "emphasize", "targets": ["pop", "births"] }
  ]
}
```

---

## What the LLM Does NOT Specify

The LLM's job is pure domain modeling + narration. These are all handled by the engine:

| Concern | Owner |
|---|---|
| Element positions | Engine (layout algorithm) |
| Element sizes | Engine (proportional to canvas + element count) |
| Animation types | Engine (based on element type, per animation mapping table) |
| Animation timing | Engine (derived from TTS phrase duration) |
| Cloud generation | Engine (auto-created for flow endpoints not connected to another stock) |
| Font sizes | Engine |
| Arrow routing | Engine (Bezier curves avoiding collisions) |
| Contrast / accessibility | Engine (auto-contrast text on fills) |

---

## Constraints Summary

| Constraint | Limit | Rationale |
|---|---|---|
| Stocks | 1–3 | More than 3 makes the canvas unreadable |
| Flows | 1–6 | Max 3 per side per stock; keeps pipes from tangling |
| Converters | 0–4 | Auxiliary variables; too many clutters |
| Connectors | 0–6 | Information arrows; more creates visual noise |
| Label length | 1–25 chars | Needs to fit inside shapes |
| Connector label | 1–20 chars | Sits on an arrow; must be short |
| Title length | 1–50 chars | Top of canvas, single line |

---

## Open Questions

1. **Stock-to-stock flows**: The layout algorithm needs to handle pipes that go between two stocks (no clouds). Should these always route in a straight horizontal line, or should they arc above/below?
2. **Feedback loop annotation**: When a connector creates a feedback loop (A→B→A cycle), should the engine automatically label it as "reinforcing" or "balancing"? Or is that the LLM's job via connector labels?
3. **Multi-stock ordering**: If the LLM defines 3 stocks, should their left-to-right order on canvas follow the order in the `stocks` array? Or should the engine infer order from flow connections (upstream → downstream)?
4. **Valve style**: Bowtie (⋈) is the standard Forrester notation. Is this recognizable enough for business school students, or should we use a simpler visual (like a tap/faucet icon)?
