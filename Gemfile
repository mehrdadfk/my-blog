# frozen_string_literal: true

source "https://rubygems.org"

# Chirpy is a full Jekyll theme gem — its gemspec pulls in Jekyll itself
# plus every plugin it needs (jekyll-seo-tag, jekyll-feed, jekyll-sitemap,
# jekyll-archives, jekyll-paginate, jemoji, jekyll-include-cache, etc.),
# so we don't list those separately.
#
# We deploy via a GitHub Actions workflow (.github/workflows/pages.yml)
# rather than GitHub's legacy branch-build pipeline, which runs Jekyll in
# "safe mode" and blocks custom plugins/themes outside its small
# allowlist — Chirpy isn't on that list, so Actions-based deploy is a
# requirement here, not just a nicety.
gem "jekyll-theme-chirpy", "~> 7.6"

platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]

# webrick is no longer bundled with Ruby 3+, but `jekyll serve` needs it.
gem "webrick", "~> 1.8"
