# ECEN 468/719 Course Website

Jekyll site for ECEN 468/719 (Texas A&M), hosted on GitHub Pages. Bootstrap 5
and MathJax load from CDNs; `_layouts/default.html` is the only layout.

## Structure

- `labs.md` — landing page (`permalink: /`); one card per `_labs/*`. Each lab
  also renders its own manual page at `/labs/<name>/` (`layout: manual`).
- `policies.md` — plain Markdown.
- `staff.md` — one contact card per `_staff/*`, grouped by `role`.
- `notes.md` — one accordion item per `_notes/*`, newest first.

## Adding content

- **Lab** (`_labs/lab-13.md`): `layout: manual`, `title`, optional `session` /
  `report_due` strings (e.g. `Week 2 (Aug 31 – Sep 4)`), an optional
  `manual_pdf` path (adds a "PDF" button), and a `downloads` list (`label` +
  `file` per code button). The Markdown **body is the lab manual**, rendered as
  a page at `/labs/lab-13/`; reference images as
  `{{ "/assets/files/lab13/img/1.png" | relative_url }}`, and leave the body
  empty to show a "coming soon" placeholder. Put files under
  `assets/files/lab13/`.
- **Note** (`_notes/note-03.md`): `title` + Markdown body, newest file first
  (`$$…$$` for math).
- **Staff** (`_staff/*.md`): `name`, `role` (`Instructor` or
  `Teaching Assistant`), `email`, `office`, `office_hours`, optional `sections`.

## Publishing

Push to GitHub, then Settings → Pages → Deploy from a branch → `main`,
`/ (root)`. In `_config.yml`, `url` is `https://<user>.github.io` and `baseurl`
matches the repo name (`/ecen468`).

## Local preview

`bundle install`, then `bundle exec jekyll serve` → `http://localhost:4000/ecen468/`.
