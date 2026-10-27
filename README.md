# Agentic Automation Workshop — Site Guide

How the site works, how to edit it, and what to ask Claude Code to do.

**Live site:** <https://uipath-practice.github.io/AgenticProfessionalCourse/>

---

## How It Works

```
docs/*.md files
      ↓
  MkDocs builds HTML
      ↓
  GitHub Actions pushes to gh-pages branch
      ↓
  GitHub Pages serves the live site
```

| File / Folder | Purpose |
|---------------|---------|
| `docs/` | All page content — one `.md` file per page |
| `mkdocs.yml` | Site config: theme, navigation, plugins |
| `docs/stylesheets/extra.css` | Custom CSS (image styles, two-column layouts) |
| `docs/*/<step>.images/` | Screenshots per lesson |
| `docs/assets/images/` | Images shared across multiple pages |
| `hooks/split_cols.py` | The `[[[ … \|N\| … ]]]` two-column shorthand |
| `scripts/` | Screenshot→metadata pipeline (Azure OpenAI vision) and translation status |
| `source/` | Existing prose for a migrated course — authoritative text for `/new-lesson` |
| `.github/workflows/deploy.yml` | Auto-deploy on every push to `main` |
| `Master/` | Authoritative course rules, templates, and formatting reference |
| `Master/CourseProfile.md` | **Every course-specific value.** Start here on a new course |
| `CLAUDE.md` | Compact authoring rules Claude reads automatically; points to `Master/` |
| `Archive/` | Archived exercises and lessons — gitignored, never deployed |
| `staging/` | Content-migration scratch — gitignored, never deployed |

**Publishing is automatic.** Push to `main` → GitHub Actions runs → site updates in ~60 seconds.
You never run a deploy command manually.

---

## Setup

```bash
pip3 install --user -r requirements.txt        # MkDocs + theme
pip3 install --user -r scripts/requirements.txt # only if using /new-lesson
```

For `/new-lesson`, copy `scripts/.env.example` to `scripts/.env` and fill in your Azure OpenAI
values. That file is gitignored.

`mkdocs.local.yml` is gitignored and does not arrive with a clone — create it once per machine with
`cp mkdocs.yml mkdocs.local.yml`. It is a standalone config, not an overlay, and it is where
`/new-exercise` registers drafts so you can preview them with full navigation.

---

## Preview Locally

```bash
python3 -m mkdocs serve
```

Opens at `http://127.0.0.1:8000<base-path>/` — the subpath comes from `site_url` in `mkdocs.yml`, so
a bare `http://127.0.0.1:8000/` will redirect. The site reloads live as you save.

`python3 -m mkdocs` rather than a bare `mkdocs` command: it works regardless of how MkDocs was
installed and does not depend on your PATH.

To check for broken links before pushing:

```bash
python3 -m mkdocs build --strict
```

Note: pages absent from `nav:` (drafts) log at INFO and do **not** fail `--strict`. A link to a page
that does not exist yet **does**.

---

## Making Changes Manually

Open the relevant `.md` file, edit, save, push. Navigation order and labels live in `mkdocs.yml`
under `nav:`.

### Quick formatting reference

Full details with examples: `Master/Formatting.md`. The essentials:

**Add a screenshot:**
```markdown
![Description of what is shown](step-slug.images/filename.png){ .screenshot }
```

**Resize** (wide images use `width="900"`, others `width="700"` or similar):
```markdown
![Description](step-slug.images/filename.png){ .screenshot width="700" }
```

**Two-column layout** (text next to screenshot):
```
[[[
Click on **Data Manager** and add a new argument.
|30|
![Data Manager panel](step-slug.images/4-data-manager.png){ .screenshot }
]]]
```

Supported ratios: `|30|`, `|50|`, `|70|`. Each delimiter on its own line.

**Callout box** — four types only (`tip`, `info`, `note`, `warning`):
```markdown
!!! tip "Title"
    Shortcut or helpful hint.
```

**Image inside a numbered list** (4-space indent preserves numbering):
```markdown
1. Open the designer and select **New**.

    ![New button](step-slug.images/1-new.png){ .screenshot }

2. Enter a name and click **Create**.
```

---

## What to Ask Claude Code

| Skill | What it does | When to use |
|-------|-------------|-------------|
| `/new-exercise` | Scaffolds an exercise: folders, stubs — **draft, not in nav** | Starting a new exercise |
| `/new-lesson` | Builds a lesson page from screenshots, and from `source/` prose when it exists | After adding screenshots to a lesson's images folder |
| `/publish-exercise` | Adds a draft exercise to navigation and the home page | When it's ready for learners |
| `/review-lesson` | Reviews one lesson against `Master/` rules (read-only) | After editing a lesson |
| `/review-exercise` | Reviews a whole exercise for per-page and cross-lesson coherence | After all lessons are reviewed |
| `/remove-lesson` | Archives a lesson, removes it from nav — asks for confirmation | Retiring a lesson |
| `/remove-exercise` | Archives an exercise, removes it from nav — asks for confirmation | Retiring an exercise |
| `/sync-translations` | Translates stale/missing pages (no-op while English-only) | After adding a locale |

### Draft and publish model

```
/new-exercise      →  creates files, not visible in nav
/new-lesson        →  adds lessons, still not visible in nav
/publish-exercise  →  adds nav entry and home page card — learners can see it
```

"Draft" means exactly one thing: **absent from `nav:` in `mkdocs.yml`.** Draft pages still build and
deploy, and remain reachable by direct URL — they are simply invisible in the navigation.

### Typical workflow

```
1. /new-exercise      →  name, description, step list (draft mode)
2. Add screenshots to docs/<exercise-slug>/<N-step-slug>.images/
3. Add reference links to docs/<exercise-slug>/documentation.txt
4. /new-lesson        →  exercise slug, step number, images folder
5. Review the generated page, edit as needed
6. /review-lesson     →  check against Master/ rules
7. /review-exercise   →  cross-lesson coherence
8. /publish-exercise  →  add to nav and home page
9. python3 -m mkdocs build --strict
10. git push          →  deploys automatically
```

### Migrating a course that already has written content

Put the existing prose in `source/<exercise-slug>/<N-lesson-slug>.md`. `/new-lesson` treats it as
authoritative and reshapes it into the lesson template rather than rewriting it, using the vision
pipeline only to verify strings and write alt text. Name harvested screenshots
`N-descriptive-name[-W].png` up front so the rename step can be skipped.

---

## Starting Another Course From This Toolchain

`Master/HOWTO.md` → "Workflow 9: Start a new course in a new repository" is the full checklist.
Fill in `Master/CourseProfile.md` first, then run its propagation checklist.

---

## Editing Tips

- Keep `python3 -m mkdocs serve` running — live reload beats any static preview
- For formatting hassles (two-column layouts, admonitions, resizing), ask Claude Code
- Write content in your editor; let Claude handle the markup
