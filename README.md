# my-blog

Mehrdad's blog. A [Jekyll](https://jekyllrb.com/) site using the
[minima](https://github.com/jekyll/minima) theme, built to be hosted for
free on [GitHub Pages](https://pages.github.com/).

See [SETUP.md](SETUP.md) for how to run it locally, publish it, and connect
a custom domain.

## Quick start

```bash
bundle install
bundle exec jekyll serve
```

Then open http://localhost:4000.

## Writing a new post

Add a file to `_posts/` named `YYYY-MM-DD-title.markdown`:

```markdown
---
layout: post
title:  "My New Post"
date:   2026-11-25 09:00:00 -0000
categories: misc
---
Post content goes here, in Markdown.
```

Commit and push — GitHub Pages rebuilds the site automatically.
