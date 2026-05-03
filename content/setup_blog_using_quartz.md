---
lastSync: Sat May 03 2025 15:05:01 GMT+0530 (India Standard Time)
title: Setup Blog using Quartz
draft: false
---

[Quartz](https://quartz.jzhao.xyz/) is a fast, batteries-included static site generator that turns a folder of Markdown files into a fully-featured digital garden or personal blog. It is built on top of Hugo and is designed to work seamlessly with Obsidian vaults. This guide walks through setting up a Quartz blog and deploying it to GitHub Pages.

### Prerequisites

Before starting, make sure you have the following installed:

- **Node.js** v18.14+ and **npm** v9.3.1+
- **Git**
- A **GitHub account** with a repository named `<your-username>.github.io`

Verify your versions:

```bash
node -v
npm -v
git --version
```

### Initial Setup

Clone the Quartz repository and install dependencies:

```bash
git clone https://github.com/jackyzha0/quartz.git <your-repo-name>
cd <your-repo-name>
npm install
npx quartz create
```

The `npx quartz create` command will prompt you to choose:
- **Empty Quartz** — start from scratch
- **Copy your Obsidian vault** — link an existing vault directory

Your content lives in the `content/` directory. The entry point is `content/index.md`, which becomes the home page.

### Configure Quartz

Edit `quartz.config.ts` to set your site metadata:

```ts
const config: QuartzConfig = {
  configuration: {
    pageTitle: "Your Blog Title",
    pageTitleSuffix: "",
    enableSPA: true,
    enablePopovers: true,
    analytics: null,
    locale: "en-US",
    baseUrl: "<your-username>.github.io",
    ignorePatterns: ["private", "templates", ".obsidian"],
    defaultDateType: "modified",
    theme: {
      fontOrigin: "googleFonts",
      cdnCaching: true,
      typography: {
        header: "Schibsted Grotesk",
        body: "Source Sans Pro",
        code: "IBM Plex Mono",
      },
      colors: {
        lightMode: { /* ... */ },
        darkMode:  { /* ... */ },
      },
    },
  },
  plugins: { /* ... */ },
}
```

The `baseUrl` field is important — it must match your GitHub Pages domain exactly, without the `https://` prefix.

### Preview Locally

Before pushing, preview the site locally:

```bash
npx quartz build --serve
```

This starts a dev server at `http://localhost:8080` with hot reload. Any changes to files in `content/` are reflected immediately.

### Update Deployment Script

To deploy to GitHub Pages, you need a GitHub Actions workflow. Quartz ships with a default `deploy.yml` but it sometimes uses outdated action versions. Replace `.github/workflows/deploy.yml` with:

```yaml
name: Deploy Quartz site to GitHub Pages

on:
  push:
    branches:
      - v4

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - name: Install Dependencies
        run: npm ci
      - name: Build Quartz
        run: npx quartz build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public

  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

Key things to note:
- The `fetch-depth: 0` flag ensures git history is available, which Quartz uses to populate `lastmod` dates on posts.
- The branch trigger (`v4`) should match whatever branch you push your content to.
- Make sure GitHub Pages source is set to **GitHub Actions** in your repo settings under *Settings → Pages → Source*.

For more details refer to the [Custom GitHub Actions workflow](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow) documentation.

### Pushing Changes

After updating the deployment script, push changes using:

```bash
npx quartz sync
```

This command:
1. Pulls any remote changes
2. Adds and commits your local changes with a timestamped message
3. Pushes to the remote repository

It is essentially a convenience wrapper around the standard git workflow. Under the hood it runs:

```bash
git pull
git add .
git commit -m "sync: <timestamp>"
git push
```

You can also use plain `git` commands if you prefer more control over commit messages.

### Writing Content

Each Markdown file in `content/` becomes a page. Quartz supports frontmatter for metadata:

```md
---
title: My Post Title
date: 2025-04-01
tags:
  - machine-learning
  - notes
draft: false
---

Your content here...
```

- `draft: true` hides the page from the published site but keeps it visible during local preview.
- Tags automatically generate tag index pages.
- Wikilinks (`[[Page Name]]`) work out of the box if you are coming from Obsidian.

### Syncing a Newer Quartz Version

Quartz releases updates regularly with bug fixes and new features. To update:

```bash
npx quartz update
```

This fetches the latest changes from the upstream Quartz repository and merges them into your local branch. You may encounter merge conflicts if your `quartz.config.ts`, `quartz.layout.ts`, or any files inside `quartz/` have been modified locally.

To resolve conflicts:

```bash
git status                  # see conflicting files
# edit files to resolve conflicts
git add <resolved-files>
git commit
```

> After an update, Quartz sometimes syncs additional workflow files into `.github/workflows/`. I keep only `deploy.yml` and delete the rest to avoid redundant or conflicting CI runs.

### Verify Deployment

Once pushed, head to the **Actions** tab of your GitHub repository. You should see a workflow run triggered by your push. The workflow has two jobs:

1. **build** — installs dependencies and runs `npx quartz build`, outputting static files to `public/`
2. **deploy** — uploads the `public/` directory to GitHub Pages

Both jobs should show a green checkmark. If either fails, click into the job to read the logs. Common failures:
- **Node version mismatch** — update the `node-version` in the workflow to match your local version
- **Missing `baseUrl`** — ensure `quartz.config.ts` has the correct `baseUrl` set
- **Pages not enabled** — check that GitHub Pages is enabled and the source is set to GitHub Actions in repo settings

Once deployed, your site will be live at `https://<your-username>.github.io`.
