---
name: sanity-next-page-builder
description: >-
  Sets up and extends a Sanity CMS + Next.js App Router marketing site with a
  page-builder (block array), shared internal/external link+CTA objects,
  Studio singletons (nav/footer), GROQ expansions, PageBuilder mapping, home
  vs [slug] routing, image URLs, and optional seed scripts. Use when adding
  Sanity to Next.js, creating page builder blocks, wiring schemas to React
  components, modeling links/CTAs, configuring singletons/structure, writing
  defineQuery GROQ, or seeding Sanity content. Complements sanity-best-practices
  for general Sanity guidance.
---

# Sanity + Next.js Page Builder

Reusable workflow for a content-driven marketing site: editors compose pages from blocks in Sanity; Next.js renders them.

For general Sanity APIs/schemas/GROQ/TypeGen, also load `sanity-best-practices` topic files as needed. This skill is the **project pattern** (wiring + conventions), not the Sanity platform encyclopedia.

## Target architecture

```text
repo/
  app/                         # Next.js App Router frontend
    src/
      sanity/                  # client, image helper, queries
      lib/links.ts             # resolveHref for link/cta objects
      components/
        PageBuilder.tsx        # _type → React block map
        blocks/                # one component per block type
        layout/                # header/footer chrome
      app/
        layout.tsx             # fetch nav/footer singletons
        page.tsx               # home page (slug "home")
        [slug]/page.tsx        # other pages + 404 / redirect home
  studio-*/                    # standalone Sanity Studio
    schemaTypes/
      index.ts
      pageType.ts
      pageBuilderType.ts       # array of block members
      objects/                 # link, cta, shared fields
      blocks/                  # one schema per block
    structure.ts               # singletons + lists
    sanity.config.ts
```

Prefer **standalone Studio** next to the app (not embedded) unless the repo already embeds Studio.

## Operating mode

1. **Schema first, then frontend.** Add/change block schema → register in `pageBuilder` + `schemaTypes` → extend GROQ → map in `PageBuilder` → React component.
2. **Reuse shared objects.** Never reinvent link/CTA fields per block; use `link` / `cta` (or shared `linkFields`).
3. **Presentational blocks.** Block React components take plain props (`href`, `src`, copy). Mapping/normalization lives in `PageBuilder` (or thin adapters), not inside every block.
4. **Preserve existing UI.** When wiring an existing design system, do not redesign components—only feed them CMS data.
5. Work **incrementally** when many blocks exist: one block end-to-end, verify in Studio + frontend, then the next—unless the user asks for a batch.

## Phase 1 — Bootstrap (new project)

1. Create Sanity project + dataset; note `projectId`.
2. Scaffold Studio (`sanity init` / CLI) with schema folder structure above.
3. In the Next app, install `next-sanity` and `@sanity/image-url`.
4. Env (app):
   - `NEXT_PUBLIC_SANITY_PROJECT_ID`
   - `NEXT_PUBLIC_SANITY_DATASET` (default `production`)
   - `SANITY_API_WRITE_TOKEN` only for seed/migrations (never expose to client)
5. Client (`createClient` from `next-sanity`): pin `apiVersion` (YYYY-MM-DD), `useCdn: false` when using short ISR/`revalidate` or needing fresher reads—match the project's caching choice.
6. Image helper: wrap `@sanity/image-url` with the client's `projectId`/`dataset`.

## Phase 2 — Core content model

### Page document

- Fields: `title` (string), `slug` (from title), `content` (`pageBuilder` array).
- Convention: reserved slug `home` for `/`. Other slugs map to `/[slug]`.

### Page builder

```ts
// pageBuilderType — array of defineArrayMember({ type: 'blockName' })
```

Every block is an **object** type with a clear `preview` (title + subtitle = block name + media when useful).

### Shared link / CTA (required pattern)

Editors choose **Internal page** vs **External URL**:

| Field | Role |
|-------|------|
| `linkType` | `'internal' \| 'external'` (radio) |
| `internalLink` | `reference` → `page` (hidden unless internal) |
| `href` | path, absolute URL, `tel:`, `mailto:` (hidden unless external) |
| `label` | only on `cta` (and labeled nav items) |

Put the three link fields in a shared `linkFields` array; spread into:

- `link` object — link only
- `cta` object — `label` + link fields
- inline objects (nav items) that need label + link

Validate: required field for the active `linkType` only.

### Reusable documents (optional)

Use document types + references when content is shared across pages (e.g. `faq`, `review`). Block fields hold `reference` / array of references; expand in GROQ.

### Singletons (nav / footer)

