---
name: tailwind-html-to-nextjs
description: Convert an approved static HTML/Tailwind prototype into componentized, statically-rendered Next.js pages matching an existing project's conventions — no CMS/data-fetching yet. Invoke explicitly once a client has approved a prototype, e.g. "use tailwind-html-to-nextjs to migrate this".
---

# HTML to Next.js

Turns a client-approved static HTML page into properly structured Next.js components inside an existing codebase. This step deliberately excludes Sanity/CMS wiring — the goal is a pixel-matched, statically-rendered baseline first.

## Requirements

- **`code-style` skill** from the `frontend` plugin ([claudepluginhub](https://www.claudepluginhub.com/skills/fubits1-frontend-plugins-frontend/code-style), source: https://github.com/fubits1/svelte-skills). Install with `npx claudepluginhub fubits1/svelte-skills --plugin frontend`. Invoke `/frontend:code-style` before writing components and follow it for naming, markup, forms and TypeScript. Where it conflicts with this skill or the existing project's conventions (e.g. its CSS-nesting rules vs. Tailwind utilities, Svelte-specific guidance), the project and this skill win. If the plugin isn't installed, tell the user and ask whether to continue without it.

## Inputs to gather before starting

- The approved HTML file (paste or upload)
- The final design system summary from the prototype step (fonts, colors, spacing) — if missing, extract it from the HTML directly and confirm with the user before proceeding
- The existing project's conventions: component folder structure, naming patterns (PascalCase vs kebab, `components/` layout), whether it uses the App Router, TypeScript vs JS, any existing shared components (Button, Container, etc.) to reuse instead of recreating

If any of these conventions aren't visible in context, look at the actual project files before writing new components — don't guess at structure.

## Process

1. **Update `tailwind.config`** (or `globals.css` theme tokens if using Tailwind v4) with the approved design system — fonts, colors, spacing/radius — so utility classes in the new components can use semantic names (`bg-primary`) rather than raw values copied from the prototype.
2. **Break the HTML into components** along natural section boundaries (Hero, ServiceCard, TestimonialGrid, Footer, etc.), matching the granularity and location conventions of the existing codebase.
3. **Hardcode all content as-is for now** — text, images, links stay static. Do not invent props or dynamic data structures; that happens in the Sanity step.
4. **Preserve responsive behavior and interactive states** (hover, focus, any JS interactivity) from the original HTML.
5. **Checkpoint**: after generating, describe how to verify pixel-parity against the original HTML (e.g. side-by-side, or note specific sections to double check) — don't just assume it matches.

## What NOT to do

- Don't add Sanity client setup, GROQ queries, or `fetch`/`getStaticProps`/server component data loading — this step is static only.
- Don't invent a different design system than what was approved — reuse the exact values from the prototype's design summary.
- Don't restructure the existing project's conventions to fit the new page; fit the new page to the existing conventions.
