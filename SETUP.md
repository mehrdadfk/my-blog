# Setup guide

This covers three things: running the blog locally, publishing it on GitHub
Pages, and (optionally) putting it on your own domain.

## 1. Run it locally

The Chirpy theme gem requires Ruby ~> 3.1. If `ruby -v` on your machine
is newer than that (this machine's default was 4.0.6), use the scoped
`ruby@3.3` Homebrew install instead of your system Ruby — it doesn't
touch your default Ruby, it's just prefixed onto `PATH` for these
commands:

```bash
cd /Users/m89251/projects/my-blog
PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH" bundle install
PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH" bundle exec jekyll serve
```

(If your system Ruby is already 3.1–3.x, drop the `PATH=...` prefix and
just run `bundle install` / `bundle exec jekyll serve` directly.)

Open http://localhost:4000 — it live-rebuilds when you save a file.

## 2. Publish on GitHub Pages (free)

This repo includes `.github/workflows/pages.yml`, which builds the site
with a modern Jekyll on GitHub's CI and deploys it to Pages. (We use this
instead of GitHub's older "deploy from a branch" pipeline because that
one is pinned to a very old Jekyll version that no longer plays well with
current Ruby — see the comment at the top of the `Gemfile`.)

1. Create a new repo on GitHub named `my-blog` (public or private both
   work with Actions-based Pages; it doesn't need to be
   `<username>.github.io` unless you want the bare
   `https://<username>.github.io` URL with no extra path).
2. Push this folder to it:
   ```bash
   git init
   git add -A
   git commit -m "Initial Jekyll blog"
   git branch -M main
   git remote add origin https://github.com/<your-username>/my-blog.git
   git push -u origin main
   ```
3. In the repo on GitHub: **Settings → Pages** → under "Build and
   deployment", set **Source** to **"GitHub Actions"**. (That's the only
   setting needed — the workflow handles baseurl/subpath automatically.)
4. Push (or re-run the workflow from the **Actions** tab) — after ~1
   minute your blog is live at:
   `https://<your-username>.github.io/my-blog/`
   (or `https://<your-username>.github.io/` if the repo is named
   `<your-username>.github.io`).

I didn't do steps 1–2 automatically — say the word and I'll create the
repo and push for you once you're happy with the content.

## 3. Get a custom domain and point it here

This is the same end result as brooker.co.za, minus the server to
maintain. Total cost: just the domain registration (~$10–15/year, no
separate hosting fee).

### a) Register the domain

Pick a registrar. For cost + ease of use, in order of preference:

| Registrar | Why |
|---|---|
| **Cloudflare Registrar** (recommended) | Sells at *wholesale* price, no markup — typically the cheapest for `.com`. DNS is also free and very fast. Catch: you can only *transfer in* an existing domain to register renewals there in some TLDs, but for most common TLDs (`.com`, `.dev`, `.me`) you can register fresh directly. |
| **Namecheap** | Slightly higher price than wholesale, but anyone can register directly, good UI, free WHOIS privacy. |
| Avoid GoDaddy | Works fine technically, but aggressive upsells and higher renewal prices after year 1. |

Steps (Namecheap, as the most universally available option):
1. Go to namecheap.com, search your desired name (e.g. `mehrdadf.dev`,
   `mehrdadf.com` — `.dev` and `.me` are nice for personal/tech blogs and
   are usually cheap, ~$12–15/yr).
2. Buy it. Enable free WHOIS privacy (usually on by default now).
3. You do **not** need to buy any hosting/email add-ons they'll try to
   upsell — you only need the domain itself.

### b) Point DNS at GitHub Pages

In your registrar's DNS management panel for the domain:

**If you want the bare domain** (`mehrdadf.dev`) to work, add these four
`A` records (GitHub Pages' fixed IPs) at the root (`@`):

```
A    @    185.199.108.153
A    @    185.199.109.153
A    @    185.199.110.153
A    @    185.199.111.153
```

**If you want `www.mehrdadf.dev`** to work too (recommended, and GitHub
will redirect one to the other depending on your `CNAME` file), add:

```
CNAME    www    <your-username>.github.io.
```

(Most people set up both: apex via the four `A` records, `www` via
`CNAME`, then tell GitHub which one is canonical in the next step.)

### c) Tell GitHub about the domain

Because this repo deploys via the GitHub Actions workflow (not the legacy
"deploy from a branch" pipeline), you need to add the `CNAME` file
yourself so it survives every rebuild:

1. Create a file named `CNAME` (no extension) at the **root of this
   repo** (next to `_config.yml`), containing just the domain on one
   line, e.g.:
   ```
   mehrdadf.dev
   ```
   Commit and push it — it'll get copied into `_site/CNAME` on every
   build, which is how GitHub Pages knows the domain.
2. In the repo: **Settings → Pages → Custom domain**, enter the same
   domain (`mehrdadf.dev`), save. Wait for the DNS check to go green
   (can take a few minutes up to a few hours for DNS to propagate).
3. Once it's green, tick **Enforce HTTPS** — GitHub provisions a free
   Let's Encrypt certificate automatically.
4. Update `_config.yml`:
   ```yaml
   url: "https://mehrdadf.dev"
   baseurl: ""
   ```
   Commit and push.

That's it — no server, no nginx, no TLS renewal to manage, and the only
recurring cost is the ~$12–15/yr domain renewal.
