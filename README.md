# Skills

A collection of reusable "skills" — self-contained instructions and workflows that extend what an AI coding agent can do. Written to be LLM-agnostic: usable in Cursor, Claude, or any other agent that supports loading Markdown instructions on demand.

## What's a Skill?

A Skill is a folder containing a `SKILL.md` file with YAML frontmatter (`name` + `description`) followed by instructions, best practices, and any supporting scripts or references the agent should use when the skill is triggered.

## Repo Structure

```
skills/
  html-to-next-tailwind/
    SKILL.md
  <next-skill>/
    SKILL.md
```

Each skill lives in its own folder named after the skill, containing at minimum a `SKILL.md`.

## Available Skills

| Skill | Description |
|---|---|
| [`html-to-next-tailwind`](./skills/html-to-next-tailwind/SKILL.md) | Converts static HTML/CSS/JS page designs into a Next.js App Router site with Tailwind CSS v4, then incrementally migrates off legacy stylesheets until tokens live only in the Tailwind config. Use when converting example HTML, design prototypes, or static marketing pages into Next.js components; when migrating `site.css` / custom CSS to Tailwind utilities; or when cleaning up duplicated `@theme` and `globals.css` after a Tailwind migration. |

## Usage

- **Cursor:** copy the skill folder into your project (or a shared rules/skills directory) and reference it in your rules so the agent can pull it in when relevant.
- **Other agents:** any tool that can load a Markdown file as context or instructions can use these the same way — point it at the relevant `SKILL.md`.

## Adding a New Skill

1. Create a new folder under `skills/` named after your skill.
2. Add a `SKILL.md` with `name` and `description` frontmatter, followed by the instructions.
3. Add a row to the table above.

## License

MIT (or update as you prefer).
