## Phase 3 — First Spoke (handle-B/team-nkp)

### Step 1: Create the spoke repo
Go to [github.com/new](https://github.com/new) logged in as **handle-B** and create:
- **Repo name:** `team-nkp`
- **Visibility:** Public
- **Initialize with:** a `README.md`

***

### Step 2: Clone and set up structure (as handle-B)
```bash
git clone https://github.com/handle-B/team-nkp.git
cd team-nkp
mkdir -p docs
```

***

### Step 3: Create `mkdocs.yml`
```yaml
site_name: NKP Documentation
docs_dir: docs

theme:
  name: material
```

***

### Step 4: Create `docs/index.md`
```markdown
# NKP Documentation

Welcome to the NKP team documentation section.
```

***

### Step 5: Create the publish workflow
```bash
mkdir -p .github/workflows
```

Create `.github/workflows/publish.yml`:

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

> ⚠️ Replace `handle-A` with your actual GitHub username for the hub.

***

### Step 6: Create the PAT (as handle-A)
- Go to **handle-A** GitHub account → **Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens → Generate new token**
- **Resource owner:** handle-A
- **Repository access:** Only `docs-hub`
- **Permissions:** Actions → **Read and Write**
- Copy the token immediately

***

### Step 7: Store PAT in spoke repo (as handle-B)
- Go to `handle-B/team-nkp` → **Settings → Secrets and variables → Actions → New repository secret**
- **Name:** `HUB_DEPLOY_TOKEN`
- **Value:** paste the PAT from Step 6

***

### Step 8: Push spoke content
```bash
git add .
git commit -m "feat: initial NKP docs setup"
git push origin main
```

***

### Step 9: Watch the chain reaction
- Go to `handle-B/team-nkp` → **Actions** → confirm `publish.yml` triggered ✅
- Go to `handle-A/docs-hub` → **Actions** → confirm `deploy.yml` was triggered by the spoke ✅

***

**Confirm these outcomes before Phase 4:**
- [ ] `team-nkp` repo exists and has `docs/index.md` and `mkdocs.yml`
- [ ] `publish.yml` visible in Actions tab on spoke repo
- [ ] `HUB_DEPLOY_TOKEN` secret stored in `team-nkp`
- [ ] Push to `main` triggered spoke workflow ✅
- [ ] Hub `deploy.yml` was triggered automatically from spoke ✅
- [ ] Hub site still live at GitHub Pages URL