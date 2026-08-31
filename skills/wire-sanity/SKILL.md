---
name: wire-sanity
description: Turn static Next.js components into Sanity-driven content, starting with schema design (reviewed before any code) followed by GROQ queries and component wiring. Invoke explicitly after Next.js components are built, e.g. "use wire-sanity to hook this up".
---

# Wire Sanity

Connects already-built, statically-rendered Next.js components to Sanity CMS. This is schema-first: schemas are proposed and approved before any query or component code is touched.

## Inputs to gather before starting

- The static Next.js components/pages to wire up
- A content map, if one exists: which blocks are **static UI chrome** (stays hardcoded), **repeatable content** (becomes a document type — testimonials, services, team members), and **singleton content** (fields on a page/settings document — hero heading, about blurb, contact info)
- If no content map exists, generate one first by walking through the components and asking the user to confirm the static/repeatable/singleton split before proposing schemas
- Sanity project conventions already in use (existing schema file structure, naming, whether using Sanity Studio v3, GROQ vs GraphQL, any existing shared schema types like `seo` or `cta`)

## Process — strictly in this order

1. **Propose schemas first, as a standalone deliverable.** For each repeatable/singleton content type identified, draft the Sanity schema (field names, types, validation) and present it before writing any query or component code. Explicitly ask the user to review and adjust field names/types now — this is the cheapest point to fix mistakes.
2. **Wait for confirmation** (or proceed if the user says the schemas look fine / asks to just continue) before moving to step 3.
3. **Write GROQ queries** for each page/component, matching the approved schemas.
4. **Wire components to fetch real data** — replace hardcoded text/images with data from the query results. Keep prop shapes close to the schema shape to minimize transformation logic.
5. **Seed Studio with the real approved content** from the original HTML/component text, not lorem ipsum, so the client sees their actual content live in the CMS.

## What NOT to do

- Don't write GROQ queries or fetch logic before schemas are confirmed — schema changes after code is written are the most expensive mistake in this workflow.
- Don't turn static UI chrome into CMS fields "just in case" — over-modeling makes the Studio harder to use for the client's actual editing needs.
- Don't skip seeding real content — an empty or lorem-ipsum-filled Studio undersells the deliverable to the client.
