# The Agent Layer

**AI Agent Security Research**
Site: [theagentlayer.net](https://theagentlayer.net) | Built with Hugo + PaperMod | Hosted on GitHub Pages

---

## Go-Live Checklist

### Prerequisites

- [Hugo Extended](https://gohugo.io/installation/) installed on your local machine (`brew install hugo` on macOS, or download from [GitHub releases](https://github.com/gohugoio/hugo/releases))
- Git installed
- A GitHub account (you: `ahburgess22`)
- Domain `theagentlayer.net` registered at name.com

---

## Step 1 — Create the GitHub Repository

1. Go to https://github.com/new
2. Repository name: `the-agent-layer`
3. Set to **Public**
4. Do NOT initialize with README, .gitignore, or license (you'll push the existing folder)
5. Click **Create repository**

---

## Step 2 — Initialize Git and Add PaperMod Theme

Open a terminal in the `the-agent-layer` folder (the one containing `hugo.toml`):

```bash
# Initialize git repo
git init

# Add the PaperMod theme as a submodule
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod

# Verify the theme is in place
ls themes/PaperMod/
```

You should see theme files (layouts, static, etc.) in `themes/PaperMod/`.

---

## Step 3 — Test the Site Locally

```bash
# Run Hugo development server
hugo server -D

# Visit http://localhost:1313 in your browser
# -D flag shows draft posts too
# Ctrl+C to stop
```

Verify both posts appear, navigation works, and dark mode loads correctly.

---

## Step 4 — Enable GitHub Pages with GitHub Actions

1. Push your code to GitHub:

```bash
# Add GitHub remote (replace with your actual repo URL)
git remote add origin https://github.com/ahburgess22/the-agent-layer.git

# Stage everything
git add .

# Initial commit
git commit -m "Initial commit: Hugo site with PaperMod theme"

# Push to main branch
git push -u origin main
```

2. In your GitHub repository, go to **Settings → Pages**
3. Under **Source**, select **GitHub Actions**
4. The workflow at `.github/workflows/deploy.yml` will trigger automatically on every push to `main`
5. After the first run completes (1–2 minutes), your site will be live at:
   `https://ahburgess22.github.io/the-agent-layer/`

> **Note:** After you point your custom domain below, the site will be at `https://theagentlayer.net`. The GitHub Pages URL will redirect to it.

---

## Step 5 — Configure Custom Domain on GitHub Pages

1. In your GitHub repository, go to **Settings → Pages**
2. Under **Custom domain**, enter: `theagentlayer.net`
3. Click **Save**
4. Check **Enforce HTTPS** (may take a few minutes to become available after DNS propagates)

GitHub will add a `CNAME` file to your repo automatically. If it doesn't, create `static/CNAME` with this content:

```
theagentlayer.net
```

---

## Step 6 — DNS Configuration at name.com

Log into name.com and navigate to **DNS Records** for `theagentlayer.net`.

### Add these records:

**CNAME record (for www subdomain):**

| Type  | Host | Answer                        | TTL  |
|-------|------|-------------------------------|------|
| CNAME | www  | ahburgess22.github.io         | 300  |

**A records (for apex/root domain — required for `theagentlayer.net` without www):**

GitHub Pages requires four A records pointing to their IP addresses:

| Type | Host | Answer         | TTL  |
|------|------|----------------|------|
| A    | @    | 185.199.108.153 | 300 |
| A    | @    | 185.199.109.153 | 300 |
| A    | @    | 185.199.110.153 | 300 |
| A    | @    | 185.199.111.153 | 300 |

> **Note:** `@` means the root/apex domain (`theagentlayer.net`). name.com may show this as blank or `@` — use whatever their UI expects for the root record.

### Verify propagation:

```bash
# Check A records are pointing to GitHub's IPs
dig theagentlayer.net +noall +answer

# Check CNAME for www
dig www.theagentlayer.net +noall +answer
```

DNS can take up to 48 hours to propagate globally, but usually resolves within 15–30 minutes.

---

## Workflow: Publishing New Posts

```bash
# Create a new post
hugo new posts/your-post-title.md

# Edit content/posts/your-post-title.md
# Set draft: false when ready to publish

# Preview locally
hugo server -D

# Publish
git add content/posts/your-post-title.md
git commit -m "Post: your post title"
git push
```

The GitHub Actions workflow automatically builds and deploys on every push to `main`. Posts go live within ~60 seconds of push.

---

## Updating the PaperMod Theme

```bash
git submodule update --remote --merge
git add themes/PaperMod
git commit -m "Update PaperMod theme"
git push
```

---

## Site Structure

```
the-agent-layer/
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions build + deploy
├── content/
│   ├── posts/
│   │   ├── clinejection-supply-chain-compromise.md   # Breach Catalog #001
│   │   └── the-liability-gap.md                      # Discourse post
│   ├── about.md
│   └── search.md
├── archetypes/
│   └── default.md
├── themes/
│   └── PaperMod/               # Added via git submodule
├── static/                     # Static assets (favicon, images, CNAME)
├── layouts/                    # Custom layout overrides (empty for now)
├── assets/                     # Hugo assets pipeline
├── .gitignore
└── hugo.toml                   # Site configuration
```

---

## Configuration Reference

Key settings in `hugo.toml`:

- **baseURL**: Set to `https://theagentlayer.net/` — update if you change the domain
- **defaultTheme**: Set to `dark` — users can still toggle light mode
- **ShowCodeCopyButtons**: Enabled — important for technical posts with code blocks
- **outputs**: Includes `JSON` for the search functionality to work

---

## Troubleshooting

**Posts not showing up:** Check `draft: false` is set in the post's front matter.

**Theme not loading:** Run `git submodule update --init --recursive` — the PaperMod submodule may not have been initialized.

**Search not working:** The search page requires the `JSON` output format in `hugo.toml` (already configured). Make sure `content/search.md` exists with `layout: "search"`.

**GitHub Actions failing:** Check the Actions tab in your repo for the error. Most common issue is Hugo version mismatch — the workflow installs Hugo `0.124.1` Extended, which is what this config was built against.

**HTTPS not available after setting custom domain:** Wait for DNS to propagate fully, then go back to Settings → Pages and click Save again. GitHub re-verifies and issues the certificate.
