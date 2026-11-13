---
name: "wire-sanity"
description: "Turn static Next.js components into Sanity-driven content, starting with schema design (reviewed before any code) followed by GROQ queries and component wiring. Invoke explicitly after Next.js components are built, e.g. \"use wire-sanity to hook this up\"."
---
 
# Wire Sanity
 
Connects already-built, statically-rendered Next.js components to Sanity CMS. This is schema-first: schemas are proposed and approved before any query or component code is touched.
 
**Default architecture: page-like documents are composed with a page builder.** Every routable page document (`page`, landing pages, the homepage) gets a `pageBuilder` array of reorderable block types, following Sanity's guide: https://www.sanity.io/docs/developer-guides/how-to-use-structured-content-for-page-building. The page's sections become blocks; the page document itself holds only page-level data (title, slug, SEO). Deviate only when the user asks to, or for the exceptions listed under "When not to use the page builder".
 
## Inputs to gather before starting
 
- The static Next.js components/pages to wire up
- A content map, if one exists. Classify every piece of content into one of four buckets:
  - **Static UI chrome** — stays hardcoded (button labels in the UI kit, decorative elements, layout wrappers).
  - **Page sections → page builder blocks** — each visual section of a page (hero, text + image, feature grid, testimonials strip, CTA, FAQ, gallery, form, logo wall…) becomes an `object` block type in the `pageBuilder` array. If the same section component appears on several pages, it is one block type, not one per page.
  - **Repeatable content → document types** — entities that exist independently of any one page and may appear in several places (testimonials, services, team members, projects, promotions). Blocks *reference* these rather than duplicating them.
  - **Global singletons** — site-wide content that is not part of a page's composition (site settings, navigation, footer, contact details, default SEO). These stay singleton documents, not page builder blocks.
- If no content map exists, generate one first by walking through the components and asking the user to confirm the split before proposing schemas. Present it as: for each page, the ordered list of sections → block type names; plus the lists of document types and singletons.
- Sanity project conventions already in use (existing schema file structure, naming, Studio v3+, GROQ vs GraphQL, any existing shared types like `seo`, `link`, `cta`, existing block types or an existing `pageBuilder`). Reuse and extend what exists rather than creating parallel types.
## Page builder modeling rules
 
- **One shared `pageBuilder` type.** Define it once as a named `array` type (e.g. `schemaTypes/objects/pageBuilder.ts`) and use it as a field on every page-like document, so all pages draw from the same block library. Use `defineArrayMember` for each block.
- **Each block is a named, registered `object` type** in its own file (e.g. `schemaTypes/blocks/heroBlock.ts`), registered in `schemaTypes`. No anonymous inline objects.
- **Objects by default, references for reuse.** Content that only lives on this page goes in the block's own fields. Content reused across pages (testimonials, promotions, team members) lives in its own document type and the block holds `reference` fields to it (e.g. a `testimonialsBlock` with `testimonials: array of reference → testimonial`).
- **Model for meaning, not presentation.** No color pickers, alignment toggles, column counts, padding or "variant: dark/light" fields unless the user explicitly asks and confirms editors genuinely need that choice. Presentation decisions belong in the frontend (e.g. a hero renders its heading as `<h1>` when it is the first block, `<h2>` otherwise). If a genuine semantic variant exists (e.g. form type: newsletter / contact), model it as a string `options.list` describing *what* it is, not how it looks.
- **Good editor UX is part of the schema:**
  - Every block gets an `icon` (from `@sanity/icons`) and a `preview` with `select`/`prepare` that shows a meaningful title, the block type as subtitle, and an image when available.
  - Images use `options: {hotspot: true}` and an `alt` field (with validation where the design relies on the image).
  - Use `options.insertMenu` on the `pageBuilder` array (with `filter: true`, and `groups` when there are more than ~6 block types; `views` with list/grid when preview images exist).
- **Every URL is an internal-or-external link.** Never model a link as a plain `url`/`string` field. Every link (CTAs, nav and footer items, card links, links inside blocks, and `link` annotations in Portable Text) uses one shared `link` object (and a `cta` object = `label` + the same link fields) with:
  - `linkType`: radio `internal` | `external`
  - `internalLink`: `reference` → page-like documents (hidden unless `internal`)
  - `href`: absolute URL, `mailto:` or `tel:` (hidden unless `external`)
  - validation that requires only the field for the active `linkType`
  - optional `openInNewTab` boolean, only if the design needs it

  **If the Studio already has a link/CTA type with internal/external handling, reuse it as-is** — keep its field names and behavior, don't create a parallel type or migrate existing content. If it exists but only supports one kind, extend it with the missing option rather than replacing it. Expand links with one shared GROQ fragment (`internalLink->{ _type, "slug": slug.current }`) and resolve them in the frontend with a single `resolveHref(link)` helper.
