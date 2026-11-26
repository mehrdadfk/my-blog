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

## 3. Custom domain: mehrdadfk.com

Domain is bought. `_config.yml` (`url: "https://mehrdadfk.com"`) and the
`CNAME` file (containing `mehrdadfk.com`) are already set up in this repo
— the only things left are DNS, at your registrar, and one field in the
GitHub repo settings.

### a) Point DNS at GitHub Pages

In your registrar's DNS management panel for `mehrdadfk.com`, add these
four `A` records at the root/apex (`@`) — GitHub Pages' fixed IPs:

```
A    @    185.199.108.153
A    @    185.199.109.153
A    @    185.199.110.153
A    @    185.199.111.153
```

If you also want `www.mehrdadfk.com` to work (GitHub will redirect it to
the apex automatically once both are set up), add:

```
CNAME    www    mehrdadfk.github.io.
```

DNS propagation can take anywhere from a few minutes to a few hours.

### b) Tell GitHub about the domain

1. Push this repo's `CNAME` file (already created, containing
   `mehrdadfk.com`) if you haven't yet — it's what tells GitHub Pages
   which domain to serve on every Actions-built deploy.
2. In the repo: **Settings → Pages → Custom domain**, enter
   `mehrdadfk.com`, save. Wait for the DNS check to go green (depends on
   step (a) having propagated).
3. Once it's green, tick **Enforce HTTPS** — GitHub provisions a free
   Let's Encrypt certificate automatically. (Until this is ticked, or
   right after the domain is first added, you may briefly see a
   certificate warning — normal during provisioning, not something to
   fix.)

### c) One transition quirk worth knowing

Once the custom domain is configured, the site's internal links/assets
are built assuming it's served at the **root** of a domain (no `/my-blog/`
prefix) — that's correct for `mehrdadfk.com/`, but it means the old
`https://mehrdadfk.github.io/my-blog/` URL will start showing a page with
broken CSS/JS (links will point at the wrong path). That's expected, not
a bug — just use `mehrdadfk.com` going forward and don't worry about the
old URL.

### d) After it's live: update Google Search Console

The `github.io` property you may have already set up in Search Console
is a *separate* property from the custom domain. Once `mehrdadfk.com` is
serving correctly over HTTPS:

1. Add a **new property** in Search Console, type **Domain**, value
   `mehrdadfk.com` — this method verifies via a DNS TXT record (added at
   the same registrar DNS panel as step (a)) and automatically covers
   `http`/`https`/`www`/apex in one property, unlike the URL-prefix type.
2. Submit `sitemap.xml` there too.
3. The old `github.io` URL-prefix property can stay or be removed —
   doesn't matter much either way once the domain is the canonical one.

Total ongoing cost from here: just the domain's yearly renewal — no
server, no nginx, no TLS renewal to manage by hand.
