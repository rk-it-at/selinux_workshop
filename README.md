# SELinux Workshop Slides

Marp-based slide deck for an 8-hour SELinux workshop with hands-on labs for RHEL 9/10.

## Contents

- `slides.md` - main workshop deck (Marp markdown)
- `theme/rk-it.css` - custom Marp theme
- `assets/` - images and graphics used by slides
- `dist/` - generated HTML/PDF output

## Requirements

- Node.js and npm
- `@marp-team/marp-cli` (installed via `npm install`)

Optional:
- `codespell` for spell checking

## Install dependencies

```bash
npm install
```

## Build slides

Build HTML and PDF:

```bash
npm run build
```

Build HTML only:

```bash
npm run build:html
```

Build PDF only:

```bash
npm run build:pdf
```

Output files:

- `dist/index.html`
- `dist/slides.pdf`

## Presenter notes

Speaker notes are embedded in `slides.md` using Marp note blocks:

```md
<!-- _notes:
Speaker notes go here
-->
```

## Styling and layout notes

- Most slides are top-aligned via the theme
- Title, chapter and exercise title slides are centered using Marp slide classes:
  - `title-slide`
  - `chapter-slide`
  - `exercise-slide`
- Break slides hide footer and page numbers (`break-slide`)

## Spell checking

A project-specific config is included in `.codespellrc`.

Run from the project directory:

```bash
codespell
```

Fix interactively:

```bash
codespell -i 3 -w slides.md
```

## Editing workflow (recommended)

1. Edit `slides.md`
2. Rebuild HTML (`npm run build:html`) for quick preview
3. Rebuild PDF (`npm run build:pdf`) to verify final layout
4. Run `codespell`

## License

See `LICENSE`.
