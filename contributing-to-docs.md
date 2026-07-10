# How to contribute to these docs

This site is built with [Jekyll](https://jekyllrb.com/docs/) and the [Just the Docs](https://just-the-docs.com/) theme. Every `.md` file with a front matter block becomes a page; `.md` files are rendered as `.html` pages on the published site.

## Editing an existing page

There are two ways to make a change:

1. **Directly in the GitHub web interface**
   * Go to the file you want to update.
   * Edit it, then either commit directly to `main`, or create a branch and open a pull request.
   * If you commit directly to `main`, wait a few minutes for the change to appear on the [GitHub Pages site](https://zendro-dev.github.io/).
2. **Clone the repository locally**
   ```
   git clone https://github.com/Zendro-dev/Zendro-dev.github.io.git
   ```
   Edit the files, then:
   ```
   git add <file>
   git commit -m "<your-commit-message>"
   git push
   ```
   Again, wait a few minutes for the change to show up on the live site.

Every push to `main` triggers a GitHub Actions build (see `.github/workflows/pages.yml`), so there is no manual deploy step.

## Adding a new page

Each section of the site lives in its own folder (`getting-started/`, `data-models/`, `guides/`, `authentication/`, `api/`), with an `index.md` as that section's landing page. To add a new page to an existing section, add a new `.md` file to that folder with a front matter block like this:

```md
---
layout: default
title: My new page
parent: <Section title, matching the section's index.md `title:`>
nav_order: <position within the section>
permalink: /<section-folder>/my-new-page
---
```

* `title` is what shows up in the left-hand navigation.
* `parent` must exactly match the `title:` of the section's `index.md` (e.g. `Getting started`, `Data models`, `User guides`, `Authentication`, `Zendro API`).
* `nav_order` controls its position among its siblings — it only needs to be unique among pages sharing the same `parent`.
* `permalink` sets the page's URL. Keep it consistent with the folder it lives in.

If the new page itself needs children (a sub-section), add `has_children: true` to its front matter, and have the child pages set `parent:` to this new page's title, optionally with `grand_parent:` if they're nested two levels deep. See any existing page for a working example — for instance, [`data-models/index.md`](data-models/index.md) has children, and [`data-models/associations.md`](data-models/associations.md) is one of them.

To add a whole new top-level section instead, create a new folder with its own `index.md` (`has_children: true`, no `parent:`, and a `nav_order` that fits where you want it among the other top-level sections), following the pattern of the existing ones.

Because navigation position and hierarchy are controlled entirely by front matter (`parent`, `nav_order`, `permalink`), there is no separate file/menu mapping to keep in sync — the front matter *is* the map. If you rename or move a page, only its own front matter needs to change; just double check nothing links to its old path with a hardcoded URL (prefer the `{% link %}` tag over hardcoded links — see below).

## Linking between pages

Prefer Jekyll's `{% link %}` tag over hardcoded relative links or absolute URLs, e.g.:

```
See [Environment variables]({% link getting-started/environment-variables.md %}) for details.
```

This takes the *file path* relative to the site root (not the URL), and Jekyll resolves it to the correct URL at build time — if a page's `permalink` changes later, links using `{% link %}` keep working, while hardcoded paths would silently break.

For images and other static assets, use a site-root-relative path (e.g. `/figures/my-image.png`), which works regardless of how deeply nested the page using it is.

## Adding a table of contents

To show a per-page table of contents, add this where you want it to appear (usually right after the page title):

```md
## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}
```

## Adding codeblocks

Code blocks get syntax highlighting. The default highlighting is plaintext, which highlights as console-like (black background, white foreground) block. This should only used for things that might appear as terminal output For commands use `bash`. Please ***don't*** add a prepending `$` to code block lines. They make copy-pasting the code very annoying.

For other syntax look [here](https://rouge-ruby.github.io/docs/file.Languages.html) for a suiting language, as the actual might not suit the needs, like `jsom` with comments works better with `js` syntax, or `graphql` with `python`

~~~md
```python
query {
   ...DoSomething
}
```

```bash
zendro start
```
~~~

## Building and previewing locally

```bash
bundle install
bundle exec jekyll serve
```

This starts a local server (by default at `http://localhost:4000`) that rebuilds the site as you edit files, so you can check navigation, links and formatting before pushing.
