# n8n Community Nodes & Custom Node Development

> **Author:** Shrinivas Ramaprasad  
> **Last updated:** April 2026  
> **Source:** [n8n Community Nodes Docs](https://docs.n8n.io/integrations/community-nodes/)

---

## Part 1 — Community Nodes

### What are Community Nodes?

Community nodes are third-party n8n nodes published to npm by the developer community. They extend n8n with integrations and capabilities not available in the core product.

**Key facts:**
- Hosted on the npm registry with the keyword `n8n-community-node-package`
- Marked with a **Package icon** in the node panel to distinguish from built-in nodes
- Can be **verified** (vetted by n8n) or **unverified** (community-submitted)
- Unverified nodes require a self-hosted n8n instance to install

---

### Verified vs. Unverified Community Nodes

| Attribute | Verified | Unverified |
|-----------|----------|------------|
| Vetted by n8n team | ✅ Yes | ❌ No |
| Available on n8n Cloud | ✅ Yes | ❌ No |
| Discoverable in node panel | ✅ Yes | ❌ No — must know package name |
| Requires self-hosted | ❌ No | ✅ Yes |
| Install from node panel | ✅ Yes | ❌ No |
| Security guarantee | Partial | None — use at own risk |

---

### Method 1 — Install Verified Nodes (from Node Panel)

**Works on:** n8n Cloud and self-hosted instances
**Who can install:** Instance Owner and Admin roles only

**Steps:**
1. Go to the canvas and open the node panel (`+` or press `N`)
2. Search for the node by name
3. If a matching verified community node exists, a **"More from the community"** section appears at the bottom of the results
4. Select the node → click **Install**
5. The node is now available to all users on the instance

---

### Method 2 — Install via Settings GUI (Unverified, npm)

**Works on:** Self-hosted instances only
**Who can install:** Owner and Admin roles only

**Steps:**
1. Go to **Settings → Community Nodes**
2. Click **Install**
3. Click **Browse** to see all npm packages tagged with `n8n-community-node-package`
4. Note the exact package name (e.g., `n8n-nodes-nvidia-nim`)
5. Return to n8n and enter the package name in **"Enter npm package name"**
6. To install a specific version: `n8n-nodes-nvidia-nim@2.3.4`
7. To install a dist-tag: `n8n-nodes-nvidia-nim@beta`
8. Check the box: **"I understand the risks of installing unverified code from a public source"**
9. Click **Install**

---

### Method 3 — Manual Command-Line Install

**Works on:** Self-hosted instances (required for queue mode)
**Use when:** GUI install is unavailable, or you want to install private packages

```bash
# Navigate to your n8n installation directory
cd ~/.n8n

# Install the community node via npm
npm install n8n-nodes-nvidia-nim

# Or install a specific version
npm install n8n-nodes-nvidia-nim@2.3.4

# Restart n8n to load the new node
pm2 restart n8n
# or
systemctl restart n8n
# or
docker restart n8n-container
```

**For Docker installations:**
```bash
# Create a custom Dockerfile
FROM n8nio/n8n:latest
RUN npm install -g n8n-nodes-nvidia-nim

# Or use the N8N_CUSTOM_EXTENSIONS environment variable
# in docker-compose.yml:
environment:
  - N8N_CUSTOM_EXTENSIONS=/home/node/.n8n/custom
```

---

### Managing Community Nodes

#### Update a Community Node
1. Go to **Settings → Community Nodes**
2. Find the node — if a newer version is available, an **Update** button appears
3. Click **Update** to upgrade to the latest version
4. To upgrade to a specific version: uninstall → reinstall with version specified

#### Downgrade a Community Node
1. **Settings → Community Nodes** → Options → **Uninstall**
2. Reinstall with the target version: `n8n-nodes-package@1.2.3`

#### Uninstall a Community Node
1. **Settings → Community Nodes**
2. Find the node → click **Options** (three dots)
3. Click **Uninstall package**
4. Confirm in the modal

---

### Notable Community Nodes (2025–2026)

| Package | What it Does |
|---------|-------------|
| `n8n-nodes-nvidia-nim` | NVIDIA NIM chat completions and image analysis |
| `n8n-nodes-nim` | NVIDIA NIM model interactions |
| `n8n-nodes-browserless` | Headless browser automation |
| `n8n-nodes-puppeteer` | Puppeteer web scraping and automation |
| `n8n-nodes-firecrawl` | AI-friendly web scraping and crawling |
| `n8n-nodes-qdrant` | Qdrant vector database (legacy — now built-in) |
| `n8n-nodes-evolution-api` | WhatsApp Evolution API integration |
| `n8n-nodes-minio` | MinIO object storage |
| `n8n-nodes-pgvector` | pgvector PostgreSQL extension (legacy — now built-in) |
| `n8n-nodes-chatwoot` | Chatwoot customer support platform |
| `n8n-nodes-appwrite` | Appwrite backend-as-a-service |
| `n8n-nodes-posthog` | PostHog analytics (legacy — now has built-in node) |

> 🔍 **Find all community nodes:** Search npm for `n8n-community-node-package` or browse the [n8n integrations page](https://n8n.io/integrations/).

---

### Community Node Risks

| Risk | Mitigation |
|------|------------|
| Malicious code | Only install from trusted, reviewed packages |
| Breaking changes on updates | Pin specific versions; test before upgrading |
| Abandoned packages | Check npm for recent updates and GitHub activity |
| Data exfiltration | Review source code on GitHub before installing |
| No SLA | Have a fallback plan using HTTP Request node |

---

## Part 2 — Custom Node Development

### When to Build a Custom Node

| Situation | Solution |
|-----------|----------|
| Service not in n8n's 300+ integrations | Build a custom node |
| Internal proprietary API | Build a private custom node |
| Specific data transformation logic | Use Code node (simpler) or custom node |
| Reusable integration across many workflows | Build a custom node |
| Want to share with the community | Build and publish to npm |

---

### Prerequisites

- **Node.js** v18 or higher
- **npm** or **pnpm**
- **TypeScript** knowledge (nodes are written in TypeScript)
- n8n installed locally for testing

---

### Step 1 — Scaffold the Node

Use the official n8n node CLI tool (strongly recommended for verified submissions):

```bash
# Install the n8n node scaffolding tool
npx n8n-node-tpl init

# Or clone the starter template
git clone https://github.com/n8n-io/n8n-nodes-starter.git n8n-nodes-myapi
cd n8n-nodes-myapi
npm install
```

---

### Step 2 — Understand the Project Structure

```
n8n-nodes-myapi/
├── credentials/
│   ├── MyApiCredentials.credentials.ts   # Auth config
│   └── icon.svg                          # Credential icon
├── nodes/
│   └── MyApi/
│       ├── MyApi.node.ts                 # Main node logic
│       └── icon.svg                      # Node icon (40x40 SVG)
├── package.json                          # npm package config
├── tsconfig.json                         # TypeScript config
├── gulpfile.js                           # Build tasks (icon copying)
└── README.md
```

---

### Step 3 — Define Credentials (credentials/MyApiCredentials.credentials.ts)

```typescript
import { ICredentialType, INodeProperties } from 'n8n-workflow';

export class MyApiCredentials implements ICredentialType {
  name = 'myApiCredentials';
  displayName = 'My API Credentials';
  documentationUrl = 'https://docs.myapi.com/auth';
  properties: INodeProperties[] = [
    {
      displayName: 'API Key',
      name: 'apiKey',
      type: 'string',
      typeOptions: { password: true },
      default: '',
      required: true,
    },
    {
      displayName: 'Base URL',
      name: 'baseUrl',
      type: 'string',
      default: 'https://api.myservice.com',
    },
  ];
}
```

---

### Step 4 — Define the Node (nodes/MyApi/MyApi.node.ts)

```typescript
import {
  IExecuteFunctions,
  INodeExecutionData,
  INodeType,
  INodeTypeDescription,
  NodeOperationError,
} from 'n8n-workflow';

export class MyApi implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'My API',
    name: 'myApi',
    icon: 'file:icon.svg',
    group: ['transform'],
    version: 1,
    description: 'Interact with My API',
    defaults: { name: 'My API' },
    inputs: ['main'],
    outputs: ['main'],
    credentials: [
      {
        name: 'myApiCredentials',
        required: true,
      },
    ],
    properties: [
      {
        displayName: 'Operation',
        name: 'operation',
        type: 'options',
        noDataExpression: true,
        options: [
          { name: 'Get Record', value: 'getRecord', action: 'Get a record' },
          { name: 'Create Record', value: 'createRecord', action: 'Create a record' },
        ],
        default: 'getRecord',
      },
      {
        displayName: 'Record ID',
        name: 'recordId',
        type: 'string',
        required: true,
        default: '',
        displayOptions: { show: { operation: ['getRecord'] } },
        description: 'The ID of the record to retrieve',
      },
    ],
  };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const items = this.getInputData();
    const returnData: INodeExecutionData[] = [];
    const credentials = await this.getCredentials('myApiCredentials');
    const operation = this.getNodeParameter('operation', 0) as string;

    for (let i = 0; i < items.length; i++) {
      try {
        if (operation === 'getRecord') {
          const recordId = this.getNodeParameter('recordId', i) as string;
          const response = await this.helpers.request({
            method: 'GET',
            url: `${credentials.baseUrl}/records/${recordId}`,
            headers: { 'X-API-Key': credentials.apiKey },
            json: true,
          });
          returnData.push({ json: response });
        }
      } catch (error) {
        if (this.continueOnFail()) {
          returnData.push({ json: { error: (error as Error).message } });
        } else {
          throw new NodeOperationError(this.getNode(), error as Error, { itemIndex: i });
        }
      }
    }
    return [returnData];
  }
}
```

---

### Step 5 — Register in package.json

```json
{
  "name": "n8n-nodes-myapi",
  "version": "1.0.0",
  "description": "n8n community node for My API",
  "keywords": ["n8n-community-node-package"],
  "main": "index.js",
  "scripts": {
    "build": "tsc && gulp build:icons",
    "dev": "tsc --watch",
    "lint": "eslint nodes credentials --ext .ts"
  },
  "n8n": {
    "n8nNodesApiVersion": 1,
    "credentials": ["dist/credentials/MyApiCredentials.credentials.js"],
    "nodes": ["dist/nodes/MyApi/MyApi.node.js"]
  },
  "devDependencies": {
    "n8n-workflow": "*",
    "typescript": "^5.0.0"
  }
}
```

> ⚠️ Package name MUST start with `n8n-nodes-` or `@scope/n8n-nodes-`. The `n8n-community-node-package` keyword MUST be in `keywords`.

---

### Step 6 — Build & Test Locally

```bash
# Build the TypeScript
npm run build

# Link to local n8n for testing
npm link

# In your local n8n installation directory
npm link n8n-nodes-myapi

# Start n8n in development mode
n8n start

# Your node now appears in the n8n node panel
```

---

### Step 7 — Publish to npm

```bash
# Login to npm
npm login

# Publish publicly
npm publish --access public

# Publish a specific version
npm version patch   # 1.0.0 → 1.0.1
npm publish
```

> ⚠️ **From 1 May 2026:** If submitting for n8n Creator Portal verification, you must publish using a GitHub Actions workflow with a provenance statement. Direct local machine publishing is not accepted for verified nodes.

**GitHub Actions publish workflow (`.github/workflows/publish.yml`):**
```yaml
name: Publish to npm
on:
  push:
    tags: ['v*']
jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          registry-url: 'https://registry.npmjs.org'
      - run: npm ci
      - run: npm run build
      - run: npm publish --access public --provenance
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

### Step 8 — Submit for n8n Verification (Optional)

1. Ensure node passes all linting: `npm run lint`
2. Publish to npm with provenance (GitHub Actions)
3. Ensure README is complete and accurate
4. Sign in to the [n8n Creator Portal](https://creators.n8n.io)
5. Submit your node for review
6. n8n team reviews for technical and UX standards
7. Once approved, node appears in the node panel search under "More from the community"

**Verification requirements:**
- Package name starts with `n8n-nodes-` or `@scope/n8n-nodes-`
- Includes `n8n-community-node-package` in keywords
- No runtime npm dependencies (only devDependencies)
- Passes the linter
- Published with GitHub Actions provenance
- Has clear documentation (README)

---

## Part 3 — Troubleshooting Community Nodes

| Problem | Cause | Fix |
|---------|-------|-----|
| Node not appearing after install | n8n not restarted | Restart n8n instance |
| "Cannot find module" error | Dependency issue | Check that devDependencies are used, not dependencies |
| Node shows but throws errors | Version mismatch | Check n8n version compatibility |
| Breaking change after update | Semver not respected | Downgrade: uninstall → reinstall specific version |
| Install blocked | Node is on blocklist | Use HTTP Request node as alternative |
| Two nodes with same name | Multiple packages | Check node icon — community nodes have a package icon |
| Queue mode — GUI install fails | Queue mode restriction | Use manual CLI install instead |

---

## Summary — Decision Guide

```
Need a node for a service?
│
├── Is it in n8n's 300+ built-in nodes?
│   └── YES → Use the built-in node
│
├── Is there a verified community node?
│   └── YES → Install via node panel (works on Cloud + self-hosted)
│
├── Is there an unverified community node?
│   └── YES → Self-hosted only → Install via Settings → Community Nodes
│
├── Does the service have an API?
│   └── YES → Use HTTP Request node (always works, no install needed)
│
└── Need reusable, shareable integration?
    └── Build a custom node → publish to npm
```
