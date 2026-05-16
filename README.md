# a cup of tea

The personal blog of [Cole McIntosh](https://github.com/colesmcintosh). Short essays on software engineering, AI, and the craft of building things.

Published at [colesmcintosh.github.io](https://colesmcintosh.github.io).

## Local development

Install dependencies and start the Jekyll server:

```sh
bundle install
bundle exec jekyll serve
```

The site will be available at [http://127.0.0.1:4000](http://127.0.0.1:4000) with live reload enabled.

## Adding a post

Create a new markdown file in `_posts/` following the `YYYY-MM-DD-title-slug.md` naming convention, with the following frontmatter:

```yaml
---
layout: post
title: "post title"
date: YYYY-MM-DD 09:00:00 -0500
categories: update
---
```

## Built with

- [Jekyll](https://jekyllrb.com) — static site generator
- [Minima](https://github.com/jekyll/minima) — theme
- GitHub Pages — hosting
