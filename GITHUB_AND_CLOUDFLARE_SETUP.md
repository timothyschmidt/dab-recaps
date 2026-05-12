# GitHub + Cloudflare Pages — One-time setup

This walks you through getting `dab_recaps_website/` published as a static site on Cloudflare Pages. **Do all steps from this folder** (`D:\DND\Death And Beyond\DAB - Project(Claude)\dab_recaps_website`).

You only need to do this once. After it's set up, publishing new recaps is just `git add . && git commit && git push`.

---

## Step 0 — Make sure git is installed

Open **PowerShell** and run:

```powershell
git --version
```

If you see a version number (e.g. `git version 2.43.0.windows.1`), you're good — skip to Step 1.

If you get "command not found", install **Git for Windows** from <https://git-scm.com/download/win>. Accept all defaults during installation. Then close and reopen PowerShell and re-check.

---

## Step 1 — Clean up the broken `.git` folder

I tried to initialize git from inside the cowork sandbox, but the Windows ↔ Linux mount corrupted the config file. There's a partially-created `.git` folder sitting in `dab_recaps_website\` that needs to go before we initialize properly.

In PowerShell, from inside `dab_recaps_website`:

```powershell
cd "D:\DND\Death And Beyond\DAB - Project(Claude)\dab_recaps_website"
Remove-Item -Recurse -Force .git
```

Verify it's gone:

```powershell
Test-Path .git
# should print:  False
```

> If `Remove-Item` complains about read-only files, run:
> `Get-ChildItem .git -Recurse -Force | ForEach-Object { $_.Attributes = 'Normal' }` then retry the `Remove-Item`.

---

## Step 2 — Initialize the repo

Still in PowerShell, in the same folder:

```powershell
git init -b main
git config user.name "Tim Schmidt"
git config user.email "timothy.adam.schmidt@gmail.com"
git config core.autocrlf false
git config core.eol lf
```

> `core.autocrlf false` + `core.eol lf` make sure Windows line endings don't sneak in. The `.gitattributes` in this folder also enforces this.

---

## Step 3 — First commit

```powershell
git add .
git status            # sanity check — should list index.html, the 3 recap HTMLs, README.md, .gitignore, .gitattributes, this file
git commit -m "Initial commit: DAB session recap site (s02 + s03)"
```

After this, `git log --oneline` should show your one commit.

---

## Step 4 — Create the GitHub repo

Open <https://github.com/new> in your browser and create a repo with:

| Field            | Value                                  |
| ---------------- | -------------------------------------- |
| Repository name  | `dab-recaps`                           |
| Description      | _(optional)_ `D&D campaign recaps for Death And Beyond` |
| Visibility       | **Public** _(required for free Cloudflare Pages on the easiest tier; private also works but the setup is slightly different)_ |
| Add README       | **Leave unchecked**                    |
| Add .gitignore   | **Leave unchecked**                    |
| Add license      | **Leave unchecked**                    |

> Why uncheck those three: you already have them locally, and pre-creating them on GitHub would force you to merge before you can push.

Click **Create repository**.

> ⚠️ **Privacy reminder before you click "public":** the three HTML files in this folder contain party-facing recap content. They mention NPC names, locations, and what your party has done. They do *not* contain DM-only material (that lives outside this folder, in `/game_state/` and `/campaign/`). But if any future recap HTML pulls in `[DM-only]`-tagged content by mistake, it would become public. Worth a once-over of new recaps before each push.

---

## Step 5 — Connect local repo to GitHub and push

GitHub's "create repo" page shows the exact commands to run. They will be:

```powershell
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/dab-recaps.git
git push -u origin main
```

Replace `YOUR_GITHUB_USERNAME` with your actual GitHub username.

**Authentication:** the first push will prompt you to authenticate. You have two easy options:

- **Browser sign-in (easiest):** when prompted, click "Sign in with your browser" — GitHub will open a tab, you authorize, and you're done. Git for Windows handles credentials from here on.
- **Personal Access Token:** create one at <https://github.com/settings/tokens?type=beta>, scope it to the `dab-recaps` repo with `Contents: Read and write`, paste it when prompted.

> Avoid: putting your password in directly. GitHub stopped supporting password auth for git years ago.

After the push, refresh your GitHub repo page — all files should be there.

---

## Step 6 — Hook up Cloudflare Pages

1. Go to <https://dash.cloudflare.com/> and sign in (create a free account if you don't have one).
2. In the left sidebar: **Workers & Pages** → **Create application** → **Pages** tab → **Connect to Git**.
3. Authorize Cloudflare to read your GitHub account (one-time). When asked which repos: pick **Only select repositories** → choose `dab-recaps`.
4. Select the `dab-recaps` repo and click **Begin setup**.
5. **Build settings — IMPORTANT:** this is a plain static site, no build needed. Set:

   | Field                  | Value     |
   | ---------------------- | --------- |
   | Production branch      | `main`    |
   | Framework preset       | `None`    |
   | Build command          | _(leave blank)_ |
   | Build output directory | `/`       |
   | Root directory         | _(leave blank)_ |

6. Click **Save and Deploy**.

Cloudflare will spin up a build (takes ~20-40 seconds for a static site) and give you a URL like:

```
https://dab-recaps.pages.dev
```

That's your live site. The landing page (`index.html`) loads automatically.

---

## Step 7 — Publishing new recaps

Once everything above is done, the loop for adding a new session recap is:

```powershell
cd "D:\DND\Death And Beyond\DAB - Project(Claude)\dab_recaps_website"

# 1. Put the new sNN-YYYY-MM-DD-*.html file in this folder
# 2. Edit index.html to add a card for it (see README.md for the pattern)

git add .
git commit -m "Add session NN recap"
git push
```

Cloudflare Pages auto-rebuilds within a minute of every push to `main`. No further action needed.

---

## (Optional) Custom domain

If you ever want it on a custom domain instead of `dab-recaps.pages.dev`:

1. In your Cloudflare Pages project → **Custom domains** → **Set up a custom domain**.
2. Enter the domain (you must own it and have its DNS managed by Cloudflare or be willing to add a CNAME).
3. Follow Cloudflare's DNS instructions. They'll provision an SSL cert automatically.

---

## Troubleshooting

**`git push` rejected with "non-fast-forward":** you probably created the GitHub repo *with* a README/license. Either delete and recreate the GitHub repo without those, or run `git pull origin main --rebase` first, then push.

**Cloudflare build "succeeded" but the page is blank:** check that `index.html` is at the repo root, not nested. Run `git ls-tree -r --name-only main` from PowerShell and confirm `index.html` is listed.

**Google Fonts not loading on the live site:** they load from `fonts.googleapis.com` via the `@import` in each HTML's `<style>`. If your browser blocks third-party fonts, the page falls back to system fonts — content is still readable. Nothing to fix on the server side.

**Need to make the repo private later:** Cloudflare Pages works with private GitHub repos too — the connection step is the same. The published *site* is always public (that's the point of Pages), but the source can be private.
