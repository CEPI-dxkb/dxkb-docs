# dxkb-docs

Documentation site for the **dxkb** web application. This is a [Sphinx](https://www.sphinx-doc.org/)
project (Markdown via [MyST](https://myst-parser.readthedocs.io/)) that builds to static HTML,
published at **`https://www.dxkb.org/docs/`**.

It is the direct counterpart to [`BV-BRC-Docs`](https://github.com/BV-BRC/BV-BRC-Docs) — the
service quick-reference pages here were adapted from that project and rebranded for dxkb.

---

## Why this repo exists — the service info dialogs

Every service page in **dxkb-web** has an **ⓘ (info)** icon in its title that opens an
**Overview** dialog describing the service. That content is **not** stored in dxkb-web. At runtime,
the app fetches an HTML page from this docs site and pulls one section out of it.

The relevant code lives in `dxkb-web` at `public/js/p3/widget/app/AppBase.js`, method `gethelp()`:

1. It fetches `docsServiceURL + applicationHelp` — e.g.
   `https://www.dxkb.org/docs/` + `quick_references/services/genome_annotation_service.html`.
2. It finds every `.infobutton` on the page and reads its `name` attribute (e.g. `overview`).
3. It grabs the element in the fetched HTML whose `id` **matches that name** and injects it into
   a dialog. `name="overview"` → the `<section id="overview">` block, `name="parameters"` →
   `<section id="parameters">`, and so on.

**The contract this repo must honor:** each service `.md` file needs a `## Overview` heading
(and optionally `## Parameters`, etc.). Sphinx/MyST auto-generates the matching HTML `id` from the
heading text (this is enabled by `myst_heading_anchors = 2` in `conf.py`). So `## Overview`
becomes `<section id="overview">`, which is exactly what the info icon looks for.

If the fetch fails (e.g. this site is unreachable), dxkb-web now shows a graceful
"Help information is currently unavailable" message instead of a silently dead icon — but the
**only** way to get real content is to publish this site. See "Deployment" below.

---

## Repository layout

```
dxkb-docs/
├── README.md                  ← you are here
├── requirements.txt           ← Python/Sphinx dependencies
└── docroot/
    ├── conf.py                ← Sphinx config (project name, theme, myst_heading_anchors)
    ├── Makefile / make.bat    ← `make html` build entry points
    ├── index.rst              ← master doc; globs in the service pages
    ├── _static/               ← favicon, etc.
    ├── spelling_wordlist.txt  ← allow-list for the spelling checker
    └── quick_references/
        └── services/          ← ONE .md file per service (the actual content)
```

Build output lands in `docroot/_build/html/` and is **git-ignored** — never commit it.

---

## Prerequisites

- **Python 3.8+** (the pinned deps in `requirements.txt` were verified on 3.8).
- `make` (standard on macOS/Linux; on Windows use `make.bat`).

---

## Build it locally

```bash
# 1. Create and activate a virtual environment (once)
python3 -m venv venv
source venv/bin/activate           # Windows: venv\Scripts\activate

# 2. Install dependencies (once)
pip install -r requirements.txt

# 3. Build the HTML
cd docroot
make html
```

The site is generated at `docroot/_build/html/`. Open
`docroot/_build/html/index.html` in a browser to browse it, or verify a single service page:

```bash
# should print: id="overview"
grep -o 'id="overview"' _build/html/quick_references/services/genome_annotation_service.html
```

> **Note on image warnings:** this repo currently contains only the `.md` text (not the
> screenshot images), so `make html` prints "image file not readable" warnings. These are
> **harmless for the info dialogs** — the dialog shows only the text of the `## Overview` section.
> If you want the fully-browsable site with screenshots, copy the corresponding `images/` folders
> from `BV-BRC-Docs/docroot/quick_references/` (and adjust as needed).

---

## Test the dialogs against a local dxkb-web (no deploy needed)

You do **not** need to deploy this site to test it. Because dxkb-web's `PathJoin` returns a
same-origin URL when `docsServiceURL` has no `http(s)://` prefix, and dxkb-web already serves its
`public/` folder at `/public/`, you can serve the built docs straight out of dxkb-web:

```bash
# From dxkb-docs, after `make html`:
cp -r docroot/_build/html/* /path/to/dxkb-web/public/docs/

# In dxkb-web, edit p3-web.conf (git-ignored, local only) and add:
#   "docsServiceURL": "/public/docs"

# Start dxkb-web (npm start) and click an ⓘ icon, e.g. http://localhost:3000/app/Annotation
```

`public/docs/` and the `docsServiceURL` override are **local test scaffolding only** — both are
git-ignored in dxkb-web and must never be committed. In production, dxkb-web keeps
`docsServiceURL` pointing at `https://www.dxkb.org/docs/` and this site is published there.

---

## Deployment (production)

The built static HTML must be served at **`https://www.dxkb.org/docs/`**. This mirrors how
`BV-BRC-Docs` is published to `bv-brc.org/docs/`.

1. `cd docroot && make html` → produces `docroot/_build/html/`.
2. Serve/copy that directory's contents to whatever hosts the `dxkb.org` `/docs/` path
   (nginx static root, object storage + CDN, a CI publish job, etc.).

> **Currently `https://www.dxkb.org/docs/` returns HTTP 403** — nothing is published there yet.
> Until someone with dxkb.org hosting access serves the build output at `/docs/`, the info icons
> in production will show the graceful fallback message. **This hosting step is the final unblock
> and lives outside both repos.**

A CI job (e.g. GitHub Action) that runs `make html` and publishes on merge to the main branch is
recommended so content edits don't require a manual build/upload.

---

## Editing / adding a service doc

- Each service maps to one file: `docroot/quick_references/services/<name>.md`. The `<name>`
  must match the `applicationHelp` path set on the corresponding widget in dxkb-web
  (`public/js/p3/widget/app/<Service>.js`).
- Always include a `## Overview` heading — that's what the info icon shows.
- Keep real tool names and paper citations intact (e.g. `RASTtk`, `PATtyFams`, `PGFams`, and
  author/journal references). Only brand/UI references (`BV-BRC` → `dxkb`, `bv-brc.org` →
  `dxkb.org`) were rebranded.
- After editing, run `make html` and confirm the section id is generated as expected.

### Docs still needing subject-matter review

The following were authored fresh (no BV-BRC equivalent to adapt) and should be reviewed by
whoever owns each pipeline:

**dxkb-only services** (no reference anywhere — highest need):
- `frustraMPNN_service.md` (FrustraMPNN)
- `stabiliNNator.md` (Protein Stability Prediction — proliNNator / disulfiNNate)
- `stability_prediction_service.md` (ThermoMPNN)
- `structure_sequence_prediction_service.md` (ProteinMPNN)

**BV-BRC services missing from BV-BRC-Docs** (drafted from related pages):
- `comparative_pathway_service.md`
- `genomad.md`
- `mobile_element_detection_service.md`
