# ECEN 468/719 Course Website

Jekyll site for ECEN 468/719 (Texas A&M), hosted on GitHub Pages. Bootstrap 5
and MathJax load from CDNs; `_layouts/default.html` is the only layout.

## Structure

- `labs.md` — landing page (`permalink: /`); one card per `_labs/*`.
- `policies.md` — plain Markdown.
- `staff.md` — one contact card per `_staff/*`, grouped by `role`.
- `notes.md` — one accordion item per `_notes/*`, newest first.

## Adding content

- **Lab** (`_labs/lab-13.md`): `title`, a `downloads` list (`label` + `file`
  per button), and optional `session` / `report_due` strings shown on the card
  (e.g. `Week 2 (Aug 31 – Sep 4)`). Put the files under `assets/files/lab13/`.
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
