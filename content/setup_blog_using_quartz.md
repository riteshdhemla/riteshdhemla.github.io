---
lastSync: Sat May 03 2025 15:05:01 GMT+0530 (India Standard Time)
title: Setup Blog using Quartz
draft: false
---

[Quartz](https://quartz.jzhao.xyz/) is a fast, batteries-included static site generator that transforms Markdown files into a fully-featured website. It is used by thousands of people for publishing personal notes, digital gardens, and blogs. Out of the box it ships with full-text search, graph view, wikilinks, backlinks, LaTeX support, syntax highlighting, popover previews, and Obsidian compatibility — no plugins to install separately.

### Prerequisites

Before starting, make sure you have the following installed:

- **Node.js** v22 or higher
- **npm** v10.9.2 or higher
- **Git**

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
npm i
npx quartz create
```

The `npx quartz create` command will prompt you to choose a content initialisation strategy:
- **Empty Quartz** — start from scratch with a blank `content/` folder
- **Copy your Obsidian vault** — point Quartz at an existing Obsidian vault directory

Your content lives in the `content/` directory. The entry point is `content/index.md`, which becomes your home page.

### Configure Quartz

Edit `quartz.config.ts` to set your site metadata. The two top-level keys are `configuration` and `plugins`.

```ts
const config: QuartzConfig = {
  configuration: {
    pageTitle: "Your Blog Title",
    pageTitleSuffix: "",
    enableSPA: true,       // single-page app routing for fast navigation
    enablePopovers: true,  // hover previews for internal links
    analytics: null,       // supports GA, Plausible, Umami, GoatCounter, etc.
    locale: "en-US",
    baseUrl: "<your-username>.github.io",
    ignorePatterns: ["private", "templates", ".obsidian"],
    defaultDateType: "modified", // "created" | "modified" | "published"
    theme: {
      fontOrigin: "googleFonts",
      cdnCaching: true,
      typography: {
        header: "Schibsted Grotesk",
        body: "Source Sans Pro",
        code: "IBM Plex Mono",
      },
      colors: {
        lightMode: { /* light palette */ },
        darkMode:  { /* dark palette  */ },
      },
    },
  },
  plugins: {
    transformers: [ /* parse and mutate content */ ],
    filters:      [ /* remove unwanted pages    */ ],
    emitters:     [ /* output RSS, tag pages… */ ],
  },
}
```

The `baseUrl` field is required for sitemaps and RSS feeds — set it to your deployed domain without `https://` or trailing slashes.

### Preview Locally

Before pushing, preview the site locally:

```bash
npx quartz build --serve
```

This starts a dev server at `http://localhost:8080` with hot reload. Changes to files in `content/` and `quartz.config.ts` are reflected immediately without a full rebuild.

### Update Deployment Script

To deploy to GitHub Pages you need a GitHub Actions workflow. Create `.github/workflows/deploy.yml`:

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
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0        # full history needed for accurate lastmod dates
      - name: Setup Pages
        uses: actions/configure-pages@v4
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
          name: github-pages

  deploy:
    needs: build
    permissions:
      id-token: write
      pages: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

Then go to your repository **Settings → Pages → Source** and select **GitHub Actions**.

A few things to keep in mind:
- `fetch-depth: 0` ensures the full git history is available — Quartz uses this to populate `lastmod` dates on each post.
- The branch trigger (`v4`) must match the branch you push your content to.
- If the deploy job fails with an environment protection error, go to **Settings → Environments**, delete the existing `github-pages` environment, and re-run the workflow — it will recreate the environment correctly.

For more details refer to the [GitHub Pages custom workflow](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow) documentation.

### Pushing Changes

After writing content or updating config, push using:

```bash
npx quartz sync
```

This is a convenience wrapper that:
1. Pulls any remote changes
2. Stages and commits everything with a timestamped message
3. Pushes to the remote repository

You can also use plain git commands if you want control over commit messages:

```bash
git add content/my-new-post.md
git commit -m "add post on X"
git push
```

### Writing Content

Each Markdown file in `content/` becomes a page. Quartz supports YAML frontmatter for metadata:

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

- `draft: true` hides the page from the published site but keeps it visible during `--serve` (local preview).
- Tags automatically generate tag index pages at `/tags/<tag-name>`.
- Wikilinks (`[[Page Name]]`) and transclusions (`![[Page Name]]`) work out of the box — no plugin needed.

> **Trailing slash caveat**: Quartz generates files as `file.html` rather than `file/index.html`, so trailing slashes are dropped from non-folder URLs. If you are migrating from another platform with trailing-slash URLs, existing backlinks may break. Cloudflare Pages handles this more gracefully if that matters to you.

### Syncing a Newer Quartz Version

Quartz releases updates regularly. To pull the latest:

```bash
npx quartz update
```

This merges the latest changes from the upstream Quartz repository into your local branch. Quartz caches your content before merging to reduce the risk of conflicts. If a conflict occurs mid-merge:

- Run `npx quartz restore` to recover your cached content and abort the merge, then resolve manually.
- Or open the conflicting files in your editor (VSCode will show conflict markers), resolve them, and run:

```bash
git add <resolved-files>
git commit
```

> After an update, Quartz sometimes syncs additional workflow files into `.github/workflows/`. I keep only `deploy.yml` and delete the rest to avoid redundant CI runs.

### Verify Deployment

Once pushed, open the **Actions** tab of your GitHub repository. You should see a workflow run triggered by your push with two jobs:

1. **build** — checks out code, installs deps, runs `npx quartz build`, uploads the `public/` directory as an artifact
2. **deploy** — publishes the artifact to GitHub Pages

Both should show a green checkmark. If either fails, click in to read the logs. Common failures:

| Symptom | Fix |
|---|---|
| Node version mismatch | Pin `node-version` in the workflow to `22` |
| `baseUrl` missing or wrong | Set it correctly in `quartz.config.ts` |
| Pages source not set | Go to Settings → Pages → Source → GitHub Actions |
| Environment protection error | Delete the `github-pages` environment in Settings → Environments and re-run |

Once deployed, your site is live at `https://<your-username>.github.io`.
