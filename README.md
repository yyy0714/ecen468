# ECEN 468 Course Website

Course website for ECEN 468 (Texas A&M University). A plain Jekyll site styled
with stock Bootstrap 5 (loaded from the jsDelivr CDN), built and hosted by
GitHub Pages. Math is typeset with MathJax.

## Layout

- `index.md` — landing page with a table of contents.
- `labs.md` — renders every document in `_labs/` as a card with download buttons.
- `policies.md` — course policies, plain Markdown.
- `staff.md` — renders every document in `_staff/` as a contact card,
  grouped by `role` (`Instructor` or `Teaching Assistant`).
- `notes.md` — renders every document in `_notes/` as an accordion item,
  newest file first.
- `_layouts/default.html` — the only layout: Bootstrap page frame with the sidebar.
- `assets/files/labXX/` — the downloadable lab manuals and code archives.

## Adding content

- **Lab:** put the files in `assets/files/lab13/`, then create `_labs/lab-13.md`
  with a `title` and a `downloads` list (`label` + `file` per entry, one entry
  per button). See any existing file in `_labs/` for the format.
- **Note:** create `_notes/note-03.md` with a `title` and a Markdown body.
  Notes are shown newest-file first. Use `$$...$$` for math.
- **Staff:** create a file in `_staff/` with `name`, `role`, `email`,
  `office`, and `office_hours`.

## Publishing

1. In `_config.yml`, `url` must be `https://<github-username>.github.io` and
   `baseurl` must match the repository name (currently `/ecen468`).
2. Push to GitHub, then enable Pages: Settings → Pages → Source:
   "Deploy from a branch" → main branch, `/ (root)`.

## Local preview (optional)

Requires Ruby: `bundle install`, then `bundle exec jekyll serve` and open
`http://localhost:4000/ecen468/`.
