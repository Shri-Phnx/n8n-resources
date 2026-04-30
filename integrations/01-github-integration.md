# GitHub Integration in n8n — Complete Guide

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026  
> **Covers:** GitHub node fields, all use cases including tracker deduplication, scheduled backups, and version control for workflows.

---

## Part 1 — Setting Up GitHub Credentials in n8n

### What Kind of GitHub Auth Does n8n Use?

n8n supports two authentication methods for GitHub:

| Method | Best For | Notes |
|--------|---------|-------|
| **Personal Access Token (PAT)** | Personal repos, automation, most use cases | Simple, works everywhere |
| **GitHub App** | Organization-level automation, fine-grained permissions | More complex setup, better for enterprise |

**Recommendation:** Use a **Fine-grained Personal Access Token** for all personal and team automations.

---

### Step 1 — Generate a GitHub Personal Access Token

1. Go to **https://github.com** → log in
2. Click your **profile picture** (top right) → **Settings**
3. Scroll all the way down in the left sidebar → click **Developer settings**
4. Click **Personal access tokens** → **Fine-grained tokens**
5. Click **Generate new token**
6. Fill in:
   - **Token name:** `n8n-automation` (or describe what it's for)
   - **Expiration:** 90 days or custom (you'll need to rotate it when it expires)
   - **Resource owner:** Your username (or organization if needed)
   - **Repository access:** Choose **Only select repositories** → select the repos n8n should access (or **All repositories** for full access)
7. **Permissions — set these based on your use cases:**

| Permission | Access Level | When You Need It |
|-----------|-------------|------------------|
| **Contents** | Read and Write | Reading/writing files in repos |
| **Metadata** | Read (auto-selected) | Required for all operations |
| **Issues** | Read and Write | Creating/reading GitHub issues |
| **Pull requests** | Read and Write | Creating PRs, reading PR data |
| **Workflows** | Read and Write | Triggering GitHub Actions |
| **Commit statuses** | Read and Write | Setting CI status |

8. Click **Generate token**
9. **IMPORTANT:** Copy the token NOW — GitHub shows it only once. It starts with `github_pat_`
10. Store it in your password manager

---

### Step 2 — Add GitHub Credential in n8n

1. In n8n: click **Settings** (gear icon) → **Credentials**
2. Click **Add credential** (top right)
3. Search for **GitHub** → select it
4. Fill in:
   - **Credential Name:** `GitHub - Personal (Shrinivas)` — use descriptive names if you have multiple
   - **Access Token:** Paste your `github_pat_...` token
5. Click **Save**
6. n8n tests the credential immediately. You should see **"Connection tested successfully"**

**If the test fails:**
| Problem | Fix |
|---------|-----|
| 401 Unauthorized | Token is wrong or expired. Regenerate on GitHub |
| 403 Forbidden | Token doesn't have the permissions needed. Edit the token on GitHub and add required permissions |
| 404 Not Found | Repo name or owner is wrong in the node, not the credential |

---

## Part 2 — GitHub Node: Every Field Explained

### Parameters Tab

#### Field: Credential to connect with
| What it is | Dropdown to select your saved GitHub credential |
|-----------|---|
| **If empty** | Node fails — credential is required |
| **Tip** | Give credentials descriptive names: "GitHub - Work PAT", "GitHub - Personal PAT" |

---

#### Field: Resource

| Resource | What You Can Do With It |
|----------|------------------------|
| **File** | Read, create, update, delete files in a repo |
| **Repository** | Get repo info, list repos, create repos |
| **Issue** | Create, get, list, edit, lock issues |
| **Pull Request** | Create, get, list, merge PRs |
| **Release** | Create, get, list, delete releases |
| **Review** | Create, get reviews on PRs |
| **User** | Get user info, check followers |
| **Organization** | Get organization details |

**Most common for automation:** `File` and `Repository`

---

#### Field: Operation (changes based on Resource)

**Resource = File:**

| Operation | What It Does | Required Fields |
|-----------|-------------|----------------|
| **Create** | Create a new file in the repo | Owner, Repo, File Path, File Content, Commit Message |
| **Delete** | Delete an existing file | Owner, Repo, File Path, Commit Message, File SHA |
| **Edit** | Update an existing file's content | Owner, Repo, File Path, File Content, Commit Message, File SHA |
| **Get** | Read a file's content and metadata | Owner, Repo, File Path |
| **List** | List files in a directory | Owner, Repo, File Path (directory) |

---

#### Field: Owner
| What it is | Your GitHub username or organization name |
|-----------|---|
| **Example** | `Shri-Phnx` (your username) or `my-org` |
| **Dynamic expression** | `{{ $vars.GITHUB_OWNER }}` — recommended |
| **If wrong** | 404 — repo not found |

---

#### Field: Repository
| What it is | The exact name of the GitHub repository |
|-----------|---|
| **Example** | `n8n-resources`, `my-tracker`, `workflow-data` |
| **Case-sensitive** | Yes — `N8N-Resources` ≠ `n8n-resources` |
| **If wrong** | 404 error |

---

#### Field: File Path
| What it is | The path to the file inside the repo |
|-----------|---|
| **Example** | `data/tracker.json` — file `tracker.json` inside `data/` folder |
| **Example** | `README.md` — file in root |
| **Example** | `reports/2026/april/summary.csv` — nested folders |
| **No leading slash** | Write `data/file.json` not `/data/file.json` |
| **Folders are created automatically** | If `data/` doesn't exist, GitHub creates it when you create a file inside it |
| **If file doesn't exist for Edit/Delete** | 404 error |

---

#### Field: File Content
| What it is | The content to write to the file (text only) |
|-----------|---|
| **Type** | Plain text. If writing JSON, convert the object to a string first |
| **Expression for JSON** | `{{ JSON.stringify($json.data, null, 2) }}` |
| **Expression for CSV** | Build the CSV string in a Code node first, then reference it |
| **If binary** | GitHub API via this node only supports text content. For binary files, use HTTP Request node with the raw GitHub API |

---

#### Field: Commit Message
| What it is | The Git commit message describing the change |
|-----------|---|
| **Example** | `"Update tracker — added 5 new records"` |
| **Best practice** | Make it descriptive and include a timestamp: `` `data: tracker update — {{ $now.toFormat('yyyy-MM-dd HH:mm') }}` `` |
| **If empty** | GitHub API rejects the request — commit message is required |

---

#### Field: File SHA
| What it is | The SHA hash of the current version of the file |
|-----------|---|
| **Why required for Edit/Delete** | GitHub uses this to prevent overwriting someone else's changes (optimistic locking). It verifies you're editing the version you think you are |
| **Where to get it** | From the output of a **Get File** operation. The SHA is in `{{ $json.sha }}` |
| **If wrong** | 409 Conflict error — the file was changed between your read and write |
| **If file is new** | Leave empty for Create operations |

---

#### Field: Branch
| What it is | The Git branch to read from or write to |
|-----------|---|
| **Default** | `main` (GitHub's default branch, previously `master`) |
| **When to change** | If you use a different default branch, or want to write to a feature branch |
| **Check your repo's default** | Go to github.com/Shri-Phnx/repo → look at the branch dropdown |

---

#### Field: Additional Parameters → Author Name / Email
| What it is | The Git author shown in commit history |
|-----------|---|
| **Default** | Uses the GitHub account associated with your PAT |
| **When to set** | When you want commits to show a specific automation author like `n8n-bot` |
| **Example** | Author Name: `n8n Automation`, Author Email: `n8n@yourdomain.com` |

---

## Part 3 — Use Case 1: Read a File from GitHub

**Scenario:** Your workflow needs to read a configuration file or data file stored in a GitHub repo.

```
Manual / Webhook Trigger
    → GitHub node
        Resource: File
        Operation: Get
        Owner: Shri-Phnx
        Repository: my-tracker
        File Path: data/processed-ids.json
        Branch: main
    → Code node (parse the file content)
    → ... rest of workflow
```

**The GitHub Get File node outputs:**
```json
{
  "name": "processed-ids.json",
  "path": "data/processed-ids.json",
  "sha": "abc123def456...",        ← Save this for updates
  "size": 1234,
  "content": "W1widGlja2V0XzEi...",  ← Base64 encoded content
  "encoding": "base64"
}
```

**Decode the Base64 content in a Code node:**
```javascript
const base64Content = $json.content;
// GitHub returns content with newlines in the base64 string — remove them
const cleanBase64 = base64Content.replace(/\n/g, '');
const decoded = Buffer.from(cleanBase64, 'base64').toString('utf-8');
const data = JSON.parse(decoded);

return [{
  json: {
    fileData: data,
    sha: $json.sha  // Keep the SHA for when we write back
  }
}];
```

---

## Part 4 — Use Case 2: Write / Update a File in GitHub

**Scenario:** Save workflow output to a file in your GitHub repo.

**Step 1 — Prepare the content in a Code node:**
```javascript
// Get the data you want to save
const records = $input.all().map(item => item.json);

// Convert to JSON string (GitHub API needs text content)
const content = JSON.stringify(records, null, 2);

// Encode to Base64 (GitHub API requires base64 encoded content)
const base64Content = Buffer.from(content).toString('base64');

return [{
  json: {
    content: base64Content,
    commitMessage: `data: update — ${new Date().toISOString()}`
  }
}];
```

**Step 2 — GitHub node:**
```
Resource: File
Operation: Create  ← for new file
            Edit   ← for existing file (requires SHA)
Owner: Shri-Phnx
Repository: my-tracker
File Path: data/output.json
File Content: {{ $json.content }}
Commit Message: {{ $json.commitMessage }}
Branch: main
File SHA: {{ $json.sha }}  ← only for Edit, from previous Get operation
```

---

## Part 5 — Use Case 3: Scheduled Workflow Backup to Private GitHub

**Goal:** Automatically export all n8n workflows as JSON and save them to a private GitHub repo on a schedule.

**Why this is valuable:**
- Version control for your workflows
- Restore any workflow to a previous state
- Move workflows between n8n instances
- Track changes over time

### The Complete Workflow

```
Schedule Trigger (every day at 2 AM)
    → HTTP Request (GET n8n workflows via n8n API)
    → Split Out (split each workflow into individual items)
    → Loop Over Items
          → Code node (prepare file content + path)
          → GitHub node (File → Create/Edit)
```

### Step 1 — Enable the n8n API

In n8n: **Settings → n8n API → Enable API**. Copy the API key shown.

### Step 2 — Get All Workflows

**HTTP Request node:**
```
Method: GET
URL: http://localhost:5678/api/v1/workflows
     (or https://your-domain.com/api/v1/workflows)
Authentication: Header Auth
  Header Name: X-N8N-API-KEY
  Header Value: your-n8n-api-key
```

**Output:** `{ data: [ {...workflow1...}, {...workflow2...} ] }`

### Step 3 — Split and Loop

**Edit Fields node:** Set `workflows` = `{{ $json.data }}`
**Split Out node:** Field to split: `workflows`

### Step 4 — Prepare Each Workflow File

**Code node (runs once per workflow):**
```javascript
const workflow = $json;
const fileName = `${workflow.name.replace(/[^a-zA-Z0-9-_]/g, '_')}_${workflow.id}.json`;
const content = JSON.stringify(workflow, null, 2);
const base64Content = Buffer.from(content).toString('base64');

return [{
  json: {
    fileName: fileName,
    filePath: `workflows/${fileName}`,
    content: base64Content,
    workflowId: workflow.id,
    workflowName: workflow.name
  }
}];
```

### Step 5 — Check if File Exists, Then Create or Update

The tricky part: you need the SHA if the file already exists (Edit), but no SHA if it's new (Create). Handle this with an If node:

```
GitHub node (Try to Get file)
  Resource: File
  Operation: Get
  File Path: {{ $json.filePath }}
  → Settings tab: On Error = Continue

If node: {{ $json.sha }} is not empty?
  [True branch]  → GitHub Edit (include SHA)
  [False branch] → GitHub Create (no SHA)
```

**Better pattern using a single Code node + HTTP Request:**
```javascript
// Try to get the SHA first
try {
  const response = await $helpers.request({
    method: 'GET',
    url: `https://api.github.com/repos/Shri-Phnx/n8n-backups/contents/${$json.filePath}`,
    headers: { 
      'Authorization': 'Bearer github_pat_your_token',
      'Accept': 'application/vnd.github.v3+json'
    }
  });
  const parsed = JSON.parse(response);
  return [{ json: { ...$json, sha: parsed.sha, isExisting: true } }];
} catch (e) {
  // File doesn't exist yet
  return [{ json: { ...$json, sha: '', isExisting: false } }];
}
```

---

## Part 6 — Use Case 4: Tracker with Deduplication (Your Use Case)

**Goal:** Build a tracker workflow where:
1. On each run, the workflow processes new items
2. Before processing, it checks GitHub to see what's already been processed
3. Only processes items NOT in the tracker
4. After processing, updates the tracker in GitHub
5. Next run picks up from where it left off

**This is completely possible and is a clean pattern for personal/low-volume workflows.**

### Architecture Overview

```
Every Run:
├── 1. Read tracker from GitHub (JSON file with processed IDs)
├── 2. Fetch new data from source (API, sheet, form, etc.)
├── 3. Filter: remove items already in tracker
├── 4. Process remaining items
├── 5. Update tracker: append newly processed IDs to JSON
└── 6. Write updated tracker back to GitHub
```

---

### The Tracker File Format

Store a simple JSON file in your private repo at `tracker/processed.json`:

```json
{
  "lastUpdated": "2026-04-30T08:00:00.000Z",
  "processedIds": [
    "ticket_001",
    "ticket_002",
    "ticket_003"
  ],
  "metadata": {
    "totalProcessed": 3,
    "lastProcessedId": "ticket_003"
  }
}
```

---

### Step-by-Step Workflow Build

#### Node 1 — Trigger
```
Schedule Trigger
  Every: 1 Hour (or whatever interval makes sense)
