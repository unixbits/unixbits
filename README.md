# UnixBits

UnixBits is a static technical website dedicated to Linux, Unix, BSD, and related systems technologies.

It combines original technical articles, practical experiments, historical material, and manually curated external resources.

The project also serves as a companion to the Leanpub book *Understanding Linux: The Kernel Perspective*.

UnixBits is built with Hugo and designed to require minimal operational maintenance.

## Project status

**Version:** v0.3 — Topic pages

The website is under local Git version control. Development and testing are performed locally on Debian.

The GitHub account `unixbits` has been reserved, but the repository has not yet been connected to GitHub or published.

## Technology

- Hugo static-site generator
- HTML and CSS
- Markdown for articles and topic pages
- YAML for structured data and content metadata
- Git for version control
- GitHub Pages planned for hosting

No JavaScript framework, database, backend, or content management system is required.

## Local development

On Debian, install Hugo and Git:

```sh
sudo apt update
sudo apt install hugo git
```

Check the Hugo version:

```sh
hugo version
```

Hugo versions vary across Debian releases. A newer Hugo release may be required if the distribution package is too old.

Start the local development server:

```sh
cd ~/projects/unixbits-v0.3
hugo server
```

Open http://localhost:1313/ in a browser.

Hugo automatically rebuilds the website when source files change. Press Ctrl+C to stop the server.

To generate the static website:

```sh
hugo --cleanDestinationDir
```

The generated website is written to `public/`, which is excluded from Git.

## Website organisation

The homepage contains six tiles:

1. Linux Internals
2. BSD & Unix
3. Unix History
4. Unix Tools
5. Real-Time Linux
6. Reading Room

The first five represent technical subjects. Reading Room provides access to the consolidated collection of curated external resources.

The homepage design is approved and should be preserved unless deliberately revised.

## Content architecture

### Topic pages

Technical topics live in:

`content/topics/`

They use the reusable Hugo template:

`layouts/topics/single.html`

Each topic page can display:

- An introduction
- Original UnixBits articles
- Manually curated external links
- Related book companions

Sections without content are omitted automatically.

### Original articles

Articles live in:

`content/articles/`

An article can be associated with one or more topics through YAML front matter:

```yaml
topics:
  - linux-internals
  - real-time-linux
```

Articles with `featured: true` can appear in the homepage's featured content area.

### Book companions

Book-related material lives in:

`content/companions/`

Companions can be associated with technical topics using the same `topics` front-matter field.

### Curated external reading

External references are maintained manually in:

`data/reading.yaml`

Each entry contains a title, URL, description, and one or more topic identifiers.

Example:

```yaml
- title: Linux kernel documentation
  url: https://docs.kernel.org/
  description: Official kernel documentation
  topics:
    - linux-internals
    - real-time-linux
```

A reference can appear on multiple topic pages without duplicating its YAML entry.

The planned Reading Room page will consolidate these references, grouped by technical subject, using the same data file.

## Important project files

| File or directory | Purpose |
|---|---|
| `hugo.toml` | Hugo configuration |
| `layouts/index.html` | Homepage template |
| `layouts/topics/single.html` | Reusable topic-page template |
| `layouts/_default/` | General page templates |
| `layouts/partials/` | Shared header and footer |
| `static/css/style.css` | Website styling |
| `content/articles/` | Original technical articles |
| `content/companions/` | Book companion material |
| `content/topics/` | Technical topic pages |
| `data/topics.yaml` | Homepage topic tiles |
| `data/reading.yaml` | Curated external resources |

## Git workflow

The project uses a local Git repository.

Check changes:

```sh
git status
git diff
```

Stage and commit completed changes:

```sh
git add .
git commit -m "Describe the change"
```

Inspect recent history:

```sh
git log --oneline -5
```

Generated Hugo output and temporary files are excluded through `.gitignore`.

GitHub integration will be configured when the website is ready for publication.

## Development roadmap

- **v0.1:** Initial visual design
- **v0.2:** Hugo content structure using Markdown and YAML
- **v0.3:** Reusable topic pages and curated reading associations
- **v0.4:** Consolidated Reading Room
- **Later:** Additional original content, book integration, and GitHub Pages deployment

## Project principles

UnixBits is both a useful technical resource and a practical environment for learning modern web-development technologies.

The guiding principles are:

- Prefer simplicity over unnecessary features.
- Keep operational maintenance minimal.
- Publish useful, durable technical material rather than chase news.
- Curate external resources manually.
- Keep the website useful independently of the book.
- Preserve the ability to learn and experiment with Hugo and related technologies.
- Avoid subscriptions, accounts, databases, and maintenance-heavy infrastructure.

There is no fixed publication schedule.

## Publication

Before deploying to GitHub Pages:

- Set `baseURL` in `hugo.toml` to the final public address.
- Configure the GitHub repository and Hugo build workflow.
- Verify internal links and external references.
- Test the production build.

The current local configuration is intended for development.
