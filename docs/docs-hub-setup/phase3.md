## Phase 3 — Complete (Consolidated)

### Step 1: Create spoke repo (as handle-B)
- Go to [github.com/new](https://github.com/new) → create `team-nkp`, Public, initialize with README

### Step 2: Clone and set up spoke structure (as handle-B)
```bash
git clone https://github.com/handle-B/team-nkp.git
cd team-nkp
mkdir -p docs .github/workflows
```

### Step 3: Create spoke `mkdocs.yml`
```yaml
site_name: NKP Documentation
docs_dir: docs

theme:
  name: material

nav:
  - Home: index.md
```
> ⚠️ Explicit `nav` is required — the hub's multirepo plugin cannot auto-resolve spoke pages without it.

### Step 4: Create `docs/index.md`
```markdown
# NKP Documentation

Welcome to the NKP team documentation section.
```

### Step 5: Create `.github/workflows/publish.yml`
```yaml
name: Publish to Docs Hub

on:
  push:
    branches: [main]
    tags:
      - "v*"

jobs:
  notify-hub:
    runs-on: ubuntu-latest
    steps:
      - name: Determine version and ref
        id: version
        run: |
          if [[ "${GITHUB_REF}" == refs/tags/* ]]; then
            echo "version=${GITHUB_REF#refs/tags/}" >> $GITHUB_OUTPUT
            echo "ref=${GITHUB_REF#refs/tags/}" >> $GITHUB_OUTPUT
          else
            echo "version=latest" >> $GITHUB_OUTPUT
            echo "ref=main" >> $GITHUB_OUTPUT
          fi

      - name: Trigger hub deploy
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.HUB_DEPLOY_TOKEN }}
          script: |
            await github.rest.actions.createWorkflowDispatch({
              owner: 'handle-A',
              repo: 'docs-hub',
              workflow_id: 'deploy.yml',
              ref: 'main',
              inputs: {
                team: 'NKP',
                version: '${{ steps.version.outputs.version }}'
              }
            });
```

### Step 6: Generate PAT (as handle-A)
- **handle-A** → Settings → Developer Settings → Fine-grained tokens → Generate new token
- Repository access: `docs-hub` only
- Permissions: Actions → **Read and Write**
- Copy token immediately

### Step 7: Store PAT in spoke (as handle-B)
- `handle-B/team-nkp` → Settings → Secrets and variables → Actions → New secret
- **Name:** `HUB_DEPLOY_TOKEN` | **Value:** PAT from Step 6

### Step 8: Push spoke
```bash
git add .
git commit -m "feat: initial NKP docs setup"
git push origin main
```

### Step 9: Wire spoke into hub `mkdocs.yml` (as handle-A, in `docs-hub`)
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
  - NKP: "!import https://github.com/handle-B/team-nkp?branch=main"

extra:
  version:
    provider: mike
```

### Step 10: Verify locally then push hub
```bash
# In docs-hub
mkdocs serve          # Confirm NKP tab shows rendered content
git add mkdocs.yml
git commit -m "feat: wire NKP spoke into hub nav"
git push origin main  # Triggers hub CI automatically
```

***

**Outcomes to verify:**

- [ ] Spoke `publish.yml` triggers hub `deploy.yml` on every push to `main` ✅
- [ ] `mkdocs serve` locally shows NKP tab with rendered spoke content ✅
- [ ] Live GitHub Pages site shows NKP section correctly ✅

***

Ready for Phase 4 (versioning with `mike`) whenever you are.