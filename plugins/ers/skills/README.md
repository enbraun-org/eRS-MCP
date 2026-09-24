# Skills (template)

Canonical skills live here. Cursor loads via root `.cursor-plugin/plugin.json` → `"skills": "./plugins/ers/skills"`. Claude loads via root `.claude-plugin/marketplace.json` → `./plugins/ers`.

## On `main`

Only `example-skill` (copy-me stub). Add real skills on a feature branch, then PR into `main`.

| Folder | Role |
| --- | --- |
| `example-skill/` | Authoring template — do not ship as a product skill |

## How to add a skill

1. Copy `example-skill/` → `<skill-name>/`.
2. Edit `SKILL.md` frontmatter and body.
3. Update this table.
4. Open a PR into `main`.
