---
name: html-to-next-tailwind
description: >-
  Converts static HTML/CSS/JS page designs into a Next.js App Router site with
  Tailwind CSS v4, then incrementally migrates off legacy stylesheets until
  tokens live only in the Tailwind config. Use when converting example HTML,
  design prototypes, or static marketing pages into Next.js components; when
  migrating site.css / custom CSS to Tailwind utilities; or when cleaning up
  duplicated @theme and globals.css after a Tailwind migration.
---

# HTML → Next.js + Tailwind

Convert a static HTML design into a maintainable Next.js App Router app, then finish with a full Tailwind utility migration (no leftover page CSS).

## Operating mode

1. **Preserve visual parity** with the source HTML. Do not redesign unless asked.
2. Work **incrementally**. Prefer migrate → user verify → next piece over big-bang rewrites.
3. Keep **one token source of truth**: `tailwind.config.ts` (or `@theme` — never both).
4. End state: components use Tailwind utilities; `globals.css` is import + config + minimal base only.

## Phase 1 — Audit the HTML

From the source file(s), inventory:

| Item | Extract |
|------|---------|
| Tokens | `:root` colors, fonts, radii, easings, shadows, breakpoints |
| Layout primitives | `.wrap`, `.eyebrow`, `.lead`, buttons, links, logo |
| Sections | Hero, intro, services, etc. — mark IDs/anchors |
| Chrome | Header, footer, floating CTAs, mobile nav |
| Media | `<img>`, background-images, inline/base64 assets |
| Behavior | Scroll reveal, carousels, sticky header, hamburger, reduced-motion |

Report a short plan (components + hooks + lib) before writing code if the page is large.

## Phase 2 — Scaffold Next.js structure

Typical layout:

```text
src/
  app/layout.tsx          # fonts, chrome, globals.css
  app/page.tsx            # compose sections
  app/globals.css         # Tailwind import + base only
  components/ui/          # Button, Wrap, Eyebrow, Lead, Reveal…
  components/layout/      # SiteHeader, SiteFooter, WhatsApp…
  components/home/        # page sections (or /[page]/)
  hooks/                  # useRevealOnScroll, useStickyHeader…
  lib/                    # content, nav, cn()
  public/images/          # extracted assets
tailwind.config.ts        # ALL design tokens
```

**Rules**

- Use `next/font` for Google fonts from the HTML `<link>`.
- Extract images into `public/`; never leave large base64 blobs in components.
- Move copy/lists into `lib/*` when it keeps JSX readable.
- Port vanilla JS to small client hooks; keep section shells as Server Components when possible.
- Shared interactive primitives (`Reveal`, header) are `"use client"`.

## Phase 3 — Tokens in Tailwind (once)

Map CSS variables → `tailwind.config.ts` `theme.extend`:

- colors, fontFamily, fontSize, spacing, maxWidth
- borderRadius, boxShadow, transitionTimingFunction
- screens (keep design breakpoints even if non-default)
- keyframes + animation for hero/cue/spin patterns

`globals.css` should stay thin:

```css
@import "tailwindcss";
@config "../../tailwind.config.ts";

@layer base {
  /* body, headings, selection, smooth scroll / reduced-motion only */
}
```

Do **not** duplicate the same tokens in `@theme inline` and the JS config.

## Phase 4 — Build components (optional interim CSS)

Two valid paths:

**A — Direct Tailwind** (preferred for new work): implement each component in utilities from the start.

**B — Bridge CSS** (useful for huge single-file HTML): port selectors into a temporary `styles/site.css`, wire components with the old class names, then run Phase 5.

Always build in this order:

1. UI primitives (`Wrap`, `Button`, `Eyebrow`, `Lead`, `SectionHead`, `LogoMark`, `ArrowLink`)
2. Layout chrome (header, footer, floating actions)
3. Page sections top → bottom
4. Motion helpers (`Reveal` / scroll observer)

Pause for visual check after primitives and after each major section when the user wants parity review.

## Phase 5 — Incremental CSS → Tailwind migration

For each component (smallest → largest):

1. Replace class-based CSS with equivalent Tailwind utilities.
2. Delete the matching rules from the legacy stylesheet.
3. Confirm nothing else depended on those selectors.
4. Ask the user to verify before the next component.

**Suggested order:** shared UI → chrome → sections → reveal/motion last.

### Reveal pattern (Tailwind, no `.rv` / `.in`)

Prefer React state over `classList`:

```ts
// hook returns { ref, visible }
// visible flips true once via IntersectionObserver
// prefers-reduced-motion → visible immediately
```

```tsx
className={cn(
  "transition-[opacity,transform] duration-[850ms] ease-brand",
  "motion-reduce:translate-y-0 motion-reduce:opacity-100 motion-reduce:transition-none",
  visible ? "translate-y-0 opacity-100" : "translate-y-[26px] opacity-0",
  delay && delayClass[delay],
)}
```

## Phase 6 — Cleanup checklist

Copy and track:

```text
Cleanup:
- [ ] No imports of site.css / page CSS leftovers
- [ ] Legacy stylesheet deleted (or empty file removed)
- [ ] globals.css = import + @config + minimal @layer base
- [ ] No duplicate @theme block mirroring tailwind.config
- [ ] Unused bridge :root vars removed
- [ ] Unused helper classes (.disp, etc.) removed
- [ ] Preflight covers img/a/button resets — don't redeclare unless intentional
- [ ] Build passes
```

## Hard constraints

- Do not invent new sections, cards, or marketing chrome “for completeness.”
- Do not change intentional user tweaks mid-migration (e.g. removed hover motion).
- Do not commit unless asked.
- Match existing project conventions (`cn`, path aliases, naming) when the repo already has them.

## Additional resources

- Detailed token + component mapping notes: [reference.md](reference.md)
