# Dustin Nguyen — personal website

Source for [ducto489.github.io](https://ducto489.github.io/), a personal portfolio focused on physics, machine learning, quantum computing, and scientific computing.

The site is built with Jekyll and is based on the [al-folio](https://github.com/alshedivat/al-folio) theme.

## Content

- `_pages/` — top-level pages such as About, Projects, and CV.
- `_projects/` — long-form project write-ups.
- `assets/img/` — images used by the site and project pages.
- `assets/json/resume.json` — structured CV data.
- `assets/pdf/Minh_Duc___CV.pdf` — downloadable full CV.

## Local development

The CI configuration uses Ruby 3.2.2 and Node.js 20.

```bash
bundle install
npm ci
python -m pip install -r requirements.txt
bundle exec jekyll serve
```

Open `http://localhost:4000` after Jekyll starts.

## Quality checks

```bash
npm run format:check
JEKYLL_ENV=production bundle exec jekyll build
npx --yes purgecss@6.0.0 -c purgecss.config.js
```

The GitHub Actions deployment workflow also runs formatting, a production build, and local-link validation before publishing to `gh-pages`.

## Deployment

Pushes to `master` are deployed automatically after the quality checks pass. Pull requests run the same checks but do not deploy.

## License and theme credit

The site retains the MIT license from the underlying al-folio theme. Site-specific content belongs to the repository owner unless otherwise noted.
