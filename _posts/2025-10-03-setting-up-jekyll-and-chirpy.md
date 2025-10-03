---
title: Jekyll & Chirpy Setup
description: >
  Get started with Jekyll & the Chirpy theme following this guide. You will learn how to install, configure, and use your first Chirpy-based website, as well as deploy it to a web server.
author: greedy
date: 2025-10-03 12:00:00 +1030
categories: [Blogging, Jekyll]
tags: [homelab, jekyll, chirpy]
---

Want to copy my blog website? Follow this guide!

## DISCLAIMER

My blog and this entire setup guide is based off Techno Tim's older video, but this post updates everything for current Jekyll, Ruby, Bundler, and Chirpy. This guide favors reproducible commands, minimal gotchas, and GitHub Pages deployment that actually works today.

## What you'll build

- A local Jekyll workspace with the Chirpy theme running at [http://127.0.0.1:4000](http://127.0.0.1:4000)
- A GitHub repository configured for Pages with workflow-based builds (no local build artifacts committed)
- A clean content workflow: write posts in Markdown, preview locally, push to publish

## Prerequisites

- Linux workstation or WSL with Git, Ruby, and Bundler; VS Code is optional but recommended
- A GitHub account and a repository (repo name can be anything; for user sites, use `username.github.io`)

---

## 1) Install Ruby and Bundler

Use system packages or a version manager. For Ubuntu 22.04+:

```bash
sudo apt update
sudo apt install -y ruby-full build-essential zlib1g-dev
```

Add gem bin path to shell:

```bash
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Install Jekyll and Bundler:

```bash
gem install jekyll bundler
```

> This isolates gems to the home directory to avoid sudo usage and permission issues common for beginners on Linux.

---

## 2) Create a new Jekyll site

```bash
jekyll new my-blog --skip-bundle
cd my-blog
```

> Skipping the initial bundle lets Chirpy be added cleanly next.

---

## 3) Add the Chirpy theme

### Theme gem approach (recommended)

Edit your Gemfile:

```ruby
gem "jekyll", "~> 4.3"
gem "jekyll-theme-chirpy", "~> 7.0"
```

In `_config.yml`:

```yaml
theme: jekyll-theme-chirpy
plugins:
  - jekyll-feed
  - jekyll-seo-tag
  - jekyll-sitemap
```

Remove minima references if present. This keeps the theme as a dependency with minimal repo clutter.

### Starter repo approach

Fork or use a Chirpy starter template and clone it locally.

Keep Gemfile and `_config.yml` as provided; customize content only.

> **Tip:** Beginners should prefer the theme gem approach.

---

## 4) Install dependencies and run locally

```bash
bundle install
bundle exec jekyll serve
```

Open [http://127.0.0.1:4000](http://127.0.0.1:4000) to preview the site.

> If port 4000 is in use, specify a port:

```bash
bundle exec jekyll serve --livereload --port 4001
```

Live reload automatically refreshes the browser on file changes, great when drafting posts in VS Code.

---

## 5) Configure site metadata

Edit `_config.yml`:

```yaml
title: "Your Blog Title"
tagline: "Optional tagline"
description: "A brief description"
url: "https://USERNAME.github.io" # for user site
baseurl: "" # for user site, "/REPO" for project site
author:
  name: greedy
links:
  - GitHub
  - Mastodon/X
timezone: Australia/Adelaide
plugins:
  - jekyll-seo-tag
  - jekyll-sitemap
  - jekyll-feed
```

If using a custom domain:

```bash
echo "blog.example.com" > CNAME
```

---

## 6) Create the first post

```bash
mkdir -p _posts
```

Create `_posts/2025-10-03-jekyll-chirpy-setup.md` with the front matter:

```yaml
---
layout: post
title: "Jekyll & Chirpy Setup"
description: "Install, configure, and deploy a Chirpy-based blog."
categories: [Blogging, Jekyll]
tags: [homelab, jekyll, chirpy]
author: greedy
date: 2025-10-03 12:00:00 +1030
---
```

Then paste the content (this article) below that block.

---

## 7) Add pages and navigation

- **About:** create `about.md` with `layout: page` and add to nav in `_config.yml`.
- **Links/Projects:** similar page with a simple Markdown list.

Chirpy exposes nav/sidebar via `_config.yml`.

---

## 8) Customize Chirpy

- **Colors/light/dark:** override minimal CSS in assets if desired.
- **Home index:** adjust `index.md` or theme settings.
- **Social cards/SEO:** ensure jekyll-seo-tag metadata is set.

**Advanced:** create `_data/navigation.yml` or `_data/social.yml` to centralize links; add analytics in `_includes/head`.

---

## 9) Initialize Git and push

```bash
git init
git add .
git commit -m "chore: initial Jekyll + Chirpy"
git remote add origin git@github.com:USERNAME/REPO.git
git branch -M main
git push -u origin main
```

Use SSH keys for smoother pushes.

---

## 10) Deploy to GitHub Pages (workflow method)

Create `.github/workflows/pages.yml`:

```yaml
name: Deploy Jekyll with Chirpy

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true
      - name: Install dependencies
        run: bundle install --path vendor/bundle
      - name: Build site
        run: bundle exec jekyll build --trace
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./_site

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

Commit and push:

```bash
git add .github/workflows/pages.yml
git commit -m "ci: GitHub Pages workflow"
git push
```

---

## 11) Domain and HTTPS

- For `username.github.io`, DNS is automatic.
- **Custom domains:** point A/AAAA or CNAME to GitHub Pages; include CNAME file.
- Enable "Enforce HTTPS" in Pages settings.

---

## 12) Writing workflow

Draft locally:

```bash
bundle exec jekyll serve --livereload
```

Helper to create new posts:

```bash
DATE=$(date +%F)
FILE="_posts/${DATE}-new-post.md"
printf -- "---\nlayout: post\ntitle: \"New Post\"\ndate: $(date +"%Y-%m-%d %H:%M:%S %z")\ncategories: [Blogging]\ntags: [note]\n---\n\n" > "$FILE"
echo "Created $FILE"
```

---

## 13) Common pitfalls and fixes

- **Ruby mismatch:** set `ruby-version: '3.2'` in workflow; commit `Gemfile.lock`.
- **baseurl mistakes:** use `{{ site.baseurl }}` for project sites.
- **Missing plugins:** workflow method ensures SEO/sitemap feed build.
- **Permission errors:** use `GEM_HOME` to avoid sudo issues.

---

## 14) Optional: containerized local dev

Create `Dockerfile`:

```dockerfile
FROM ruby:3.2-bookworm
RUN apt-get update && apt-get install -y build-essential && rm -rf /var/lib/apt/lists/*
WORKDIR /srv/jekyll
COPY Gemfile* ./
RUN gem install bundler && bundle install || true
COPY . .
EXPOSE 4000
CMD ["bash","-lc","bundle install && bundle exec jekyll serve --host 0.0.0.0 --livereload"]
```

Build and run:

```bash
docker build -t jekyll-chirpy .
docker run --rm -p 4000:4000 -v "$PWD":/srv/jekyll jekyll-chirpy
```

---

## 15) Next steps

- Add analytics, comments (Giscus), sitemap.
- Set up backup workflow for `_site` and repo.
- Ensure local preview, `_config.yml`, Actions workflow, HTTPS, and first post render correctly.

This guide uses a modern toolchain with GitHub Actions, giving beginners and power users a predictable and clean blogging workflow.