## Phase 2 — Hub CI Workflow

### Step 1: Create the workflow file
```bash
mkdir -p .github/workflows
```

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy Docs Hub

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      team:
        description: "Team section (e.g. NKP, NC2)"
        required: true
        default: "all"
      version:
        description: "Version to deploy (e.g. v1.0) or 'latest'"
        required: true
        default: "latest"

permissions:
  contents: write

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-python@v5
        with:
          python-version: "3.x"

      - name: Install dependencies
        run: pip install mkdocs-material mkdocs-multirepo-plugin mike

      - name: Configure git for mike
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

      - name: Deploy hub
        run: mkdocs gh-deploy --force
```

***

### Step 2: Push the workflow
```bash
git add .github/workflows/deploy.yml
git commit -m "feat: add hub CI deploy workflow"
git push origin main
```

***

### Step 3: Verify auto-trigger
- The push above will **automatically trigger** `deploy.yml` via the `push` to `main` rule
- Go to `handle-A/docs-hub` → **Actions** tab
- Confirm the workflow run completes with a ✅

***

### Step 4: Test manual trigger
- In Actions tab → select **Deploy Docs Hub** → **Run workflow**
- Leave inputs as defaults (`all`, `latest`)
- Confirm it completes ✅ and site still loads at `https://handle-A.github.io/docs-hub/`

***

**Confirm these outcomes before Phase 3:**

- [ ] `deploy.yml` pushed and visible under Actions tab
- [ ] Auto-triggered run on push completed ✅
- [ ] Manual `workflow_dispatch` test completed ✅
- [ ] Site still live and unchanged at GitHub Pages URL