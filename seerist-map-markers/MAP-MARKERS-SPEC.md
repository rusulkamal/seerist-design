# Seerist Map Markers — Design Specification

**Product:** Seerist Monitor  
**Author:** Design  
**Prototype:** https://rusulkamal.github.io/seerist-design/seerist-map-markers/  
**Last updated:** May 2026

---

## Overview

The map is the primary surface for understanding what is happening in the world. Markers are the core visual language of that surface. Every marker on the map answers two questions instantly:

1. **What kind of thing is this?** (event, analysis, news, asset)
2. **How much should I trust it?** (Seerist-verified vs. third-party)

The system uses **shape**, **border style**, **colour**, and **icon** — each carrying exactly one meaning — so users can read the map at a glance without consulting the legend.

---

## Visual Language Rules

| Property | What it communicates |
|---|---|
| **Shape** | Source type — who produced this |
| **Border style** | Verification status — how confirmed it is |
| **Border / icon colour** | Severity — how dangerous |
| **Icon** | Category — what kind of incident |
| **Badge** | Special status modifier (breaking, future) |

---

## 1. Event Markers

Events are incidents on the ground. They are **produced and curated by Seerist**. This makes them the most authoritative markers on the map.

### Shape
**Circle.** A circle communicates a point in space where something physically happened.

### Size
34 × 34 px

### Statuses

There are four event statuses. The border style is how users read them:

| Status | Border | Badge | Meaning |
|---|---|---|---|
| **Breaking** | Solid 2px | ⚡ Red badge (top-right) | Happening right now, just confirmed |
| **Verified** | Solid 2px | — | Confirmed by Seerist |
| **Unverified** | Dashed 2px | — | Reported but not yet confirmed |
| **Future** | Solid 2px | 📅 Calendar badge (bottom-right) | Scheduled or anticipated event |

> **Why dashes for unverified?** Dashes visually imply incompleteness. Users intuitively read a dashed outline as "not fully confirmed" without needing a tooltip.

### Severity Colours

Severity is communicated through the border and icon colour. The background is always white so the colour reads clearly.

| Severity | Colour | Hex |
|---|---|---|
| **High** | Red | `#dc2626` |
| **Medium** | Orange | `#ea580c` |
| **Low / None** | Stone | `#78716c` |

### Breaking Pulse

Breaking events have an animated pulse ring that radiates outward from the marker. This makes them immediately visible on a busy map. The pulse is subtle — it signals urgency without being distracting.

- Animation: expands from 0.9× to 1.8× the marker size, fading out
- Duration: 2.6 seconds per cycle
- Each breaking marker pulses at a slightly different offset so they don't all fire in sync

### Icons

The icon inside every event marker represents its **category**. The same icon appears in the event list panel for consistency.

| Icon | Category |
|---|---|
| ✈️ Jet fighter | Conflict & Military |
| 🔫 Person with rifle | Terrorism |
| 📢 Bullhorn | Unrest & Protests |
| ⚓ Anchor | Maritime |
| 🌀 Hurricane | Disaster |
| 🛡️ Shield | Crime |
| 🏛️ Landmark | Political Risk |
| ⚖️ Gavel | Official Actions |
| ❤️ Heart pulse | Health |

---

## 2. News & Social Markers

News & Social markers represent **third-party reporting** — articles, social media posts, and wire feeds about a topic in an area. They are less authoritative than Seerist Events because they have not been independently verified by Seerist analysts.

### Shape
**Rounded square (squircle).** Intentionally different from the circular event markers. When users see a squircle, they immediately know this is not a Seerist ground-truth event — it is media coverage of a topic.

### Size
34 × 34 px — same as events so they have equal visual weight on the map.

### Style
- White background
- Slate border (`#94a3b8`)
- Slate icon colour

### Icons
Like events, the icon shows the **category** of the content (bullhorn for unrest, anchor for maritime, etc.). This allows users to compare: "there's a bullhorn circle here (verified Seerist protest event) and a bullhorn squircle nearby (news coverage of protests in this area)."

### Visibility
Shown from zoom level 2 and above.

---

## 3. Analysis Markers

