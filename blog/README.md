# Blog — how it works

A zero-build Markdown blog. You write posts as plain `.md` files; the pages render
them in the browser (no Jekyll, no build step, works on GitHub Pages as-is).

## Files

- `index.html` — the blog landing page. Lists every post from `posts.json`, newest first.
- `post.html` — renders a single post. Opened as `post.html?p=<slug>`.
- `posts.json` — the list of posts (title, date, tags, excerpt). **This drives the index.**
- `posts/*.md` — one Markdown file per post.
- `blog.css` — shared styling, reusing the portfolio's colour palette + dark/light theme.

## Add a new post (2 steps)

1. **Create the Markdown file** in `posts/` and name it with the slug you'll use.
   The convention is `YYYY-MM-DD-short-title.md`, e.g.:

   ```
   posts/2026-09-01-gradient-descent.md
   ```

   Slugs may only contain letters, numbers and hyphens.

2. **Add one entry to `posts.json`** (newest anywhere — the page sorts by date):

   ```json
   {
     "slug": "2026-09-01-gradient-descent",
     "title": "How Gradient Descent Actually Works",
     "date": "2026-09-01",
     "tags": ["Machine Learning", "Optimization"],
     "excerpt": "A short summary shown on the blog index."
   }
   ```

   `slug` **must** match the `.md` filename (without the `.md`). That's the only rule.

Commit and push — GitHub Pages serves it automatically.

## What you can use in a post

Standard Markdown, plus:

- **Code blocks** with syntax highlighting — ```` ```python ````, ```` ```java ````, etc.
- **Math** via KaTeX: inline with `$ ... $`, display with `$$ ... $$`.
- Tables, blockquotes, images (put images under `posts/` or `../resources/` and link them).

The first `# Heading` in the file is optional — the title from `posts.json` is shown
at the top automatically, and a matching leading `# Title` is stripped to avoid a duplicate.

## Previewing locally

Because posts are loaded with `fetch()`, opening the files directly from disk
(`file://`) is blocked by the browser. Run a tiny local server from the
`subbarayudu-chakali` folder instead:

```bash
python -m http.server 8000
```

Then open http://localhost:8000/blog/ . On the live GitHub Pages site this isn't
needed — it just works.
