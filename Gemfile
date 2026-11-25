source "https://rubygems.org"

# Plain, current Jekyll — not the `github-pages` meta-gem, which pins a
# very old Jekyll (3.9) incompatible with modern Ruby (3.4+/4.x) due to
# stdlib gems (csv, bigdecimal, etc.) it assumes are still bundled.
# We deploy via a GitHub Actions workflow (see .github/workflows/pages.yml)
# instead of GitHub's legacy branch-build pipeline, so we aren't limited
# to the old `github-pages` gem's pinned plugin versions.
gem "jekyll", "~> 4.3"
gem "minima", "~> 2.5"

group :jekyll_plugins do
  gem "jekyll-feed"
  gem "jekyll-sitemap"
  gem "jekyll-seo-tag"
end

# webrick is no longer bundled with Ruby 3+, but `jekyll serve` needs it.
gem "webrick", "~> 1.8"
