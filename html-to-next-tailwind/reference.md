# Reference — HTML → Next.js + Tailwind

Companion to `SKILL.md`. Read when mapping a large HTML prototype or finishing a CSS purge.

## Token mapping cheat sheet

| CSS custom property | Tailwind config key | Example utility |
|---------------------|---------------------|-----------------|
| `--cream` | `colors.cream` | `bg-cream` |
| `--ink` / `--ink-70` | `colors.ink` / `ink.70` | `text-ink-70` |
| `--display` / `--sans` | `fontFamily.display/sans` | `font-display` |
| `--ease` | `transitionTimingFunction.brand` | `ease-brand` |
| `--arch` | `borderRadius.arch` | `rounded-arch` |
| custom mq `760px` | `screens.sm` (override defaults) | `sm:` |
| `.wrap` padding `44px` | `spacing.wrap` | `px-wrap` |
| section padding `130px` | `spacing.section` | `py-section` |

Name utilities after the design language (`sage`, `sand`, `ink`), not generic AI palette names.

## Component extraction heuristics

| HTML pattern | Component |
|--------------|-----------|
| `.wrap` | `Wrap` |
| `.eyebrow` / `.lead` | `Eyebrow` / `Lead` |
| `.btn` + modifiers | `Button` with `variant` / `size` |
| `.arrowlink` | `ArrowLink` + optional `ArrowLinkLabel` |
| logo SVG/mark | `LogoMark` |
| sticky header + burger | `SiteHeader` + sticky/menu hooks |
| footer columns | `SiteFooter` |
| `.rv` + `data-d` | `Reveal` / `RevealLink` + `useRevealOnScroll` |
| `<section id="…">` blocks | one file per section under `components/[page]/` |

## Content vs presentation

Put into `lib/` when:

- Nav links, phone/WhatsApp URLs, CTAs
- Card grids (listings, services, reviews)
- Anything editors will later move to a CMS

Keep one-off layout strings in the section component.

## Interim `site.css` bridge (path B)

If you must ship a working page before utilities are done:

1. Copy only still-needed selectors into `src/styles/site.css`.
2. Import once from `layout.tsx`.
3. Migrate component-by-component; delete rules as you go.
4. When empty, delete the file and the import.

Never leave a permanent dual system (full `site.css` + full Tailwind theme).

## `globals.css` final shape

Keep:

- `@import "tailwindcss"`
- `@config` pointing at `tailwind.config.ts`
- Base: `html` scroll behavior, `body` surface/type color, `h1–h3` display face, `::selection`, reduced-motion scroll override

Drop:

- Duplicated `@theme inline { … }` that mirrors the config
- Legacy `:root` bridges (`--cream: var(--color-cream)`) once unused
- Redundant Preflight overrides (`img`, `a`, `button`) unless the design needs extras
- Dead classes (`.disp` with no JSX usage)

Prefer `@apply` for base theme colors so the config remains the only token table.

## Parity verification prompts

After each migrated chunk, ask the user to check:

- First viewport / hero
- Sticky header + mobile nav
- Hover/focus on buttons and cards
- Scroll reveals (and reduced-motion if relevant)
- Breakpoints used by the design (not only default Tailwind)

## Common pitfalls

- **Border utilities fighting**: a base `border-transparent` can override variant borders — put transparent only on variants that need it.
- **Default screens**: designs often use `760 / 960 / 1120` — override `screens` instead of forcing Bootstrap-like defaults.
- **Client boundaries**: don't mark entire pages `"use client"` for one reveal; wrap small clients.
- **Font variables**: put `next/font` CSS variables on `<html>`, reference them from `fontFamily` in the config.
