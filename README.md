# Living Threads Haiti

Static website for Living Threads at `livingthreadshaiti.com`. Source repository: `overvale/living-threads-haiti`, branch `main`; the local folder is named `livingthreadshaiti`.

The main pages are `index.html`, `linens.html` and `ordering.html`, with CSS, `img/` assets and the checked-in order-form PDF. `haitiprojectssample.html` contains an external embedded form; `error.html` is a legacy redirect page. `CNAME` records the custom domain.

## Cloud setup

Editing and serving this site work on Linux or macOS. Provision **Git** and **Python 3** in the environment; Python's standard library is sufficient. No Node/npm packages, credentials, database, or operating-system-specific tools are needed.

From the repository root, the repeatable setup recipe is:

```sh
git status --short
python3 --version
test -f index.html
test -f style.css
```

An existing checkout needs no dependency installation. There is no compilation or generated output: the checked-in HTML, CSS and assets are the site. Edit those files directly.

## Checks

Run this standard-library check from the repository root. It parses every root HTML page and checks the existence of local page, stylesheet, image and document references. It does not contact external websites, check URL fragments, validate HTML/CSS conformance, or replace a visual review.

```sh
python3 - <<'PY'
from html.parser import HTMLParser
from pathlib import Path
from urllib.parse import unquote, urlsplit

root = Path.cwd()
missing = []
checked = 0

class CheckLinks(HTMLParser):
    def __init__(self, page):
        super().__init__()
        self.page = page

    def handle_starttag(self, tag, attrs):
        global checked
        for key, value in attrs:
            if key not in {"href", "src"} or not value:
                continue
            url = urlsplit(value)
            if url.scheme or url.netloc or not url.path:
                continue
            target = (root / unquote(url.path.lstrip("/"))
                      if url.path.startswith("/")
                      else self.page.parent / unquote(url.path))
            if target.is_dir():
                target = target / "index.html"
            checked += 1
            if not target.is_file():
                missing.append(f"{self.page.name}: {value}")

pages = sorted(root.glob("*.html"))
assert pages, "No HTML pages found"
for page in pages:
    CheckLinks(page).feed(page.read_text(encoding="utf-8"))
if missing:
    raise SystemExit("\n".join(missing))
print(f"OK: {len(pages)} HTML pages; {checked} local links/assets exist")
PY
```

Before handing off, review `git diff --check`, `git diff` and `git status --short`. There is no separate automated test suite.

## Preview

```sh
python3 -m http.server 8000 --bind 127.0.0.1
```

Open `http://127.0.0.1:8000/` locally, or use the cloud environment's forwarded port/private preview URL. If its port forwarding requires listening on all interfaces, use `--bind 0.0.0.0` instead. Choose another port if 8000 is already in use. Stop the server with Ctrl-C.

Start the server at the repository root so root-relative asset links work. Review the main pages at desktop and narrow widths. Changes are visible after refreshing the browser; no rebuild is needed.

Check `/`, `/linens.html`, `/ordering.html` and `/living-threads-order-form.pdf`.

## Optional requirements and publication

The stylesheet loads Lato from Google Fonts. Viewing that font and the embedded Google form requires network access; neither service is needed for setup or local link checks. Their current availability must be checked separately when relevant. The static server does not process form submissions or send email.

The local `living-threads-order-form-template.pages` is ignored by Git and does not follow a fresh cloud checkout. Editing that optional source template requires compatible Pages tooling; it is not needed to edit, check or preview the website or its existing PDF.

`.github/workflows/static.yml` publishes the repository through GitHub Pages on pushes to `main` or manual workflow dispatch. Its job uses Ubuntu and uploads the repository directly, with no build step. Local setup, checks and preview do not trigger that workflow. Preserve `CNAME` and leave publication for a separately requested step.
