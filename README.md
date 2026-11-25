# my-blog

Mehrdad's blog. A [Jekyll](https://jekyllrb.com/) site using the
[Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) theme, built to
be hosted for free on [GitHub Pages](https://pages.github.com/) via a
GitHub Actions build (not the legacy branch-build pipeline — Chirpy isn't
on GitHub's restricted theme allowlist that pipeline requires).

See [SETUP.md](SETUP.md) for how to run it locally, publish it, and connect
a custom domain.

## Quick start

Chirpy's gemspec requires Ruby ~> 3.1; this machine's default Ruby was
newer, so gems are installed/run with Homebrew's `ruby@3.3` instead (see
`.ruby-version`). Adjust the `PATH` prefix below if your system Ruby is
already in range.

```bash
PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH" bundle install
PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH" bundle exec jekyll serve
```

Then open http://localhost:4000. (The GitHub Actions workflow pins Ruby
3.3 too, so CI builds are unaffected by local Ruby version either way.)

## Writing a new post

Add a file to `_posts/` named `YYYY-MM-DD-title.markdown`:

```markdown
---
layout: post
title:  "My New Post"
date:   2026-11-25 09:00:00 -0000
categories: [misc]
---
Post content goes here, in Markdown.
```

Posts are served at `/posts/<title>/` (Chirpy's URL scheme — see the
`defaults` block in `_config.yml`). Commit and push — GitHub Pages
rebuilds the site automatically.

## Editing the About page / sidebar links

- About page content: `_tabs/about.md`
- Contact icons shown in the sidebar (GitHub, email, RSS, etc.):
  `_data/contact.yml`, driven by the `github.username` / `social.email`
  fields in `_config.yml`
- Site title/tagline/description: top of `_config.yml`
