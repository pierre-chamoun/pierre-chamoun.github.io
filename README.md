# Pierre Chamoun - Academic Portfolio

This is the source for [pierre-chamoun.github.io](https://pierre-chamoun.github.io/), my personal academic portfolio site built with [Hugo](https://gohugo.io/). It includes sections for an About page, Publications, News, Blog, and a downloadable CV.

## Structure

- `hugo.yaml` — site configuration (title, menu, theme params)
- `content/` — Markdown content for pages/posts
- `layouts/` — custom HTML templates
- `archetypes/` — templates for new content files
- `data/` — site data files
- `static/` — static assets (images, etc.)
- `assets/` — assets processed by Hugo's pipeline
- `cv/` — CV source/output

## Running locally

1. [Install Hugo](https://gohugo.io/installation/) (extended version recommended).
2. Clone the repo and start the dev server:

   ```bash
   git clone https://github.com/pierre-chamoun/pierre-chamoun.github.io.git
   cd pierre-chamoun.github.io
   hugo server
   ```

3. Open `http://localhost:1313` in your browser.

## Make it your own

Feel free to clone or fork this repo as a starting point for your own academic/portfolio website. To customize it:

1. Update `hugo.yaml` with your own name, tagline, institution, social links, and accent color.
2. Replace the content in `content/` with your own pages, publications, news, and blog posts.
3. Swap the avatar image in `static/img/`, and edit `cv/main.tex` (and `cv/sections/`) with your own CV.
4. Push to GitHub. The included Actions workflow (`.github/workflows/deploy.yml`) builds and deploys the site automatically.