```

#### Node 2 — Read the Tracker from GitHub
```
GitHub node:
  Resource: File
  Operation: Get
  Owner: Shri-Phnx
  Repository: my-private-tracker
  File Path: tracker/processed.json
  Branch: main

Settings Tab:
  On Error: Continue
  Always Output Data: ON
```

Why `On Error: Continue`? On the very first run, the file doesn't exist yet. This prevents the workflow from stopping.

#### Node 3 — Parse the Tracker

```javascript
// Code node — runs once
let processedIds = [];
let fileSha = '';

// Check if GitHub returned a file (sha field exists) or an error
if ($json.sha) {
  fileSha = $json.sha;
  // Decode base64 content
  const cleanBase64 = $json.content.replace(/\n/g, '');
  const decoded = Buffer.from(cleanBase64, 'base64').toString('utf-8');
  const tracker = JSON.parse(decoded);
  processedIds = tracker.processedIds || [];
}

// Store in the workflow state
return [{
  json: {
    processedIds: processedIds,
    fileSha: fileSha,
    isFirstRun: fileSha === ''
  }
}];
```

#### Node 4 — Fetch New Data from Your Source

This is whatever your workflow actually processes. Examples:

```
// Example A: Fetch from an API
HTTP Request node:
  GET https://api.example.com/items?status=pending

