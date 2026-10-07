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

- CV data lives in `data/cv.yaml` and is rendered by a project template.
- Content per language: `content/en/`, `content/ru/`. Translations share a `translationKey`.
- MUST NOT replace or soften profanity or slang in post text.
- Every image (PNG, JPEG, WebP) added to the repo MUST be compressed with the TinyPNG API before commit. The key is read from the `TINIFY_API_KEY` environment variable.
- Image file names MUST be descriptive Latin kebab-case, 2–4 words (e.g. `xinput-list-touchpad.png`), not `1.png` or `image0.png`.
- Every image MUST have alt text in the post's language that describes what the image shows.
