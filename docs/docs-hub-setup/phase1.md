## Phase 1 — Hub Repo Setup

### Step 1: Create the repo
Go to [github.com/new](https://github.com/new) and create:
- **Repo name:** `docs-hub`
- **Visibility:** Public
- **Initialize with:** a `README.md`

Confirm ✅ before continuing.

***

### Step 2: Clone it locally
```bash
git clone https://github.com/handle-A/docs-hub.git
cd docs-hub
```

***

### Step 3: Install dependencies locally
```bash
pip install mkdocs-material mkdocs-multirepo-plugin mike
```

***

### Step 4: Create the folder structure
```bash
mkdir -p docs
```

***

### Step 5: Create `mkdocs.yml`
```yaml
site_name: Company Docs Hub
site_url: https://handle-A.github.io/docs-hub/

theme:
  name: material
  features:
    - navigation.tabs
    - navigation.top

plugins:
  - search
  - multirepo:
      cleanup: true

nav:
  - Home: index.md

extra:
  version:
    provider: mike
```

***

### Step 6: Create `docs/index.md`
```markdown
# Welcome to Company Docs Hub

This is the central documentation portal.
Team-specific documentation will appear here as sections are onboarded.
```

***

### Step 7: Verify locally
```bash
mkdocs serve
```
Open `http://127.0.0.1:8000` — confirm the site loads with no errors.

***

### Step 8: Push and deploy
```bash
git add .
git commit -m "feat: initial hub setup"
git push origin main

# First deploy to create gh-pages branch
mkdocs gh-deploy
```

***

### Step 9: Enable GitHub Pages
- Go to `handle-A/docs-hub` → **Settings → Pages**
- Source: **Deploy from branch** → `gh-pages` → `/ (root)`
- Save

***

**Confirm these outcomes before we move to Phase 2:**
- [ ] Repo exists on GitHub
- [ ] `mkdocs serve` ran cleanly locally
- [ ] `gh-pages` branch was created after `mkdocs gh-deploy`
- [ ] GitHub Pages is enabled and site is live at `https://handle-A.github.io/docs-hub/`