# CLAUDE.md

Personal academic site built on [al-folio](https://github.com/alshedivat/al-folio) (Jekyll).
Deployed by `.github/workflows/deploy.yml` on push, which builds with **Ruby 3.3.5**.

## Local preview

The system Ruby (2.6) cannot build this site, and `bundle` on `PATH` resolves to it. Use the
Homebrew `ruby@3.3` keg explicitly:

```bash
export PATH="/opt/homebrew/opt/ruby@3.3/bin:$PATH"
export GEM_HOME="$HOME/.gem/ruby/3.3.0" GEM_PATH="$HOME/.gem/ruby/3.3.0"
bundle exec jekyll build --watch &                                # rebuild on edit
python3 -m http.server 4000 --directory _site --bind 127.0.0.1    # http://localhost:4000
```

First-time setup: `brew install ruby@3.3 imagemagick`, then
`bundle config set --local path vendor/bundle && bundle install`.

Four things that will otherwise waste time:

- **`jekyll serve` is broken on this machine — use the two commands above instead.** WEBrick
  serves static files with the `sendfile` syscall, which endpoint security (Santa) blocks. Pages
  return HTTP 200 with a **0-byte body**, so CSS, images, and PDFs silently fail while HTML loads
  fine — the site renders unstyled and imageless, which looks like missing files but is not.
  `baseurl` is empty, so a plain static server over `_site` resolves paths identically. The only
  loss is live-reload; refresh manually.

- **Do not use Ruby 4.x.** Ruby 4.0 removed `CGI.parse`, which the bundler/Jekyll stack still
  calls; `bundle install` dies with `undefined method 'parse' for class CGI`. Match CI's 3.3.
- **ImageMagick is required** — `imagemagick.enabled: true` (`_config.yml`). Without the binary
  the build fails. For a quick preview you can flip it to `false`; CI still generates responsive
  images on deploy.
- **Gems must go somewhere writable.** If endpoint security (e.g. Santa) blocks writes to
  `/opt/homebrew`, the `GEM_HOME` export above is what makes `bundle install` work — setting only
  `bundle config path` is not enough, because the gem *cache* still writes to the system gem dir.
  Both `vendor/` and `.bundle` are gitignored.

`Gemfile.lock` is tracked despite being listed in `.gitignore` (committed before that rule), and
pins `BUNDLED WITH 2.5.18`. A newer bundler may rewrite it — check `git status` before committing.

## Content

- `_news/` — one Markdown file per item, `inline: true`. Rendered by `_includes/news.liquid` as a
  flat reverse-chronological table; it does **not** group by year. The homepage shows only the
  most recent few (`announcements.limit` in `_config.yml`); `/news/` shows all.
- `_projects/` — one file per research area, grouped on `/projects/` by `category`. The categories
  that render, and their order, come from `display_categories` in `_pages/projects.md`; a project
  whose `category` is not listed there is silently dropped. `importance` orders within a category.
  `img` is optional (the card template guards it).
- `_bibliography/papers.bib` — set `related_publications: true` in a project's front matter and
  cite with `{% cite key1 key2 %}` to render a References section, rather
  than hand-typing citations.

### News dates: omit the timezone offset

Write `date: 2022-05-04 12:00:00` with **no** UTC offset. `_config.yml` sets no `timezone`, so
Jekyll uses the build machine's local zone — an explicit offset that disagrees with it shifts the
displayed date by a day (a `+0800` timestamp renders as the previous day when built in US Pacific,
and CI builds in UTC). A bare midday time renders identically everywhere.

### Placeholders

Unfinished news entries are kept as `_news/0000-00-00-TODO-*.md` with `published: false`, so they
do not render until the date and text are filled in and the flag is removed.
