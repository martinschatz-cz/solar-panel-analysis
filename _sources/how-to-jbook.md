# How to build and publish this Jupyter Book (Miniconda → gh-pages)

This document explains a step-by-step procedure to build the repository's Jupyter Book locally using Miniconda and publish the generated static site to the `gh-pages` branch on GitHub. It includes reproducible environment configuration, build commands (PowerShell), and an example GitHub Actions workflow that automatically builds and pushes to `gh-pages`.

--

## 1. Assumptions

- Repository root contains the Jupyter Book sources (`_config.yml`, `chapters/`, etc.).
- You want static files served from the `gh-pages` branch (Pages configured to use `gh-pages` / `/`).
- You have permission to push branches and edit repository Pages settings.

## 2. Create a reproducible conda environment

Create `environment.yml` at the repository root. Example content:

```yaml
name: jbook-solarp
channels:
	- conda-forge
dependencies:
	- python=3.10
	- pip
	- pip:
		- jupyter-book
		- myst-parser
		- sphinx
		- sphinx-book-theme
		- jupyter-cache
		- nbclient
```

Notes:
- Pin `python` to the minor or patch version you prefer (e.g. `python=3.10.12`) for reproducibility.

On Windows PowerShell, create and activate the environment:

```powershell
# From the repository root
conda env create -f environment.yml
conda activate jbook-solarp
```

If you prefer Miniconda installer steps, install Miniconda then run the commands above.

## 3. Build the book locally

With the environment active:

```powershell
# Build HTML site
jb build .

# Output directory: _build/html
# Quick preview (Windows): opens default browser
Start-Process _build/html/index.html
```

Important: your `_config.yml` currently contains:

```yaml
execute:
	execute_notebooks: "off"
```

This ensures notebooks are not re-run during `jb build` and that the existing saved outputs are used.

## 4. Manual publish to `gh-pages` (local)

Option A — use `ghp-import` (recommended, simple):

```powershell
pip install ghp-import
ghp-import -n -p -f _build/html -b gh-pages
```

- `-n` adds a `.nojekyll` file, `-p` pushes, `-f` forces overwrite, `-b` sets branch name.

Option B — use git directly (subtree or orphan branch):

```powershell
# Example: force-push _build/html as gh-pages (overwrites branch)
git checkout --orphan gh-pages
git --work-tree _build/html add --all
git --work-tree _build/html commit -m "Publish book"
git push origin HEAD:gh-pages --force
git checkout -  # return to previous branch
```

After the first push, set GitHub Pages settings:
- Go to `Settings` → `Pages` → Source → select `gh-pages` branch and `/ (root)`.

## 5. Automated publish via GitHub Actions (recommended)

Create `.github/workflows/deploy-gh-pages.yml` with the following example. It uses Miniconda (conda-incubator/setup-miniconda), builds the book, then pushes to `gh-pages` with `peaceiris/actions-gh-pages`.

```yaml
name: Build and deploy Jupyter Book to gh-pages

on:
	push:
		branches:
			- main
	workflow_dispatch:

jobs:
	build-and-deploy:
		runs-on: ubuntu-latest
		steps:
			- uses: actions/checkout@v4

			- name: Set up Miniconda
				uses: conda-incubator/setup-miniconda@v2
				with:
					activate-environment: jbook
					environment-file: environment.yml
					python-version: '3.10'

			- name: Install extras via pip
				run: |
					pip install --no-cache-dir jupyter-book

			- name: Build the book
				run: |
					jb build .

			- name: Deploy to gh-pages
				uses: peaceiris/actions-gh-pages@v3
				with:
					github_token: ${{ secrets.GITHUB_TOKEN }}
					publish_dir: ./_build/html
					publish_branch: gh-pages
```

Notes:
- `environment-file: environment.yml` uses the `environment.yml` you added to create the conda environment in the runner.
- `peaceiris/actions-gh-pages@v3` will create or update the `gh-pages` branch and push the site contents.
- `GITHUB_TOKEN` provided by GitHub Actions has sufficient permission to push to `gh-pages` for typical repository workflows.

## 6. Configure GitHub Pages

After the workflow runs and pushes to `gh-pages`:

- Open the repository on GitHub → `Settings` → `Pages`.
- Set the Source to `gh-pages` branch and `/ (root)` if not already set.
- The Pages URL will appear there once available.

## 7. Optional recommendations

- Pin exact Python patch versions in `environment.yml` and workflow (`'3.10.12'`) for determinism.
- If you do not want notebook outputs included, set `execute_notebooks` to `force` or `auto` depending on your desired behavior — but your current value `off` preserves notebook outputs.
- Consider adding a `Makefile` or a short `scripts/` helper to simplify local builds and publishing commands.

## 8. Troubleshooting

- If Actions fails due to permissions to push, check repository Actions permissions and branch protection rules (adjust as necessary).
- If build fails due to missing packages, add them to `environment.yml` (or `pip` extras) and re-run locally to test.
- If notebooks contain large attachments, consider cleaning outputs or using `jupyter-cache` workflows to manage caching.
