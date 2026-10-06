# Project instructions

Personal bilingual blog on Hugo with the Blowfish theme.
EN is served from `/`, RU from `/ru/`.

## Hugo version

- Use Hugo extended 0.166.0, the same version as `.github/workflows/deploy.yml`.
- The version MUST stay within `[module.hugoVersion]` in `themes/blowfish/config.toml`.
- Locally it lives at `~/.local/bin/hugo`. Check with `hugo version` before building.

## Checks

- Build: `hugo --minify`. It MUST finish without errors.

## Run

- Dev server with drafts: `hugo server -D`, then open http://localhost:1313/ and http://localhost:1313/ru/.

## Theme

- Blowfish is a git submodule in `themes/blowfish`, pinned to a release tag.
- MUST NOT edit files inside `themes/blowfish`. Put overrides in the project `layouts/`, `assets/`, or `i18n/`.
- Site config lives in `config/_default/`. There is no root `hugo.toml`.

## Content

- Posts are plain Markdown. MUST NOT use theme shortcodes, so the theme can be replaced.
- CV data lives in `data/cv.yaml` and is rendered by a project template.
- Content per language: `content/en/`, `content/ru/`. Translations share a `translationKey`.
- MUST NOT replace or soften profanity or slang in post text.
