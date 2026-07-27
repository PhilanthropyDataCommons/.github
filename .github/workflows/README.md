# Shared workflows

## `add-to-project.yml`

A reusable workflow that adds the triggering issue (or pull request) to the
organization's project board. Repositories opt in by calling it from their own
workflow; the org workflow template **Add to project** (Actions → New workflow →
"Add to project") drops in the three-line caller.

### One-time organization setup

1. **Create the `pdc-bot` GitHub App** (Org → Settings → Developer settings →
   GitHub Apps → New GitHub App):
   - Disable the webhook.
   - Permissions:
     - Organization → Projects: **Read & write**
     - Repository → Metadata: **Read** (mandatory)
     - Repository → Issues: **Read**
     - Repository → Pull requests: **Read & write**
   - Installable on this account only, then **Install** on **All repositories**.
   - Generate a private key and note the App ID.

2. **Store the credentials** (Org → Settings → Secrets and variables → Actions):
   - Variable `PDC_BOT_APP_ID` — the App ID.
   - Variable `ADD_TO_PROJECT_URL` — the project URL,
     e.g. `https://github.com/orgs/PhilanthropyDataCommons/projects/<n>`.
   - Secret `PDC_BOT_PRIVATE_KEY` — the contents of the downloaded `.pem`.

   Set each to be available to all repositories (include the private repo).

3. **Allow other repositories to call this workflow** (this `.github` repo →
   Settings → Actions → General → Access): set to
   **Accessible from repositories in the organization**.

A repository's caller workflow does not succeed until all three steps above are
complete and the `v1` tag exists.

### Adding a repository

Add `.github/workflows/add-to-project.yml` to the repository (use the **Add to
project** template):

```yaml
name: Add to project

on:
  issues:
    types: [opened, transferred, reopened]

jobs:
  add-to-project:
    uses: PhilanthropyDataCommons/.github/.github/workflows/add-to-project.yml@v1
    secrets:
      PDC_BOT_PRIVATE_KEY: ${{ secrets.PDC_BOT_PRIVATE_KEY }}
```

To also add pull requests, extend the `on:` block with a `pull_request_target`
trigger (using `pull_request_target` rather than `pull_request` so the workflow
has access to org secrets on contributions from forks).

### Releasing changes

Callers pin to the `v1` tag rather than `main`, so changes on `main` do not
reach any repository until the tag moves. To publish the current state of
`main`:

```sh
git tag -f v1 main
git push -f origin v1
```

Cut a `v2` rather than moving `v1` for a breaking change — a new required secret
or input, or a different trigger contract — and repoint each caller as it is
ready.

Actions used inside the reusable workflow are pinned to exact patch versions and
kept current by Dependabot.
