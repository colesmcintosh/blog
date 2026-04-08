# Cole Blog

Minimal Jekyll blog scaffold for GitHub Pages.

## What's included

- Minimal Jekyll setup for local development
- Default `minima` theme
- Home page and About page
- One starter blog post
- GitHub Actions workflow for GitHub Pages deploys

## Customize

Update these values in `_config.yml`:

- `title`
- `email`
- `url`
- `github_username`

## Run locally

1. Install Ruby and Bundler if needed.
2. Install dependencies:

```bash
bundle install --path vendor/bundle
```

3. Start the site:

```bash
bundle exec jekyll serve
```

4. Open `http://localhost:4000`

## Deploy on GitHub Pages

1. Create a GitHub repository.
2. Push this project to the repository.
3. In GitHub, open `Settings > Pages`.
4. Set the source to `GitHub Actions`.

The included workflow builds and deploys the site automatically.

If this repository is a project site instead of `colesmcintosh.github.io`, set `baseurl` in `_config.yml` to the repo name, for example `/cole-blog`.
