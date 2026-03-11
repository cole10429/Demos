# Demos

This repository is a collection of static HTML pages.

## Yes — this is doable

GitHub Pages can turn this repository into a live website.

That gives you:

- one main index page for all demos: `https://cole10429.github.io/Demos/`
- one direct link for each individual demo page, such as `https://cole10429.github.io/Demos/crowbot/`

## Exact steps to turn this on in GitHub

Do this one time:

1. Open this repository on GitHub
2. Click the **Settings** tab at the top of the repository
3. In the left sidebar, click **Pages**
4. In the **Build and deployment** section, set **Source** to **GitHub Actions**
5. Wait a moment for GitHub to save that setting
6. Click the **Actions** tab
7. Open the workflow named **Deploy static site to Pages**
8. Click **Run workflow**
9. Choose the `main` branch
10. Click the green **Run workflow** button
11. Wait for the workflow to finish
12. Open your live site at `https://cole10429.github.io/Demos/`

If the workflow has already run successfully once, future pushes to `main` will redeploy the site automatically.

## How to use it after that

Once GitHub Pages is enabled, you will have:

- a main index page that lists all demos
- a direct URL for each demo that you can send to a client

For this repository, the live URLs are:

- Index: `https://cole10429.github.io/Demos/`
- ClickUp demo: `https://cole10429.github.io/Demos/clickup/`
- Crowbot Fleet Explorer: `https://cole10429.github.io/Demos/crowbot/`
- The Data Pipe: `https://cole10429.github.io/Demos/data-pipe/`
- The Data Pipe Pitch: `https://cole10429.github.io/Demos/data-pipe-pitch/`
- Results Test Page: `https://cole10429.github.io/Demos/results/test-page.html`

If you fork this repository, replace `cole10429` with your GitHub username or organization in the URLs above.

## How to add a new demo later

When you create a new demo, do these two things:

1. Add the new HTML page to this repository
   - example: `/home/runner/work/Demos/Demos/my-new-demo/index.html`
2. Add a link to it inside `/home/runner/work/Demos/Demos/index.html`

Example link:

```html
<li><a href="./my-new-demo/index.html">My New Demo</a></li>
```

After that, when the site deploys again:

- the main index will show the new demo
- the new demo will also have its own direct link at `https://cole10429.github.io/Demos/my-new-demo/`

## Local preview

Because these are static pages, you can also preview them locally with a simple web server:

```bash
cd /path/to/Demos
python3 -m http.server 8000
```

Then open `http://localhost:8000/` in your browser.
