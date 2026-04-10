# GitHub Workflows

This directory contains GitHub Actions workflows for automating various tasks in the demo-accounts project.

## Available Workflows

### 🔄 Sync Postman Spec with GitHub File

**File:** `sync-postman-spec.yml`

**Purpose:** Synchronizes API specifications between Postman and your GitHub repository, supporting bidirectional sync.

---

## Sync Postman Spec Workflow

### Overview

This workflow allows you to keep your API specifications in sync between Postman and your GitHub repository. It supports three sync modes:

1. **Postman → Repository** - Update repo file from Postman spec
2. **Repository → Postman** - Update Postman spec from repo file
3. **Both** - Check both directions and sync as needed

### Prerequisites

#### 1. Create Postman API Key

1. Go to [Postman Account Settings](https://go.postman.co/settings/me/api-keys)
2. Click **Generate API Key**
3. Name it (e.g., "GitHub Actions Sync")
4. Copy the API key

#### 2. Add API Key to GitHub Secrets

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Name: `POSTMAN_API_KEY`
5. Value: Paste your Postman API key
6. Click **Add secret**

#### 3. Get Your Postman Spec ID

**Method 1: From Postman App**
1. Open your spec in Postman
2. Click the three dots (•••) menu
3. Select **View Info**
4. Copy the **Spec ID** (UID format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)

**Method 2: From Postman API**
```bash
curl -X GET "https://api.postman.com/specs" \
  -H "X-Api-Key: YOUR_POSTMAN_API_KEY" \
  -H "Accept: application/vnd.api.v10+json"
```

### How to Use

#### Option 1: Via GitHub UI (Recommended)

1. **Navigate to Actions Tab**
   - Go to your repository on GitHub
   - Click the **Actions** tab

2. **Select Workflow**
   - Find "Sync Postman Spec with GitHub File"
   - Click on it

3. **Run Workflow**
   - Click **Run workflow** button
   - Fill in the parameters:
     - **Spec ID:** Your Postman spec UID
     - **File Path:** Path to file in repo (e.g., `postman/schemas/accounts.yaml`)
     - **Sync Direction:** Choose one:
       - `repo-to-postman` - Update Postman from repo
       - `postman-to-repo` - Update repo from Postman
       - `both` - Check and sync both ways
     - **Create PR:** Check if you want a Pull Request (recommended)
   - Click **Run workflow**

4. **Review Results**
   - Wait for workflow to complete
   - Check the summary for sync results
   - If PR was created, review and merge it

#### Option 2: Via GitHub CLI

```bash
# Sync from Postman to Repository
gh workflow run sync-postman-spec.yml \
  -f spec_id="your-spec-id-here" \
  -f file_path="postman/schemas/accounts.yaml" \
  -f sync_direction="postman-to-repo" \
  -f create_pr=true

# Sync from Repository to Postman
gh workflow run sync-postman-spec.yml \
  -f spec_id="your-spec-id-here" \
  -f file_path="postman/schemas/accounts.yaml" \
  -f sync_direction="repo-to-postman" \
  -f create_pr=false

# Bi-directional sync
gh workflow run sync-postman-spec.yml \
  -f spec_id="your-spec-id-here" \
  -f file_path="postman/schemas/accounts.yaml" \
  -f sync_direction="both" \
  -f create_pr=true
```

#### Option 3: Via API

```bash
curl -X POST \
  "https://api.github.com/repos/YOUR_USERNAME/demo-accounts/actions/workflows/sync-postman-spec.yml/dispatches" \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -H "Accept: application/vnd.api.v10+json" \
  -d '{
    "ref": "main",
    "inputs": {
      "spec_id": "your-spec-id-here",
      "file_path": "postman/schemas/accounts.yaml",
      "sync_direction": "both",
      "create_pr": "true"
    }
  }'
```

### Workflow Parameters

| Parameter | Required | Type | Description | Default |
|-----------|----------|------|-------------|---------|
| `spec_id` | Yes | String | Postman Spec UID | - |
| `file_path` | Yes | String | Path to file in repository | - |
| `sync_direction` | Yes | Choice | Direction of sync | `both` |
| `create_pr` | No | Boolean | Create PR for repo changes | `true` |
| `debug` | No | Boolean | Enable debug mode (saves artifacts) | `false` |

#### Sync Direction Options

- **`repo-to-postman`**
  - Reads file from GitHub repository
  - Updates Postman spec with file content
  - No PR created (changes in Postman only)

- **`postman-to-repo`**
  - Fetches spec from Postman
  - Updates file in repository
  - Creates PR if `create_pr` is true

- **`both`**
  - Compares both sources
  - Syncs in both directions if differences found
  - Useful for keeping everything in sync

#### Debug Mode

- **`debug: false`** (default)
  - Normal operation
  - No artifacts created
  - Saves storage space
  - Use for regular syncs

- **`debug: true`**
  - Debug mode enabled
  - Saves artifacts with stringified content, payload, and API response
  - Artifacts available even if sync fails
  - Use when troubleshooting issues

**When to enable debug mode:**
- ✅ Sync is failing and you need to see what was sent to Postman
- ✅ Need to verify stringification is correct
- ✅ Want to compare local vs workflow behavior
- ✅ Debugging a 404, 400, or other API error
- ❌ Regular syncs that work fine (wastes storage)

### Use Cases

#### 1. Update Repository from Postman (After editing spec in Postman)

```yaml
Inputs:
  spec_id: "abc123-def456-ghi789"
  file_path: "postman/schemas/accounts-with-schema.yaml"
  sync_direction: "postman-to-repo"
  create_pr: true
```

**Result:** Creates a PR with the updated spec from Postman

#### 2. Update Postman from Repository (After editing file in repo)

```yaml
Inputs:
  spec_id: "abc123-def456-ghi789"
  file_path: "postman/schemas/accounts-with-schema.yaml"
  sync_direction: "repo-to-postman"
  create_pr: false
```

**Result:** Updates the spec in Postman directly

#### 3. Keep Everything in Sync

```yaml
Inputs:
  spec_id: "abc123-def456-ghi789"
  file_path: "postman/schemas/accounts-with-schema.yaml"
  sync_direction: "both"
  create_pr: true
```

**Result:** Syncs in both directions, creates PR if repo changes

### Workflow Steps

The workflow performs the following steps:

1. **Checkout Repository** - Gets the latest code
2. **Setup Node.js** - Prepares the environment
3. **Validate Inputs** - Checks that parameters are valid and validates file type
4. **Get Postman Spec** - Fetches spec metadata from Postman
5. **Get Spec Definition** - Retrieves the actual spec content
6. **Compare and Sync (Postman → Repo)** - If needed, updates repo file
7. **Prepare Sync Payload (Repo → Postman)** - Stringifies content and creates JSON payload
8. **Upload Sync Artifacts** - Saves artifacts if debug mode enabled (optional)
9. **Sync to Postman API** - Sends the prepared payload to Postman using PATCH
10. **Create Pull Request** - Creates PR for repo changes (optional)
11. **Commit Changes** - Commits directly if no PR requested
12. **Summary** - Provides workflow summary with artifact information (if debug enabled)

### How Content Stringification Works

When syncing from repository to Postman, the workflow properly stringifies the file content for JSON transmission:

**Process:**
1. **Read File** - Reads the raw YAML/JSON content
2. **Stringify** - Uses `jq -Rs .` to convert to JSON string
   - Replaces newlines with `\n`
   - Escapes quotes with `\"`
   - Escapes backslashes with `\\`
   - Handles special characters properly
3. **Create Payload** - Wraps in proper JSON structure: `{"content": "..."}`
4. **Validate** - Verifies stringification was successful
5. **Save Artifacts** - Saves all content to artifacts **BEFORE** API call
6. **Send to API** - Transmits to Postman API using PATCH method
7. **Update Artifacts** - Adds API response to artifacts (even on failure)

**HTTP Method:**
- Uses `PATCH` to update spec file content (partial update)
- Endpoint: `PATCH /specs/{specId}/files/{filePath}`
- Content-Type: `application/json`

**Example Transformation:**
```yaml
# Original YAML file
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
```

Becomes:
```json
{
  "content": "openapi: 3.0.0\ninfo:\n  title: My API\n  version: 1.0.0\n"
}
```

This ensures the Postman API receives properly formatted content that preserves all formatting, indentation, and special characters.

### Workflow Artifacts

When syncing from repository to Postman with **debug mode enabled** (`debug: true`), the workflow saves debugging artifacts. This helps troubleshoot issues without cluttering your storage with artifacts from successful runs.

**Artifact Name:** `postman-sync-artifacts-{run-id}`

**Retention:** 7 days

**When Artifacts are Created:**
- ✅ Only when `debug: true` is set
- ✅ Only for repo-to-postman or both sync directions
- ✅ Saved before the API call (available even if sync fails)

**How to Enable:**
```bash
# Via GitHub CLI
gh workflow run sync-postman-spec.yml \
  -f spec_id="your-spec-id" \
  -f file_path="postman/schemas/accounts.yaml" \
  -f sync_direction="repo-to-postman" \
  -f debug=true
```

**Via GitHub UI:**
1. Go to Actions → "Sync Postman Spec with GitHub File"
2. Click "Run workflow"
3. Check the "Enable debug mode" checkbox
4. Fill in other parameters
5. Run workflow

**Files Included:**

1. **`stringified-content.txt`** ✅ Saved BEFORE API call
   - The raw stringified file content as a JSON string
   - Shows exactly what was generated by `jq -Rs .`
   - Example: `"openapi: 3.0.0\ninfo:\n  title: My API\n..."`

2. **`json-payload.json`** ✅ Saved BEFORE API call
   - The complete JSON payload sent to Postman API
   - Includes the `content` field with the stringified spec
   - Example: `{"content": "openapi: 3.0.0\ninfo:..."}`

3. **`source-file-path.txt`** ✅ Saved BEFORE API call
   - The path to the source file that was synced
   - Example: `postman/schemas/accounts-with-schema.yaml`

4. **`spec-id.txt`** ✅ Saved BEFORE API call
   - The Postman spec ID that was updated
   - Example: `abc123-def456-ghi789`

5. **`metadata.txt`** ✅ Saved BEFORE API call, updated AFTER
   - Sync metadata including:
     - `root_file` - The Postman spec root file path
     - `sync_timestamp` - When the sync was initiated (UTC)
     - `needs_sync` - Whether sync was needed (true/false)
     - `http_status` - HTTP response code from Postman API (added after API call)
     - `sync_status` - success or failed (added after API call)
   - Example:
     ```
     root_file=index.yaml
     sync_timestamp=2025-01-07T14:30:45Z
     needs_sync=true
     http_status=200
     sync_status=success
     ```

6. **`api-response.json`** ⚠️ Saved AFTER API call (only if API was called)
   - The raw response from Postman API
   - Useful for debugging errors
   - Contains error details if sync failed

**How to Download Artifacts:**

**Via GitHub UI:**
1. Go to the workflow run page
2. Scroll down to "Artifacts" section
3. Click on `postman-sync-artifacts-{run-id}` to download

**Via GitHub CLI:**
```bash
# List artifacts for a workflow run
gh run view {run-id} --json artifacts

# Download artifacts
gh run download {run-id} -n postman-sync-artifacts-{run-id}
```

**Use Cases:**
- **Debugging** - Verify the stringification is correct
- **Verification** - Confirm the payload matches expectations
- **Troubleshooting** - Diagnose API errors by reviewing the exact content sent
- **Testing** - Compare stringified output against local testing

### Pull Request Behavior

When `create_pr` is `true` and changes are synced to the repository:

- **Branch Created:** `sync-postman-spec-{spec-id}`
- **PR Title:** "Sync: Update spec from Postman ({spec-name})"
- **PR Body:** Includes spec details and change summary
- **Labels:** `sync`, `postman`, `automated`
- **Reviewer:** Workflow triggering user

The PR includes:
- Spec ID and name
- File path that changed
- Link to workflow run
- Details about the sync

### Example Workflow Run

**Normal Mode** (debug: false):
```
🔄 Postman Spec Sync Summary

Spec ID: `abc123-def456-ghi789`
Spec Name: Retail Accounts API
File Path: `postman/schemas/accounts-with-schema.yaml`
Sync Direction: both

✅ Repository updated from Postman spec
✅ Postman spec updated from repository
```

**Debug Mode** (debug: true):
```
🔄 Postman Spec Sync Summary

Spec ID: `abc123-def456-ghi789`
Spec Name: Retail Accounts API
File Path: `postman/schemas/accounts-with-schema.yaml`
Sync Direction: both

✅ Repository updated from Postman spec
✅ Postman spec updated from repository

📦 Debug Artifacts
Debug mode enabled - artifacts saved for troubleshooting.

Artifact name: `postman-sync-artifacts-1234567890`

Includes:
- `stringified-content.txt` - The raw stringified file content
- `json-payload.json` - The complete JSON payload sent to Postman API
- `source-file-path.txt` - The source file path
- `spec-id.txt` - The Postman spec ID
- `metadata.txt` - Sync metadata (timestamp, root file, status)
- `api-response.json` - Postman API response (if sync was attempted)
```

### Error Handling

The workflow handles various error scenarios:

| Error | Description | Resolution |
|-------|-------------|------------|
| Invalid API Key | Postman API key not valid | Check `POSTMAN_API_KEY` secret |
| Spec Not Found | Spec ID doesn't exist | Verify spec ID from Postman |
| File Not Found | File doesn't exist in repo | Check file path or create file |
| Permission Error | GitHub token lacks permissions | Update repository permissions |
| API Rate Limit | Too many API calls | Wait and retry later |

### Troubleshooting

#### Workflow Fails to Find Spec

**Problem:** "Failed to fetch spec from Postman (HTTP 404)"

**Solutions:**
- Verify the Spec ID is correct
- Check that the spec exists in Postman
- Ensure API key has access to the spec
- Verify the spec is in your workspace

#### File Not Found Error

**Problem:** "File 'path/to/file.yaml' not found in repository"

**Solutions:**
- Check the file path is correct
- Ensure file exists in the repository
- Create the file if syncing from Postman to repo

#### Permission Denied

**Problem:** "Permission denied" when creating PR

**Solutions:**
- Ensure GitHub Actions has write permissions
- Go to Settings → Actions → General → Workflow permissions
- Select "Read and write permissions"
- Save changes

#### API Key Invalid

**Problem:** "Unauthorized (HTTP 401)"

**Solutions:**
- Verify `POSTMAN_API_KEY` secret is set
- Check the API key is still valid in Postman
- Generate a new API key if needed
- Update the GitHub secret

#### Content Stringification Issues

**Problem:** "Failed to update spec in Postman" with malformed content error

**Solutions:**
- Check workflow logs for "Stringified Content Preview"
- Verify file encoding is UTF-8
- Ensure file doesn't contain null bytes or invalid characters
- Check file size isn't too large (>10 MB)
- Review the JSON payload preview in logs

**Test stringification locally:**
```bash
# Test how your file will be stringified
cat postman/schemas/accounts.yaml | jq -Rs .

# Should output a JSON string with \n for newlines
"openapi: 3.0.0\ninfo:\n  title: My API\n..."
```

#### Debugging with Artifacts

**Problem:** Need to verify what was sent to Postman API

**Solution:**
```bash
# Download the artifacts from the workflow run
gh run download {run-id} -n postman-sync-artifacts-{run-id}

# Compare the stringified content with your local stringification
cd postman-sync-artifacts-{run-id}
cat stringified-content.txt

# Compare with local
cat ../postman/schemas/accounts.yaml | jq -Rs .

# Check the JSON payload structure
cat json-payload.json | jq .

# Verify it matches expected format
cat json-payload.json | jq -r '.content' | head -c 200
```

**Use artifacts to:**
- Verify stringification matches local testing
- Confirm JSON payload structure
- Debug API errors by seeing exact content sent
- Validate special character handling
- Compare successful vs failed runs

### Best Practices

1. **Use Pull Requests**
   - Set `create_pr: true` for repo changes
   - Review changes before merging
   - Maintains audit trail

2. **Sync Direction**
   - Use `both` for regular syncing
   - Use specific direction when you know the source of truth
   - Avoid conflicts by coordinating team changes

3. **File Organization**
   - Keep specs in dedicated folder (e.g., `postman/schemas/`)
   - Use descriptive filenames
   - Match Postman spec names when possible

4. **Regular Syncing**
   - Run workflow after making changes in either location
   - Consider scheduling regular syncs
   - Document your sync strategy with the team

5. **Version Control**
   - Always review PRs before merging
   - Add meaningful commit messages
   - Tag releases when specs stabilize

### Scheduling Automatic Syncs

To run the sync automatically on a schedule, add this to the workflow:

```yaml
on:
  workflow_dispatch:
    # ... existing inputs ...
  
  schedule:
    # Run every day at 2 AM UTC
    - cron: '0 2 * * *'
```

**Note:** Scheduled runs will use default parameters, so you may need to modify the workflow to support multiple specs.

### Integration with CI/CD

You can integrate this workflow with your CI/CD pipeline:

```yaml
# In your main CI workflow
- name: Sync Spec from Postman
  uses: actions/github-script@v7
  with:
    script: |
      await github.rest.actions.createWorkflowDispatch({
        owner: context.repo.owner,
        repo: context.repo.repo,
        workflow_id: 'sync-postman-spec.yml',
        ref: 'main',
        inputs: {
          spec_id: '${{ secrets.POSTMAN_SPEC_ID }}',
          file_path: 'postman/schemas/accounts.yaml',
          sync_direction: 'postman-to-repo',
          create_pr: 'true'
        }
      })
```

### Advanced Usage

#### Multiple Specs

To sync multiple specs, create a matrix strategy:

```yaml
strategy:
  matrix:
    spec:
      - id: "spec-1-id"
        file: "postman/schemas/accounts.yaml"
      - id: "spec-2-id"
        file: "postman/schemas/users.yaml"
```

#### Custom Branch Names

Modify the PR creation step to use custom branch names:

```yaml
branch: feature/sync-${{ inputs.file_path }}
```

#### Notifications

Add Slack/Teams notifications:

```yaml
- name: Notify Slack
  uses: slackapi/slack-github-action@v1
  with:
    payload: |
      {
        "text": "Spec synced: ${{ steps.get-spec.outputs.spec_name }}"
      }
```

### Security Considerations

1. **API Key Management**
   - Store API keys as GitHub secrets only
   - Rotate keys periodically
   - Use least privilege principle
   - Never commit keys to repository

2. **Branch Protection**
   - Enable branch protection on main
   - Require PR reviews
   - Prevent direct pushes

3. **Workflow Permissions**
   - Limit to necessary permissions
   - Review workflow changes carefully
   - Audit workflow runs regularly

### Support

For issues or questions:
1. Check workflow run logs in Actions tab
2. Review error messages in step summaries
3. Verify all prerequisites are met
4. Check Postman API status page

### Related Documentation

- [Postman API Documentation](https://www.postman.com/postman/workspace/postman-public-workspace/documentation/12959542-c8142d51-e97c-46b6-bd77-52bb66712c9a)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [OpenAPI Specification](https://swagger.io/specification/)

---

**Quick Start Checklist:**
- [ ] Add `POSTMAN_API_KEY` to GitHub secrets
- [ ] Get your Postman Spec ID
- [ ] Run workflow from Actions tab
- [ ] Review and merge PR
- [ ] Verify sync completed successfully