// Example B: Read from Google Sheets
Google Sheets node:
  Operation: Read Rows

// Example C: Read from Airtable
Airtable node:
  Operation: List
  Filter by formula: {status} = 'new'
```

#### Node 5 — Filter Out Already-Processed Items

```javascript
// Code node — runs once for all items
// Get the processed IDs from Node 3
const processedIds = $('ParseTracker').first().json.processedIds;

// Get all new items from Node 4
const allItems = $input.all();

// Filter: keep only items NOT already processed
const newItems = allItems.filter(item => {
  const itemId = item.json.id;  // Use whatever field is your unique ID
  return !processedIds.includes(itemId);
});

if (newItems.length === 0) {
  console.log('No new items to process');
  // Return an empty item so workflow continues but nothing is processed
  return [{ json: { newItemsCount: 0, items: [] } }];
}

console.log(`Found ${newItems.length} new items to process`);
return newItems;
```

#### Node 6 — If (New Items Exist?)

```
If node:
  Left value: {{ $json.id }}  (or any field that exists on a real item)
  Operator: Is Not Empty

[True output]  → Your processing nodes
[False output] → No Operation node (stop gracefully)
```

#### Node 7 — Your Processing Logic

This is your actual workflow — send emails, create records, call APIs, whatever you're automating. The processed items flow through here.

#### Node 8 — Collect Processed IDs

```javascript
// Code node — runs once for all items after processing
// Collect IDs of everything we just processed
const processedItems = $input.all();
const newlyProcessedIds = processedItems.map(item => item.json.id);