Analysis markers represent **Seerist Analyst reports** — in-depth written assessments about a region or topic. These are produced by Seerist, making them high-trust content, but they are documents rather than events.

### Shape
**Rounded square (squircle).** Like News & Social, the squircle shape communicates "this is content about an area" rather than "this is something that happened at this point."

### Style
- White background
- Green border (`#10b981`) — green distinguishes Seerist-authored content from third-party news
- Green icon colour (matches border)
- Icon: document / file-lines

### How it differs from News & Social
Both are squircles, but the **green border = Seerist** vs **slate border = third-party**. Users learn this colour rule quickly because it is consistent across the whole system.

### Visibility
Shown from zoom level 2 and above.

---

## 4. Asset Markers

Assets are **the user's own tracked locations** — offices, field teams, embassies, facilities, vessels. They are entirely separate from events and intel.

### Shape
**Circle with coloured fill.** Assets use a solid colour fill (no white background) which makes them visually distinct from every other marker type at a glance.

### Size
32 × 32 px

### Style
Each asset type has its own colour set by the organisation (e.g. red for crisis teams, blue for logistics, teal for facilities). The icon inside represents the asset type (building, users, anchor, etc.).

### Behaviour
- Always visible at all zoom levels — assets are never clustered
- Never affected by the event type filters
- Toggled on/off via the Assets layer in the Layers panel

---

## 5. Cluster Markers

When multiple events are geographically close together at the current zoom level, they are merged into a **cluster bubble** to prevent the map from becoming unreadable.

### Appearance
- Circle with a count number inside
- Size scales with event count: small (≤7 events), medium (8–15), large (16+)

### Severity Colour
The cluster bubble adopts the colour of the **highest-severity event inside it**:

| Highest severity inside | Cluster colour |
|---|---|
| High | Red border & text |
| Medium | Orange border & text |
| None / Low | Slate border & text |

This means a red cluster bubble immediately signals "there are high-severity events in this area" before the user even clicks.

### Behaviour
- Zoom in to break clusters apart into individual markers
- Clusters only apply to non-breaking events — **breaking events are always shown individually**, never hidden inside a cluster
- Cluster position is the geographic centre (centroid) of all member events
- Hovering a cluster shows a tooltip with a breakdown of event types inside

### Zoom thresholds
| Zoom level | Cluster radius |
|---|---|
| < 2.5 | Very aggressive (world view) |
| 2.5 – 4 | Moderate |
| 4 – 6 | Loose |
| 6 – 7 | Minimal |
| 7+ | No clustering |

---

## 6. Same-Location Stack

When two or more **non-breaking events exist at the exact same coordinates**, they are shown as a stacked group marker rather than overlapping circles.

### Appearance
Three stacked rings with a number on the front showing how many events are at that location.

### Behaviour
- Clicking opens a list view in the popover showing all events at that location
- Breaking events at the same location are shown separately (always individual) — the count on the stack only reflects non-breaking events

---

## Summary: Shape = Source

The single most important rule in the marker system:

| Shape | Source |
|---|---|
| ⬤ Circle | Seerist Event — something happened here |
| ▣ Green squircle | Seerist Analysis — we wrote about this area |
| ▣ Slate squircle | News & Social — third-party reporting on this area |
| ⬤ Filled circle | Asset — your organisation has something here |
| ⬤ Numbered circle | Cluster — multiple events in this area, zoom to expand |

---

## Filtering

Users can show/hide markers by type using the **Layers panel** (top-right of map):

| Filter | Controls |
|---|---|
| Breaking | Breaking event markers |
| Verified | Verified event markers |
| Unverified | Unverified event markers |
| Future | Future event markers |
| News & Social | News & Social squircle markers |
| Analysis | Analysis squircle markers |
| Assets layer | All asset markers |
| Incidents layer | All event markers |

---

## Interactions

| Action | Result |
|---|---|
| Click any event marker | Opens detail popover with title, location, time, source, and actions |
| Click a stack marker | Opens a list of all events at that location |
| Click a cluster | (Planned) Zooms in to expand the cluster |
| Hover an analysis / news marker | Shows a tooltip with the document or article title |
| Click map background | Closes any open popover |