- **Page document shape:** `title`, `slug` (validated, unique), `pageBuilder`, and `seo` (reuse an existing `seo` type if present). The homepage is a `page` document chosen via a reference in site settings (or a reserved slug, whichever the project already does).
### When not to use the page builder
 
- **Structured content types with a fixed template** — blog posts, case studies, team members, services detail pages where every entry shares the same layout. Give these fixed fields (plus Portable Text for long-form body). Optionally add a `pageBuilder` field for *extra* sections below the fixed template if the design calls for it, but don't replace the structured fields with blocks.
- **Global singletons** (settings, nav, footer).
- If the user explicitly prefers fixed-field pages for a very small site, follow that, but mention the trade-off once.
## Process — strictly in this order
 
1. **Propose schemas first, as a standalone deliverable.** Include: the shared `pageBuilder` type and its list of blocks; each block type (fields, types, validation, preview, icon); each document type; singletons; and, for each existing page, the ordered block list it will be seeded with. Explicitly ask the user to review and adjust block boundaries, field names and types now — this is the cheapest point to fix mistakes. Call out any place you chose a reference over an inline object (or vice versa) and why.
2. **Wait for confirmation** (or proceed if the user says the schemas look fine / asks to just continue) before moving to step 3.
3. **Write GROQ queries.** For page-like documents, query by slug and project the `pageBuilder` with a per-type projection, always keeping `_key` and `_type`, and resolving references inside the relevant block:
   ```groq
   *[_type == "page" && slug.current == $slug][0]{
     title,
     seo,
     pageBuilder[]{
       _key,
       _type,
       _type == "heroBlock" => { heading, tagline, image },
       _type == "testimonialsBlock" => {
         heading,
         testimonials[]->{ _id, quote, name, role, photo }
       },
       _type == "ctaBlock" => { heading, link }
     }
   }
   ```
 
   Keep block projections as named fragments (one per block, next to the block's query file) and compose them into the page query, so adding a block means adding one fragment. Use `defineQuery` and TypeGen if the project uses them. Add a slugs query for `generateStaticParams`.
4. **Wire components to fetch real data.**
   - Create a single `PageBuilder` renderer that maps `_type` → block component, iterates the array using `_key` as the React key, passes `index` (for heading-level / first-block decisions), and renders nothing (with a dev-only console warning) for unknown types.
   - Refactor each existing static section component into the matching block component; keep its props close to the block schema shape to minimise transformation logic.
   - Pages become a thin route (e.g. `app/[slug]/page.tsx` plus the homepage route) that fetches the page and renders `<PageBuilder blocks={page.pageBuilder} />`. Keep static rendering (`generateStaticParams`, tag- or time-based revalidation per project convention).
   - If the project uses Visual Editing / Presentation, add the data attributes needed for click-to-edit and drag-to-reorder on blocks.
5. **Seed Studio with the real approved content** from the original HTML/component text, not lorem ipsum, so the client sees their actual content live in the CMS. For each page, create the `page` document with its `pageBuilder` blocks in the same order as the original static page, each with a unique `_key`; create referenced documents (testimonials, etc.) first and link them; upload the real images as assets with their alt text.
## What NOT to do
 
- Don't write GROQ queries or fetch logic before schemas are confirmed — schema changes after code is written are the most expensive mistake in this workflow.
- Don't model pages as a fixed list of section fields (`heroHeading`, `aboutText`, `servicesIntro`…) on the page document — use the page builder so editors can add, remove and reorder sections without a developer.
- Don't create a separate block type per page for what is visually the same section — one reusable block type.
- Don't put presentation settings (colors, alignment, spacing, layout variants) into blocks by default.
- Don't duplicate reusable content inside blocks when it should be a referenced document.
- Don't turn static UI chrome into CMS fields "just in case" — over-modeling makes the Studio harder to use for the client's actual editing needs.
- Don't use plain string/`url` fields for links — always the shared internal/external link type.
- Don't skip block previews and icons — an array of blocks all labelled "Object" is unusable for editors.
- Don't skip seeding real content — an empty or lorem-ipsum-filled Studio undersells the deliverable to the client.
