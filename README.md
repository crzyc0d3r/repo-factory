# repo-factory

Push-triggered publisher. Pushing a branch named `incoming/<repo-name>` to this
repository makes the `publish` workflow:

1. create the repo `<repo-name>` under this account if it does not exist
   (public; description taken from `.repo-meta.json` at the branch root),
2. force-push the branch content — minus `.repo-meta.json` — to that repo's `main`,
3. delete the `incoming/<repo-name>` branch here.

This exists so an automated pipeline that can reach `github.com` (git push)
but not `api.github.com` (REST) can still create and publish new repositories:
the API call happens inside GitHub Actions instead.

## Setup (one time)

Add a repository secret named **`REPO_CREATE_TOKEN`** (Settings → Secrets and
variables → Actions): a classic personal access token with `repo` scope.

## Usage

```bash
cd <project>                       # committed working tree, branch main
cat > .repo-meta.json <<'EOF'
{"description": "One-sentence repo description."}
EOF
git add .repo-meta.json && git commit -m "meta"
git push https://<user>:<token>@github.com/<user>/repo-factory.git HEAD:refs/heads/incoming/<repo-name>
```

Within ~30 seconds the target repo exists and contains the branch content.
Re-pushing the same name republishes (the create step treats HTTP 422
"already exists" as success). The commit that lands on the target's `main` is
the pushed tip commit amended to drop `.repo-meta.json`, keeping its message.