// Get existing IDs from the tracker
const existingIds = $('ParseTracker').first().json.processedIds;
const existingSha = $('ParseTracker').first().json.fileSha;

// Merge old + new IDs
const allIds = [...existingIds, ...newlyProcessedIds];

// Build the updated tracker object
const updatedTracker = {
  lastUpdated: new Date().toISOString(),
  processedIds: allIds,
  metadata: {
    totalProcessed: allIds.length,
    lastProcessedId: newlyProcessedIds[newlyProcessedIds.length - 1] || existingIds[existingIds.length - 1]
  }
};

// Encode to base64 for GitHub API
const content = JSON.stringify(updatedTracker, null, 2);
const base64Content = Buffer.from(content).toString('base64');

return [{
  json: {
    content: base64Content,
    sha: existingSha,
    isFirstRun: existingSha === '',
    newlyProcessedCount: newlyProcessedIds.length,
    totalProcessed: allIds.length
  }
}];
```

#### Node 9 — Write Updated Tracker to GitHub

**If node: Is First Run?** (`{{ $json.isFirstRun }}` is true)

```
[True / First run] → GitHub node:
  Resource: File
  Operation: Create
  Owner: Shri-Phnx
  Repository: my-private-tracker
  File Path: tracker/processed.json
  File Content: {{ $json.content }}
  Commit Message: "tracker: initial commit — {{ $now.toISO() }}"
  Branch: main

