# UnixBits v0.2 — local Hugo project

The v0.1 visual design is retained. Content now lives in Markdown and YAML.

## On Debian

Install Hugo and Git:

```sh
sudo apt update
sudo apt install hugo git
```

Check the installed version with `hugo version`. Hugo versions vary across Debian releases; if your version is too old to build this project, install a newer Hugo release.

Start the local preview:

```sh
cd ~/projects/unixbits-v0.2
hugo server
```

Open http://localhost:1313/ in Firefox. Press Ctrl+C to stop the server.

Build the static website with `hugo`; output is written to `public/`.

## Editing content

- Articles: `content/articles/*.md` (set `featured: true` to show an article on the homepage).
- Book chapter companions: `content/companions/*.md`.
- Reading Room bookmarks: `data/reading.yaml`.
- Topic cards: `data/topics.yaml`.
- Approved styling: `static/css/style.css`.
- Homepage template: `layouts/index.html`.

**No GitHub account or internet connection is needed to develop locally** (after Hugo is installed).

The illustrative namespace article and placeholder topic pages should be reviewed before publication.

## Note about publication

Before deploying to GitHub Pages, set `baseURL` in `hugo.toml` to the final site address and configure the build workflow. The current `localhost` value is deliberately for local development.
