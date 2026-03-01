---
name: Interface Design
dcc_uri: dev/rules/interface-design
description: Interface design rules for dashboards, admin panels, SaaS apps, and tools. NOT for marketing sites or landing pages.
version: '1.0.0'
dcc_definition_type: agent
dcc_tags:
  - dev
  - frontend
  - design
  - ui
  - interface
---

# Interface Design Rules

These rules apply when building interface design — dashboards, admin panels, SaaS apps, tools, settings pages, data interfaces. NOT for marketing sites, landing pages, or campaigns.

---

## The Core Problem

AI generates generic output by default. Your training has seen thousands of dashboards. The patterns are strong. You can explore the domain, name a signature, state your intent — and still produce a template.

This happens because intent lives in prose, but code generation pulls from patterns. Defaults hide in places that feel like infrastructure: typography, navigation, data presentation, token names. Everything is design. The moment you stop asking "why this?" is the moment defaults take over.

---

## Intent First (Required Before Code)

Answer these before touching code. Not in your head — state them explicitly:

**Who is this human?** Not "users." The actual person. Where are they when they open this? What's on their mind? A teacher at 7am is not a developer debugging at midnight is not a founder between meetings. Their world shapes the interface.

**What must they accomplish?** Not "use the dashboard." The verb. Grade submissions. Find the broken deployment. Approve the payment. The answer determines what leads, what follows, what hides.

**What should this feel like?** Say it with words that mean something. Not "clean and modern" — those mean nothing. Warm like a notebook? Cold like a terminal? Dense like a trading floor? Calm like a reading app? The answer shapes color, type, spacing, density — everything.

If you cannot answer these with specifics, stop and ask the user. Do not guess. Do not default.

---

## Product Domain Exploration (Required)

Do not propose any direction until you produce all four:

**Domain:** 5+ concepts, metaphors, vocabulary from this product's world. Not features — territory.

**Color world:** What colors exist naturally in this product's domain? If this product were a physical space, what would you see? List 5+ colors that belong there and nowhere else.

**Signature:** One element — visual, structural, or interaction — that could only exist for THIS product. If you can't name one, keep exploring.

**Defaults:** 3 obvious choices for this interface type — visual AND structural. You can't avoid patterns you haven't named.

Your direction must explicitly reference all four. Test: Remove the product name from your proposal. Can someone identify what this is for? If not, it's generic.

---

## Every Choice Must Be A Choice

For every decision, you must explain WHY:
- Why this layout and not another?
- Why this color temperature?
- Why this typeface?
- Why this spacing scale?
- Why this information hierarchy?

If your answer is "it's common" or "it's clean" or "it works" — you haven't chosen, you've defaulted.

**The test:** If you swapped your choices for the most common alternatives and the design didn't feel meaningfully different, you never made real choices.

**Sameness is failure.** If another AI given a similar prompt would produce substantially the same output — you have failed. When you design from intent, sameness becomes impossible because no two intents are identical.

---

## The Mandate (Before Showing Output)

Before showing the user, look at what you made. Ask: "If they said this lacks craft, what would they mean?" That thing you just thought of — fix it first.

Run these checks:

**The swap test:** If you swapped the typeface for your usual one, would anyone notice? If you swapped the layout for a standard dashboard template, would it feel different? The places where swapping wouldn't matter are the places you defaulted.

**The squint test:** Blur your eyes. Can you still perceive hierarchy? Is anything jumping out harshly? Craft whispers.

**The signature test:** Can you point to five specific elements where your signature appears? Not "the overall feel" — actual components.

**The token test:** Read your CSS variables out loud. Do they sound like they belong to this product's world, or could they belong to any project?

If any check fails, iterate before showing.

---

## Craft Foundations

### Subtle Layering

Surfaces stack. A dropdown sits above a card which sits above the page. Build a numbered system — base, then increasing elevation levels. Each jump should be only a few percentage points of lightness. You can barely see the difference in isolation, but when surfaces stack, the hierarchy emerges.

In dark mode, higher elevation = slightly lighter. In light mode, higher elevation = slightly lighter or uses shadow. Whisper-quiet shifts you feel rather than see.

### Token Architecture

Every color traces back to primitives: foreground (text hierarchy), background (surface elevation), border (separation hierarchy), brand, semantic (destructive, warning, success). No random hex values.

**Text Hierarchy:** Four levels — primary, secondary, tertiary, muted. Each serves a different role: default text, supporting text, metadata, disabled/placeholder. If you're only using two, your hierarchy is too flat.

**Border Progression:** A scale that matches intensity to importance — standard separation, softer separation, emphasis, maximum emphasis. Not every boundary deserves the same weight.

**Control Tokens:** Form controls need dedicated tokens for backgrounds, borders, focus states. Don't reuse surface tokens.

### Spacing

Pick a base unit and stick to multiples. Build a scale for different contexts — micro spacing for icon gaps, component spacing within buttons/cards, section spacing between groups, major separation between distinct areas. Random values signal no system.

### Depth

Choose ONE approach and commit:
- **Borders-only** — Clean, technical. For dense tools.
- **Subtle shadows** — Soft lift. For approachable products.
- **Layered shadows** — Premium, dimensional. For cards that need presence.
- **Surface color shifts** — Background tints establish hierarchy without shadows.

Do not mix approaches.

### Typography

Build distinct levels distinguishable at a glance. Headlines need weight and tight tracking. Body needs comfortable weight for readability. Labels need medium weight that works at smaller sizes. Data needs monospace with tabular number spacing. Don't rely on size alone — combine size, weight, and letter-spacing.

### States

Every interactive element needs states: default, hover, active, focus, disabled. Data needs states: loading, empty, error. Missing states feel broken.

### Navigation Context

Screens need grounding. A data table floating in space feels like a component demo, not a product. Include navigation showing where you are, location indicators, user context.

---

## Prohibited Patterns

- Harsh borders — if borders are the first thing you see, they're too strong
- Dramatic surface jumps — elevation changes should be whisper-quiet
- Inconsistent spacing — the clearest sign of no system
- Mixed depth strategies — pick one approach and commit
- Missing interaction states — hover, focus, disabled, loading, error
- Dramatic drop shadows — shadows should be subtle
- Large radius on small elements
- Pure white cards on colored backgrounds
- Thick decorative borders
- Gradients and color for decoration — color should mean something
- Multiple accent colors — dilutes focus
- Different hues for different surfaces — keep the same hue, shift only lightness

---

## Workflow

**Communication:** Be invisible. Never announce "I'm in ESTABLISH MODE" or "Let me check system.md". Jump into work. State suggestions with reasoning.

**Suggest + Ask:** Lead with exploration and recommendation:
```
Domain: [5+ concepts from the product's world]
Color world: [5+ colors from this domain]
Signature: [one element unique to this product]
Rejecting: [default 1] → [alternative], [default 2] → [alternative], [default 3] → [alternative]

Direction: [approach that connects to the above]
```

Then ask: "Does that direction feel right?"

**If system.md exists:** Read `.interface-design/system.md` and apply. Decisions are made.

**If no system.md:**
1. Explore domain — produce all four required outputs
2. Propose — direction must reference all four
3. Confirm — get user buy-in
4. Build — apply principles
5. Evaluate — run the mandate checks before showing
6. Offer to save to `.interface-design/system.md`

---

## System Persistence

After completing a task, always offer to save patterns to `.interface-design/system.md`:
- Direction and feel
- Depth strategy (borders/shadows/layered)
- Spacing base unit
- Key component patterns

Save patterns when used 2+ times, reusable across the project, or have specific measurements worth remembering. Don't save one-off components or temporary experiments.