1. Document types e.g. `navigation`, `footer`.
2. Studio `structure`: open with **fixed** `documentId` matching the type name (`navigation`, `footer`).
3. Hide those types from the default document list (filter `documentTypeListItems`).
4. Optionally restrict creation/duplication via `sanity.config` document actions / new-document templates so only the singleton IDs exist.
5. Frontend queries by `_id == "navigation"` (not `_type` alone).

Let Sanity generate ordinary document `_id`s. Use explicit IDs **only** for singletons.

## Phase 3 — Studio structure checklist

```text
Studio structure:
- [ ] Singleton list items first (Navigation, Footer)
- [ ] Divider + Pages list
- [ ] Divider + reusable docs (FAQs, Reviews, …)
- [ ] Filter default list to avoid duplicate singleton entries
- [ ] Icons: import from `@sanity/icons/<Name>` (named path). Avoid deprecated/missing barrel exports.
```

## Phase 4 — GROQ + Next data layer

### Link expansion fragment

Reuse one GROQ snippet everywhere links appear:

```groq
{
  ...,
  internalLink->{
    "slug": slug.current
  }
}
```

Apply to `primaryCta`, `secondaryCta`, `link`, `cta`, nested `items[]`, `regions[]`, `places[]`, singleton nav/footer links, and referenced docs that embed CTAs.

### Page query

- `*[_type == "page" && slug.current == $slug][0]{ … content[]{ … } }`
- For reference arrays inside a block, use conditional projections, e.g. `_type == "faqs" => { …, faqs[]->{ … } }`.

Use `defineQuery` from `next-sanity` for queries.

### Resolve href in the app

```ts
// lib/links.ts
// internal + slug "home" → "/"
// internal + other slug → "/{slug}"
// external → href as stored
```

`PageBuilder` / layout map Sanity objects → `{ label, href }` before passing to UI.

### Fetch options

Match project caching. A common marketing default:

```ts
const options = { next: { revalidate: 30 } }
```

Document the choice; don't assume static generation unless configured.

## Phase 5 — Routing

| Route | Behavior |
|-------|----------|
| `/` | Fetch page with slug `home`; `notFound()` if missing |
| `/[slug]` | Fetch by slug; if slug is `home`, `permanentRedirect('/')`; else `notFound()` if missing |
| `not-found` | Dedicated 404 UI |

Layout loads nav/footer singletons and passes resolved links into header/footer.

## Phase 6 — Add a new block (repeatable)

Copy this checklist:

```text
New block:
- [ ] schemaTypes/blocks/{name}Type.ts (object + preview + icon)
- [ ] Registered in pageBuilderType `of: […]`
- [ ] Exported in schemaTypes/index.ts
- [ ] GROQ: expand any link/cta/reference fields on this block
- [ ] components/blocks/{Name}.tsx (or reuse existing UI section)
- [ ] Case in PageBuilder switch (_type → props via toCta / toImage / resolveHref)
- [ ] Studio: create/edit on a page, verify frontend
```

**Image fields:** always include `alt` (required) and `options: { hotspot: true }` when cropping matters. Build URLs in the mapper with width/height/`fit` as needed.

**Unknown `_type`:** log a warning and return `null`—don't crash the page.

## Phase 7 — Seed script (optional)

When the user wants content injected from local `lib/*` or `public/` assets:

1. Script under `app/scripts/` using `@sanity/client` + **write token**.
2. Idempotent: reuse assets by `originalFilename`, pages by slug, reviews/FAQs by stable key, singletons by fixed `_id`.
3. Generate `_key` for array items; set `_type` on objects (`cta`, `link`, block names).
4. Upload images from `public/`; don't invent remote URLs.
5. Document env + npm script in the script header.

Do **not** commit tokens. Do **not** run seed against production without explicit user OK.

## Hard constraints

- Do not create deterministic UUIDs for normal documents.
- Do not use plain string URL fields for CTAs when the shared internal/external pattern exists—extend `linkFields` instead.
- Do not put GROQ-resolved shape assumptions inside presentational components.
- Do not commit, deploy, or rotate Sanity tokens unless asked.
- Prefer existing block/UI components over new card layouts when wiring a designed site.
- When both this skill and `html-to-next-tailwind` apply: build/migrate UI with the HTML skill; make it CMS-driven with **this** skill.

## Pairing with html-to-next-tailwind

Typical sequence for a static design → CMS site:

1. HTML → Next + Tailwind sections (`html-to-next-tailwind`)
2. Extract copy into `lib/*` if helpful
3. Model blocks + seed or manual Studio entry (this skill)
4. Replace hardcoded page composition with `<PageBuilder content={page.content} />`

## Additional resources

- Field/GROQ templates: [reference.md](reference.md)
