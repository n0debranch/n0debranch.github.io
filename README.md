# n0debranch — mccorkell.au

A personal learning journal built with the official Chirpy Starter and Jekyll.
This is an ordinary Git repository. Editing, building, and hosting do not require Codex.

## Current status

Configured for https://mccorkell.au with Australia/Brisbane time.
The provisional title and author name are n0debranch. Change them in _config.yml.
Repository: https://github.com/n0debranch/n0debranch.github.io
GitHub Pages uses the included GitHub Actions workflow, with mccorkell.au saved as the custom domain.
The production build and internal link checks passed on GitHub Actions. DNS at VentraIP still needs to point to GitHub Pages; HTTPS will become available after DNS validation.
The instructions below also document how to reproduce the setup independently.

## Publish on GitHub Pages

1. Create an empty repository named n0debranch.github.io under your GitHub account (public for GitHub Free). Do not initialise it with a README.
2. Push this folder using GitHub Desktop or Git. Include the hidden .github directory: it contains the build and deploy workflow.
3. In the repository's Settings > Pages, choose GitHub Actions as the publishing source.
4. Run Build and Deploy from the Actions tab if the initial push happened before Pages was enabled. Future pushes to main or master trigger it automatically.
5. Verify mccorkell.au in your GitHub account's Pages settings, then set mccorkell.au as the repository's custom domain before changing DNS.
6. At your DNS provider, configure the domain using GitHub's current instructions linked below. Keep unrelated email records intact.
7. Enable Enforce HTTPS once GitHub's certificate is ready.

The included CNAME file records the intended domain, but GitHub Actions deployments require the custom domain to be set in Pages settings too.

Official domain instructions:
https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site

## Write an article without Codex

1. Copy _drafts/article-template.md to _posts/YYYY-MM-DD-your-article-title.md, using the publication date.
2. Replace the title, description, categories, tags, and body. Suggested categories: Learning Notes, Projects, Guides.
3. Put screenshots in assets/img/posts/ and link them with Markdown, for example ![Screenshot description](/assets/img/posts/example.png).
4. Commit and push the file. You can also create or edit it directly on GitHub.
5. Watch the Build and Deploy workflow. It builds the site and checks internal links before deployment.

Files in _drafts are not published by the normal build. The draft template is not a published article.

## Preview locally (optional)

Install Ruby 3.4 and Bundler, or use the starter's development container with Docker and VS Code. On Windows, the development container is the upstream recommended route.

    bundle install
    bundle exec jekyll serve

Open http://127.0.0.1:4000. To include drafts, use:

    bundle exec jekyll serve --drafts

Restart the server after changing _config.yml.

## Build and validate

    bundle exec jekyll build
    bundle exec htmlproofer _site --disable-external

The GitHub Actions workflow performs a production build and internal link checks.
A successful workflow is the first full build verification if Ruby is unavailable locally.

## Maintenance and portability

- Site settings: _config.yml
- About page: _tabs/about.md
- Articles: _posts/
- Drafts: _drafts/
- Deployment: .github/workflows/pages-deploy.yml
- Theme dependency: Gemfile

Keep a clone or backup of this repository. Any editor can modify it. GitHub runs publishing independently of your computer or Codex. The generated _site directory can also be hosted by another static web host; you would update the deployment configuration and DNS when moving.

Theme upgrades should be previewed and built before publishing. Review upstream release notes and starter changes when updating Gemfile.

Upstream starter: https://github.com/cotes2020/chirpy-starter
Setup documentation: https://chirpy.cotes.page/posts/getting-started/
The upstream MIT license is retained in LICENSE.
