# Death And Beyond — Session Recaps

Static site for the **Death And Beyond** D&D 5e (2024) campaign. Hosts party-facing session recaps for the table.

Published via **Cloudflare Pages** from this repo (`dab-recaps`).

## Structure

```
.
├── index.html                              # Landing page — lists every recap
├── s02-2026-04-13-party-recap.html         # S02 party recap
├── s03-2026-04-29-party-recap.html         # S03 party recap (casual)
└── s03-2026-04-29-the-drowned-hand.html    # S03 technical summary
```

Each recap is a fully self-contained HTML file (inline CSS, no JS, fonts pulled from Google Fonts). They can be opened locally by double-clicking or served as-is.

## Adding a new recap

1. Drop the new `sNN-YYYY-MM-DD-<slug>.html` file into this folder.
2. Open `index.html` and add a new `<section class="session-group">` for the session, or a new `<a class="card">` inside an existing one. Match the styling of existing cards.
3. Commit and push:

   ```
   git add .
   git commit -m "Add session NN recap"
   git push
   ```

4. Cloudflare Pages will rebuild automatically and the site will be live within ~30 seconds.

## Conventions

- Filenames use the project-wide pattern: `sNN-YYYY-MM-DD-short-title.html`.
- Two recap types per session may exist: a **party recap** (casual, table-facing) and a **technical summary** (detailed, with rules/rolls). The technical summary cards use the cool-blue accent (`--frost` variable) instead of gold.
- All recaps share the same color palette so the index and pages feel cohesive.

## Local preview

This is a static site — just open `index.html` in a browser. No build step.

If you want to test relative links exactly as Cloudflare Pages will serve them, run a quick local server from this folder:

```
python -m http.server 8000
```

Then visit `http://localhost:8000`.
