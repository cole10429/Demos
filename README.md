# Demos

This repository is a collection of static HTML pages.

## View the site in a browser from GitHub

The simplest way to open `main/index.html` or any individual page from GitHub is to publish the repository with **GitHub Pages**.

1. In GitHub, open **Settings** → **Pages**
2. Under **Build and deployment**, set **Source** to **GitHub Actions**
3. Push to `main` or run the **Deploy static site to Pages** workflow manually

Once GitHub Pages is enabled, the live URLs for this repository are:

- Index: `https://cole10429.github.io/Demos/`
- ClickUp demo: `https://cole10429.github.io/Demos/clickup/`
- Crowbot Fleet Explorer: `https://cole10429.github.io/Demos/crowbot/`
- The Data Pipe: `https://cole10429.github.io/Demos/data-pipe/`
- The Data Pipe Pitch: `https://cole10429.github.io/Demos/data-pipe-pitch/`
- Results Test Page: `https://cole10429.github.io/Demos/results/test-page.html`

If you fork this repository, replace `cole10429` with your GitHub username or organization in the URLs above.

## Local preview

Because these are static pages, you can also preview them locally with a simple web server:

```bash
cd /path/to/Demos
python3 -m http.server 8000
```

Then open `http://localhost:8000/` in your browser.
