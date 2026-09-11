# Connecting Advana GitLab to Advana Databricks

**Purpose:** Configure Databricks Git integration (Git folders / Repos) to authenticate against
a self-managed Advana GitLab instance, so notebooks and code can be pulled from / pushed to
GitLab directly within a Databricks workspace.

**Audience:** Anyone on the team setting up Databricks Git integration against Advana GitLab
for the first time, or troubleshooting an existing connection.

**Last updated:** 2026-09-10

---

## Prerequisites

- An active Advana GitLab account with at least Developer-level access to the target repo.
- An active Advana Databricks workspace account.
- Know your Advana GitLab server's base URL (e.g. `https://<your-advana-gitlab-host>`).

---

## Steps

### 1. Generate a GitLab personal access token (PAT)

In Advana GitLab: **User Settings → Access Tokens**.

- Scopes: `api`, `read_repository`, `write_repository`
- Set an expiration date and track it for rotation
- **Important:** Databricks Git integration only supports **user-level** personal access
  tokens. Project- or group-level access tokens are not supported and will fail silently or
  reject pushes even when pulls succeed.

### 2. Note your GitLab server URL

Copy the base URL of the Advana GitLab instance. You'll need this if a Databricks workspace
admin has restricted which Git hosts are reachable (see step 6).

### 3. Open Git integration settings in Databricks

In the Advana Databricks workspace: click your username (top right) → **Settings** →
**Linked accounts** (older UI: "Git Integration"). Click **Add Git credential**.

### 4. Select the correct GitLab provider type

- **GitLab** — if the instance behaves like gitlab.com
- **GitLab Self-Managed** (a.k.a. GitLab Enterprise Edition in some UIs) — for a self-hosted
  instance with a custom domain, which Advana's almost certainly is

Selecting the wrong provider type is the most common cause of a connection that fails without
a clear error.

### 5. Enter credentials

- Git provider username/email: your Advana GitLab account email
- Token: the PAT from step 1

Save.

### 6. Check the workspace Git URL allow list

Some Databricks admin consoles restrict which Git remote URLs can be used:
**Admin Console → Repos → Git URL Allow List**.

If the Advana GitLab server URL isn't on that list, ask the Databricks workspace admin to add
it. Being a GitLab admin doesn't necessarily mean you're a Databricks workspace admin, so this
is worth checking *before* attempting a clone, not after it fails.

### 7. Clone the repo into a Databricks Git folder

**Workspace → Repos** (newer UI: "Git folders") → **Add Repo** → paste the HTTPS clone URL of
the Advana GitLab repository. Databricks will authenticate using the credential from step 5.

---

## Known limitations / gotchas

| Issue | Cause | Fix |
|---|---|---|
| Only one Git provider works at a time | Databricks supports a single active Git credential per user per workspace | Swap credentials in Linked accounts when switching providers/accounts |
| Clone/pull hangs or times out (not an auth error) | Databricks and GitLab may sit in different network enclaves/impact levels | Escalate to the Advana platform team as a network issue, not a credentials issue |
| `git pull` works but `push` fails | Token was generated as a project-level access token instead of a personal (user-level) one | Regenerate the token from **User Settings**, not a project's settings |

---

## Troubleshooting checklist

1. Confirm the token was created under **your personal GitLab user settings**, not a project.
2. Confirm the provider type selected in Databricks matches the instance (Self-Managed vs. standard).
3. Confirm the GitLab server URL is on the Databricks Git URL allow list (if enabled).
4. If it times out rather than rejecting auth, treat it as a network/firewall issue.
