# ID Vision Docs

Documentation-only repository. No application source code lives here — the
ID Vision Worker and Hub services are developed elsewhere; this repo only
documents how to install, configure, and operate them.

Published to GitHub Pages: https://qasemt.github.io/id-vision-docs/

## Stack

- Jekyll 4.3 with `jekyll-remote-theme`
- Theme: `jekyll/minima@1e8a445`, dark skin
- `baseurl: "/id-vision-docs"` — the site is served from a subpath
- Deployed by `.github/workflows/jekyll-gh-pages.yml` on every push to `main`

## Local preview

```bash
bundle install          # first time only
bundle exec jekyll serve
# http://localhost:4000/id-vision-docs/
```

Note the `/id-vision-docs/` suffix — omitting it gives a 404 because of
`baseurl`. Use `--livereload` while editing.

## Conventions

- **Content pages are Markdown in the repo root**, except the component setup
  guides under `worker/` and `hub/`. Images go in `assets/`.
- **Internal links are relative and keep the `.md` extension**
  (`[Camera Setup](camera-setup.md)`) so they resolve both on GitHub and on the
  built site. Never hardcode `/id-vision-docs/` into a link.
- **Every new page must be added to the document table in `index.md`.**
  Nothing else links pages together.
- `_sass/minima/custom-styles.scss` hides the theme's auto-generated page title
  and post header. This means the `# H1` inside each Markdown file is the only
  visible title — every page needs one, and it should match its `index.md` link
  text.
- `_sass/minima/custom-variables.scss` overrides the theme's default variables
  (currently `$content-width`, widened from minima's default 800px so tables
  with several columns of text don't get squeezed). It's loaded before the
  theme sets its `!default` values, so a plain assignment there wins.
- Layout overrides live in `_includes/default.html` and `_includes/header.html`,
  overriding the remote theme's versions. Editing them changes every page.

## Writing documentation

Follow the `idvision-docs` skill in `.claude/skills/` for page structure, tone,
formatting, and the pre-publish checklist.

Two rules override anything else:

1. **Never invent versions, hardware specs, ports, paths, or download URLs.**
   These are load-bearing for integrators deploying on customer sites. If a
   value is unknown, insert `<!-- TODO: confirm -->` and say what is missing.
2. **Do not silently "fix" inconsistencies between pages** (for example the OS
   support mismatch between `system_requirements.md` and
   `installation-guide.md`). Flag them and ask — only the author knows which
   version is correct.
3. **Never name the underlying face-detection library (e.g. "InsightFace") in
   any document.** Refer to it generically — "the face detector", "the
   detection engine" — regardless of what the source code or its comments
   call it. This applies to all pages, present and future.

## Git

Default branch is `main`; pushing to it deploys immediately. Preview locally
before pushing.

Commit `.claude/skills/` so the team shares the same authoring rules. Ignore
personal Claude config (`.claude/settings.local.json`) and Jekyll build output
(`_site/`, `.jekyll-cache/`).