[False / File exists] → GitHub node:
  Resource: File
  Operation: Edit
  Owner: Shri-Phnx
  Repository: my-private-tracker
  File Path: tracker/processed.json
  File Content: {{ $json.content }}
  Commit Message: "tracker: +{{ $json.newlyProcessedCount }} new, total {{ $json.totalProcessed }} — {{ $now.toISO() }}"
  Branch: main
  File SHA: {{ $json.sha }}
```

---

### Complete Workflow Diagram

```
Schedule Trigger (hourly)
    ↓
GitHub: Get File (tracker/processed.json)  [On Error: Continue]
    ↓
Code: Parse Tracker → { processedIds[], fileSha }
    ↓
HTTP Request / Sheet / API: Get new data
    ↓
Code: Filter (remove already processed IDs)
    ↓
If: New items exist?
    ↓ [YES]                              ↓ [NO]
Your Processing Logic           No Operation (stop gracefully)
    ↓
Code: Build updated tracker JSON + base64 encode
    ↓
If: Is first run?
    ↓ [YES - Create]        ↓ [NO - Edit with SHA]
GitHub: File Create    GitHub: File Edit
```

---

### ⚠️ Limitations of Using GitHub as a Tracker

| Limitation | Detail | When It Matters |
|-----------|--------|----------------|
| **Concurrent access** | If two workflow instances run at the same time and both read/write the same file, one commit will overwrite the other | If you trigger the same workflow manually while a scheduled run is active |
| **GitHub API rate limits** | 5,000 requests/hour for authenticated users | Not a problem for most use cases, but don't run on sub-minute schedules |
| **File size** | GitHub API has a 100MB file size limit for regular API access | If your tracker grows to millions of entries, use a database instead |
| **No atomic transactions** | Read-modify-write is not atomic — there's a gap between reading and writing | Low-volume personal workflows: acceptable. High-volume production: use a DB |

---

### Better Alternative for High Volume: PostgreSQL or Google Sheets

If the GitHub approach hits limits:

**Option A — PostgreSQL (if you have a server):**
```sql
-- One-time setup
CREATE TABLE processed_items (
  item_id VARCHAR(255) PRIMARY KEY,
  processed_at TIMESTAMP DEFAULT NOW(),
  workflow_name VARCHAR(255)
);

-- In n8n: Postgres node
-- Check: SELECT item_id FROM processed_items WHERE item_id = '{{ $json.id }}'
-- Insert: INSERT INTO processed_items (item_id, workflow_name) VALUES ('{{ $json.id }}', 'my-workflow')
```

**Option B — Google Sheets (easiest, no server needed):**
```
1. Google Sheets: Read all rows → get array of processed IDs
2. Code: filter incoming items against the array
3. Process filtered items
4. Google Sheets: Append new rows for each processed item
```

**Option C — n8n's own Static Data (simplest, but data is local to n8n):**
```javascript
// Read persisted data
const staticData = this.getWorkflowStaticData('global');
const processedIds = staticData.processedIds || [];

// After processing
staticData.processedIds = [...processedIds, ...newlyProcessedIds];
// Data persists across executions automatically
```
This is the simplest approach but data lives inside n8n — not accessible from outside.

---

## Part 7 — Use Case 5: Trigger n8n from GitHub Events

**Use the GitHub Trigger node** (not the Webhook node) for this.

```
GitHub Trigger node:
  Events: push, pull_request, issues, release, etc.
  Owner: Shri-Phnx
  Repository: my-project
  Credential: your GitHub PAT

