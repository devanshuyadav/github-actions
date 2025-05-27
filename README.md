# github-actions

### Manual Trigger

- Generate Fine-grain token first. Read and write access to actions and workflows
- use 'export GITHUB_TOKEN="..."' in terminal to set this token
- To check set env variable: env | grep GITHUB
- To remove set env variables: unset GITHUB_TOKEN
- Command to trigger workflow:
  gh workflow run manual.yml -f name="Devanshu Yadav" -f greeting=Bonjour -f age=22 -f data="$(cat mydata)"
  where mydata is a file which contains base64 encoded text of the message to be printed.

### Webhook

## 🚀 Triggering GitHub Actions with `repository_dispatch`

This repository demonstrates how to use GitHub’s `repository_dispatch` event to **trigger workflows via external requests** — either within the same repo or across different repos.

---

### 📌 What is `repository_dispatch`?

The `repository_dispatch` event allows you to programmatically trigger a GitHub Actions workflow with optional custom data (`client_payload`). It’s especially useful when:

- Triggering workflows from outside GitHub (scripts, APIs, cron jobs, etc.)
- Passing custom payloads (e.g., build metadata)
- Coordinating workflows across repositories

---

## 🧩 Supported Use Cases

### ✅ 1. Single-Repo Trigger

Trigger a workflow in the **same repository** using `curl`.

#### Workflow File (`.github/workflows/webhook.yml`):

```yaml
name: 'Single Repo Webhook Listener'

on:
  repository_dispatch:
    types: [webhook]

jobs:
  run-on-dispatch:
    runs-on: ubuntu-latest
    steps:
      - name: Echo payload
        run: |
          echo "Event: $GITHUB_EVENT_NAME"
          echo "Payload key: ${{ github.event.client_payload.key }}"
```

#### Curl Command:

```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: token YOUR_PERSONAL_ACCESS_TOKEN" \
  -d '{"event_type": "webhook", "client_payload": {"key": "hello-world"}}' \
  https://api.github.com/repos/YOUR_USERNAME/YOUR_REPO/dispatches
```

🧪 Result: The workflow is triggered in the same repository.

---

### 🔁 2. Cross-Repo Trigger

Use one repository (e.g., `frontend`) to **trigger a workflow in another repo** (e.g., `deployment`) — ideal for CI/CD pipelines.

#### ✅ In **Repository A** (triggering repo)

Send the dispatch event using a script or action:

```bash
curl -X POST \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: token YOUR_PERSONAL_ACCESS_TOKEN" \
  -d '{"event_type": "deploy", "client_payload": {"env": "production"}}' \
  https://api.github.com/repos/YOUR_USERNAME/REPO_B/dispatches
```

#### ✅ In **Repository B** (receiving repo)

Add this workflow to `.github/workflows/on-deploy.yml`:

```yaml
name: 'Deployment Trigger Listener'

on:
  repository_dispatch:
    types: [deploy]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Show environment
        run: |
          echo "Triggered deployment for environment: ${{ github.event.client_payload.env }}"
```

🧪 Result: When the curl is run in Repo A, Repo B's workflow is triggered and processes the payload.

---

### 🔐 Token Requirements

The token used in the `Authorization` header must:

- Belong to a user with access to the target repository
- Have the following scopes:

  - `repo` (for private repos)
  - `public_repo` (for public repos)

---

### ✅ Best Practices

- Keep `event_type` simple and meaningful (`build`, `webhook`, `deploy`, etc.)
- Include only necessary data in `client_payload`
- Use dispatches for decoupled, reusable workflows
- For cross-repo triggers, use secrets to store PATs securely
