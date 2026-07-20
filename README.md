# repo-factory

Publishing pipeline for an environment that can `git push` to github.com but
cannot reach api.github.com. The `publish` workflow sweeps branches named
`incoming/<repo-name>` and, for each one:

1. creates the repo `<repo-name>` under this account if it does not exist
   (public; description taken from `.repo-meta.json` at the branch root),
2. force-pushes the branch content — minus `.repo-meta.json` — to that repo's
   `main`, keeping the tip commit's message,
3. deletes the `incoming/<repo-name>` branch here.

The sweep runs on every push to `main` (an empty commit is a fine trigger) and
can also be started manually from the Actions tab (`workflow_dispatch`).
Branches from a failed publish are kept and retried on the next sweep.

## Setup (one time)

Add a repository secret named **`REPO_CREATE_TOKEN`** (Settings → Secrets and
variables → Actions): a classic personal access token with `repo` scope.

## Usage

```bash
cd <project>                       # committed working tree
cat > .repo-meta.json <<'EOF'
{"description": "One-sentence repo description."}
EOF
git add .repo-meta.json && git commit --amend --no-edit   # keep meta in the tip commit
git push https://<user>:<token>@github.com/<user>/repo-factory.git HEAD:refs/heads/incoming/<repo-name>
# fire the sweep:
git commit --allow-empty -m "publish <repo-name>"
git push https://<user>:<token>@github.com/<user>/repo-factory.git HEAD:refs/heads/main~0:refs/heads/main 2>/dev/null \
  || true  # (any push to main works — see below for the simple form)
```

Simplest trigger from a clone of this repo:

```bash
git clone https://github.com/<user>/repo-factory.git && cd repo-factory
git commit --allow-empty -m "publish sweep"
git push origin main
```

Within ~30 seconds each pending `incoming/*` branch becomes a published repo.
Re-publishing an existing name is allowed (the create step treats HTTP 422
"already exists" as success) and force-overwrites the target's `main`.
