# Drew Herren's Personal Website

A personal academic website built with [Quarto](https://quarto.org).

## Local Development

1. Install [Quarto](https://quarto.org/docs/get-started/)
2. Run `quarto preview` to start a local server
3. Edit `.qmd` files and see changes live

## Deployment

Push to the `master` branch to trigger GitHub Pages deployment, or run:

```bash
quarto publish gh-pages
```

## Structure

- `index.qmd` - Homepage
- `publications.qmd` - Research publications
- `portfolio.qmd` - Project portfolio
- `talks.qmd` - Conference talks
- `teaching.qmd` - Teaching history
- `cv.qmd` - Curriculum vitae
- `posts/` - Blog posts

## Note

This used to be a jekyll-based website, forked from https://github.com/academicpages/academicpages.github.io. I have removed the fork dependency but kept the commit history, which explains the large number of contributors from the former jekyll project.
