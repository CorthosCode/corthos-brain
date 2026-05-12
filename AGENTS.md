# corthos-brain

Personal Obsidian Zettelkasten — Java/Spring backend engineering. All notes in **Russian**.

## File naming

Leading emoji indicates topic:

| Emoji | Topic |
|-------|-------|
| ☕ | Java / Kotlin |
| 📘 | Data Base / Math |
| 📗 | Testing |
| 📒 | Hibernate |
| 🔐 | Security |
| 🍃 | Spring Framework |
| 🔨 | Architecture / tools |
| ♻️ | Microservices |
| 🧰 | Git |
| 📑 | GraphQL |
| 🤖 | AI |
| 💻 | Terminal |
| 📖 | Problems |

Numbered series: Testing (`Глава 1..13`), Security (`1..11`), Data Base (`1..16`), Hibernate (`Глава 1..6`).

## Structure

Topic directories with subfolders `General/`, `Optional/`, `Other/` for larger topics.

## Style

- **YAML frontmatter**: minimal / optional — only `tags:` and `aliases:` when present
- **Internal links**: `[[wikilinks]]` (Obsidian `alwaysUpdateLinks: true`)
- **Images**: `assets/`, markdown `![](path)` syntax (not `![[...]]`)
- **No** Dataview queries or Templater

## Git

Manual commits (`autoSaveInterval: 0`). Message format: `Обновление от: YYYY-MM-DD HH:mm:ss`  
Remote: `https://github.com/CorthosCode/corthos-brain`

## Zotero

Plugin configured but currently unused. Template: `_template/ZOTERO.md`. Output: `_zotero/`, attachments: `_attachments/`. Citation: `[[{{citekey}}]]`.

## What's absent

README · Makefile · scripts · package.json for vault · `.cursor*` · `CLAUDE.md`
