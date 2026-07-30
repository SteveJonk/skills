# Skills

A collection of reusable "skills" — self-contained instructions and workflows that extend what an AI coding agent can do. Written to be LLM-agnostic: usable in Cursor, Claude, or any other agent that supports loading Markdown instructions on demand.

## What's a Skill?

A Skill is a folder containing a `SKILL.md` file with YAML frontmatter (`name` + `description`) followed by instructions, best practices, and any supporting scripts or references the agent should use when the skill is triggered.


## Available Skills

| Skill | Description |
|---|---|
| [`html-to-next-tailwind`](./skills/html-to-next-tailwind/SKILL.md) | Converts static HTML/CSS/JS page designs into a Next.js App Router site with Tailwind CSS v4, then incrementally migrates off legacy stylesheets until tokens live only in the Tailwind config. Use when converting example HTML, design prototypes, or static marketing pages into Next.js components; when migrating `site.css` / custom CSS to Tailwind utilities; or when cleaning up duplicated `@theme` and `globals.css` after a Tailwind migration. |
| [`sanity-next-page-builder`](./skills/sanity-next-page-builder/SKILL.md) | Connects a NextJS site to sanity, setting up the schema, hooking up the blockrenderer, seeding the cms |


## Usage

- **Cursor:** copy the skills into your project (or a shared rules/skills directory) and reference it in your rules so the agent can pull it in when relevant.
- **Other agents:** any tool that can load a Markdown file as context or instructions can use these the same way — point it at the relevant `SKILL.md`.


## License

MIT (or update as you prefer).
