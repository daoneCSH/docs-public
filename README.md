# docs-public

This repository is the source of truth for the public DADP documentation site served at `docs.dadp.co.kr`.

## Repository Layout

- `docs/`: published documentation sources
- `mkdocs.yml`: MkDocs site configuration
- `requirements.txt`: MkDocs build dependencies

## Local Build

```bash
python -m pip install -r requirements.txt
python -m mkdocs build
```

The static site is generated into `site/`.

## Cloudflare Pages

This repository is intended to be connected directly to Cloudflare Pages using Git integration.

Recommended Pages settings:

- Production branch: `main`
- Build command: `pip install -r requirements.txt && python -m mkdocs build`
- Build output directory: `site`
- Root directory: leave blank

After the project is created, attach `docs.dadp.co.kr` as a custom domain in the Pages dashboard.