→ If (event.action == 'opened')
    → Slack: "New PR opened: {{ $json.pull_request.title }}!"
→ If (event.ref includes 'main')
    → HTTP Request: trigger deployment webhook
```

**Common automation patterns:**
- New PR → Slack notification + assign reviewer
- PR merged to main → trigger deployment
- New issue labeled "bug" → create Jira ticket
- New release published → send announcement email

---

## Part 8 — Use Case 6: Save Workflow Output as a Report

**Goal:** Every time a workflow runs, save a Markdown or JSON report to a GitHub repo for review.

```javascript
// Code node — build the report
const results = $input.all();
const reportDate = new Date().toISOString().split('T')[0];

let markdownReport = `# Workflow Report — ${reportDate}\n\n`;
markdownReport += `**Executed at:** ${new Date().toLocaleString('en-IN', {timeZone: 'Asia/Kolkata'})}\n`;
markdownReport += `**Total items processed:** ${results.length}\n\n`;
markdownReport += `## Results\n\n`;
markdownReport += `| ID | Status | Details |\n|-----|--------|---------|\n`;

results.forEach(item => {
  markdownReport += `| ${item.json.id} | ${item.json.status} | ${item.json.details} |\n`;
});

const base64 = Buffer.from(markdownReport).toString('base64');
const fileName = `report-${reportDate}-${Date.now()}.md`;

return [{ json: { content: base64, fileName, reportDate } }];
```

```
GitHub node:
  Operation: Create
  File Path: reports/{{ $json.reportDate }}/{{ $json.fileName }}
  File Content: {{ $json.content }}
  Commit Message: "report: auto-generated {{ $json.reportDate }}"
```

---

## Part 9 — Use Case 7: GitHub as a Configuration Store

**Goal:** Store your workflow configuration (URLs, thresholds, recipient lists) in a GitHub file so you can edit it without modifying the workflow itself.

**config.json in your repo:**
```json
{
  "alertThreshold": 100,
  "recipientEmails": ["shrinivas@example.com", "team@example.com"],
  "slackChannel": "#alerts",
  "apiBaseUrl": "https://api.yourservice.com",
  "enabled": true
}
```

**In your n8n workflow:**
```
Schedule Trigger
    → GitHub: Get File (config.json)
    → Code: Decode base64 → parse JSON → extract settings
    → If: config.enabled == true?
        [YES] → Continue with workflow using config values
        [NO]  → Stop (workflow is disabled via config)
```

Now you can enable/disable workflows or change settings by editing a file on GitHub — no need to touch n8n itself.

---

## Part 10 — Common GitHub Integration Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `404 Not Found` | Repo name/owner wrong, or file doesn't exist | Double-check owner and repo name. Use Get first to verify file exists |
| `409 Conflict` | SHA mismatch — file was updated between your Get and Edit | Get the file again to get current SHA, then Edit |
| `422 Unprocessable Entity` | File content not base64 encoded | Encode with `Buffer.from(text).toString('base64')` |
| `403 Forbidden` | Token doesn't have required permissions | Edit token on GitHub → add Contents: Write permission |
| `401 Unauthorized` | Token expired or invalid | Generate new token on GitHub, update n8n credential |
| `GithubError: Reference not found` | Branch name wrong | Check the actual branch name on GitHub (main vs master) |
| File content looks garbled | Base64 not decoded | Use `Buffer.from(content.replace(/\n/g,''), 'base64').toString('utf-8')` |

---

## Part 11 — Impossible Things & Better Alternatives

| Request | Is It Possible? | Better Approach |
|---------|----------------|----------------|
| Atomic read+write (no race condition) | ❌ Not with GitHub API alone | Use PostgreSQL or Redis with transactions |
| Store binary files (images, PDFs) | ✅ Via HTTP Request to raw GitHub API with base64 | Standard GitHub node supports text only — use HTTP Request for binaries |
| Private repos with public webhook triggers | ✅ Use GitHub Trigger + PAT with private repo access | — |
| Search across all files in a repo | ❌ GitHub API search is limited | Clone repo locally or use GitHub's search API via HTTP Request |
| Realtime updates when GitHub file changes | ✅ Use GitHub Trigger node with "push" event | — |
| Store millions of records | ❌ GitHub will throttle, large files are slow | Use PostgreSQL, MongoDB, or Supabase instead |
| Track which USER last modified data | ✅ GitHub commit history shows exactly who changed what | This is a GitHub advantage over a plain database |
