# PR Submit

A GitHub App that issues short-lived, repository-scoped tokens to approved workflows that create branches and pull requests.

## Use PR Submit

Install the [PR Submit GitHub App](https://github.com/apps/pr-submit) in the repositories where it should be available.

Add `.github/pr-submit.yml` listing the workflows allowed to request tokens:

```yaml
workflows:
  - .github/workflows/prepare-release.yml
  - .github/workflows/bump-pre-commit-hooks.yml
```

The configuration and the allowed workflows must already be merged into the repository's default branch before they can request a token.

Give the job permission to request a GitHub OIDC token:

```yaml
permissions: {}

jobs:
  prepare-release:
    permissions:
      contents: read
      id-token: write
```

Request a token:

```yaml
- name: Get PR Submit token
  id: pr-submit
  uses: tiangolo/pr-submit@0.0.1
```

Use the token to push the branch and create the pull request:

```yaml
- name: Create pull request
  env:
    GH_TOKEN: ${{ steps.pr-submit.outputs.token }}
  run: |
    branch="example-${GITHUB_RUN_ID}-${GITHUB_RUN_ATTEMPT}"
    git config user.name "pr-submit[bot]"
    git config user.email "pr-submit[bot]@users.noreply.github.com"
    git switch -c "$branch"
    git add -A
    git commit -m "Example change"
    git remote set-url origin "https://x-access-token:${GH_TOKEN}@github.com/${GITHUB_REPOSITORY}.git"
    git push --set-upstream origin "$branch"
    gh pr create --base "$GITHUB_REF_NAME" --head "$branch" --title "Example change" --body "Example change."
```

## Security

PR Submit accepts GitHub OIDC tokens only from `schedule` and `workflow_dispatch` workflows running on the repository's default branch. The requesting workflow must be listed in `.github/pr-submit.yml` and must match the version currently on the default branch. Manually triggered workflows must be requested by an actor who currently has write permission.

The returned GitHub App installation token has `contents: write` and `pull_requests: write` permissions and is scoped to the repository that requested it.

Do not add the GitHub App to branch protection or ruleset bypass lists.

## Deploy your own

Create a GitHub App with read and write access to repository contents and pull requests, then generate a private key. It does not need webhooks, user authorization, a client secret, or any other repository permissions.

Deploy this FastAPI app, for example to [FastAPI Cloud](https://fastapicloud.com), and set:

- `GITHUB_CLIENT_ID`: the GitHub App client ID.
- `GITHUB_APP_PRIVATE_KEY`: the GitHub App private key.
- `OIDC_AUDIENCE`: the public URL of the deployed app.

Pass the deployment URL to the Action:

```yaml
with:
  url: https://pr-submit.example.com
```

## License

This project is licensed under the terms of the MIT license.
