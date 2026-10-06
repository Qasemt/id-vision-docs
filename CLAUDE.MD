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

# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.
