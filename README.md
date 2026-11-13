# Skills

A collection of reusable "skills" — self-contained instructions and workflows that extend what an AI coding agent can do. Written to be LLM-agnostic: usable in Cursor, Claude, or any other agent that supports loading Markdown instructions on demand.

## What's a Skill?

A Skill is a folder containing a `SKILL.md` file with YAML frontmatter (`name` + `description`) followed by instructions, best practices, and any supporting scripts or references the agent should use when the skill is triggered.

## Available Skills

| Skill | Description |
|---|---|
| [`site-prototype`](./skills/site-prototype/SKILL.md) | Builds a standalone Tailwind HTML prototype for a client site, including a proposed design system, for client sign-off before any Next.js/Sanity work begins. |
| [`tailwind-html-to-nextjs`](./skills/tailwind-html-to-nextjs/SKILL.md) | Converts an approved static HTML/Tailwind prototype into componentized, statically-rendered Next.js pages matching an existing project's conventions — no CMS wiring yet. |
| [`wire-sanity`](./skills/wire-sanity/SKILL.md) | Turns static Next.js components into Sanity-driven content, schema-first: schema proposed and approved before any query or component code is touched. |


## Usage

- **Cursor:** copy the skills into your project (or a shared rules/skills directory) and reference it in your rules so the agent can pull it in when relevant.
- **Claude Code:** the `skill files/` folder has packaged `.skill` bundles ready to install; or point Claude at a `SKILL.md` directly.
- **Other agents:** any tool that can load a Markdown file as context or instructions can use these the same way — point it at the relevant `SKILL.md`.

## License

MIT (or update as you prefer).
