---
name: site-prototype
description: Build a standalone Tailwind-powered HTML prototype for a client website, including a proposed design system (fonts, colors, spacing) when none exists yet. Invoke explicitly when starting a new client site design, e.g. "use site-prototype for [client]".
---

# Site Prototype

Produces a single static HTML file (Tailwind via CDN) that can be sent straight to a client for design sign-off, before any Next.js/Sanity work begins.

## Inputs to gather before starting

Ask the user for anything missing from the conversation:
- Client name / business type
- Brand info: existing logo, colors, fonts, or "none — you choose"
- Reference sites they like (style, not to copy)
- Page structure / sections needed (hero, services, testimonials, etc.)
- Copy they already have, or "placeholder is fine for now"

## Output requirements

1. **A single `.html` file** using Tailwind CDN (`<script src="https://cdn.tailwindcss.com"></script>`). No build step — this is throwaway scaffolding for client review, not production code.
2. **If no design system exists yet** (default/no Tailwind config): don't reach for generic defaults. Actively design one:
   - Pick a font pairing (heading + body) and load via Google Fonts link tag
   - Pick a small color palette (1 primary, 1 accent, neutrals) — real hex/OKLCH values, not `blue-500`
   - Pick a spacing/radius rhythm (e.g. consistent `rounded-xl`, consistent section padding scale)
3. **A short written summary** immediately after the file, listing the design decisions in plain language:
   - Fonts used and why
   - Color values and their role (primary/accent/neutral)
   - Any layout conventions (max-width, section spacing) established
   
   This summary is what gets turned into the real `tailwind.config` later — write it so it stands alone without needing to re-read the HTML.

## What NOT to do

- Don't wire up any framework, don't add React, don't reference Sanity — pure static HTML/CSS/minimal JS only.
- Don't use placeholder gray boxes for images if real image direction was given — use descriptive `<img>` alt text and a real (even if stock/placeholder-service) src so the client sees a believable page.
- Don't over-explain the code back to the user — lead with the file, keep commentary to the design-decision summary.
