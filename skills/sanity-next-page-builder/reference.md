# Sanity + Next page builder — templates

Copy/adapt these; rename types to match the project.

## linkFields

```ts
import {defineField} from 'sanity'

export const linkFields = [
  defineField({
    name: 'linkType',
    title: 'Link type',
    type: 'string',
    options: {
      list: [
        {title: 'Internal page', value: 'internal'},
        {title: 'External URL', value: 'external'},
      ],
      layout: 'radio',
      direction: 'horizontal',
    },
    initialValue: 'external',
    validation: (rule) => rule.required(),
  }),
  defineField({
    name: 'internalLink',
    title: 'Page',
    type: 'reference',
    to: [{type: 'page'}],
    hidden: ({parent}) => parent?.linkType !== 'internal',
    validation: (rule) =>
      rule.custom((value, context) => {
        const parent = context.parent as {linkType?: string} | undefined
        if (parent?.linkType === 'internal' && !value) return 'Select a page'
        return true
      }),
  }),
  defineField({
    name: 'href',
    title: 'URL',
    type: 'string',
    description: 'Path (/about), full URL, tel: or mailto:',
    hidden: ({parent}) => parent?.linkType !== 'external',
    validation: (rule) =>
      rule.custom((value, context) => {
        const parent = context.parent as {linkType?: string} | undefined
        if (parent?.linkType === 'external' && !value?.trim()) return 'Enter a URL'
        return true
      }),
  }),
]
```

## cta + link objects

```ts
// cta: label + ...linkFields
// link: ...linkFields only
```

## resolveHref

```ts
export function resolveHref(link: {
  linkType?: 'internal' | 'external' | null
  href?: string | null
  internalLink?: {slug?: string | null} | null
} | null | undefined): string | undefined {
  if (!link) return undefined
  if (link.linkType === 'internal') {
    const slug = link.internalLink?.slug
    if (!slug) return undefined
    return slug === 'home' ? '/' : `/${slug}`
  }
  return link.href || undefined
}
```

## GROQ link expansion

```ts
const linkExpansion = /* groq */ `{
  ...,
  internalLink->{
    "slug": slug.current
  }
}`
```

Use as `` primaryCta${linkExpansion} `` etc. inside `defineQuery`.

## Singleton structure item

```ts
S.listItem()
  .title('Navigation')
  .id('navigation')
  .child(
    S.document()
      .schemaType('navigation')
      .documentId('navigation')
      .title('Navigation'),
  )
```

Query: `*[_id == "navigation"][0]{ … }`.

## Minimal PageBuilder case

```tsx
case 'hero':
  return (
    <Hero
      key={block._key}
      title={block.title as string | undefined}
      primaryCta={toCta(block.primaryCta as SanityCta)}
      // …
    />
  )
```

`toCta` = label + `resolveHref`. Images via `urlFor(source)?.width(w).height(h).fit('crop').url()`.

## Client

```ts
import {createClient} from 'next-sanity'

export const client = createClient({
  projectId: process.env.NEXT_PUBLIC_SANITY_PROJECT_ID,
  dataset: process.env.NEXT_PUBLIC_SANITY_DATASET || 'production',
  apiVersion: '2026-07-26', // pin a date
  useCdn: false,
})
```

## Seed helpers

```ts
function cta(label: string, href: string) {
  return {_type: 'cta' as const, label, linkType: 'external' as const, href}
}

function externalLink(href: string) {
  return {_type: 'link' as const, linkType: 'external' as const, href}
}

// pages: createOrReplace by slug lookup
// singletons: createOrReplace({ _id: 'navigation', _type: 'navigation', … })
```
