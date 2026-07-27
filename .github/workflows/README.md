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
   - Generate a private key and note the Client ID.

   The Projects permission lives under **Organization permissions**, not the
   identically named one under Repository permissions — an org project board is
   unreachable with only the repository grant.

   Creating the app does not install it. Install it explicitly, and re-approve
   the installation whenever the app's permissions change, or the installation
   keeps the permissions it was created with.

2. **Store the credentials** (Org → Settings → Secrets and variables → Actions):
   - Variable `PDC_BOT_CLIENT_ID` — the Client ID (`Iv23...`), not the App ID.
   - Variable `ADD_TO_PROJECT_URL` — the project URL,
     e.g. `https://github.com/orgs/PhilanthropyDataCommons/projects/<n>`.
   - Secret `PDC_BOT_PRIVATE_KEY` — the contents of the downloaded `.pem`.

   Set each to be available to all repositories (include the private repo).

3. **Allow other repositories to call this workflow** (this `.github` repo →
   Settings → Actions → General → Access): set to
   **Accessible from repositories in the organization**.

A repository's caller workflow does not succeed until all three steps above are
complete and the `v1` tag exists. To check what the installation was actually
granted:

```sh
gh api /orgs/PhilanthropyDataCommons/installations \
  --jq '.installations[] | select(.app_slug=="pdc-bot") | .permissions'
```

`organization_projects: write` must be present.

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
git fetch origin
git tag -f v1 origin/main
git push -f origin v1
```

Tag `origin/main` rather than `main` so a stale local checkout cannot publish
the wrong commit under a force-pushed tag.

Do not attach a GitHub release to `v1`. Releases can be marked immutable, which
would permanently prevent the tag from moving.

Cut a `v2` rather than moving `v1` for a breaking change — a new required secret
or input, or a different trigger contract — and update each caller as it is
ready.

Actions used inside the reusable workflow are pinned to exact patch versions and
kept current by Dependabot.
