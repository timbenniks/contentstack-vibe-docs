# Contentstack CLI Plugin Development Guide for AI Agents

This guide covers developing external plugins for the Contentstack CLI, based on real-world implementation patterns and best practices. It is written for AI coding assistants to help implement CLI plugins quickly and accurately.

## Introduction

The Contentstack CLI supports modular extensibility via external plugins, which allow you to:

- Extend CLI functionality with custom commands
- Integrate with Contentstack APIs (Management API, Delivery API)
- Be installed globally or locally
- Follow consistent CLI patterns and conventions

Plugins are built using the **oclif** framework and integrate seamlessly with the Contentstack CLI ecosystem.

### What This Guide Covers

This guide walks you through:

- Setting up a plugin project from scratch
- Understanding the plugin architecture
- Implementing commands with proper authentication and region handling
- Testing your plugin
- Publishing and distributing your plugin
- Best practices and common patterns

### Reference Plugins

Study these official plugins for real-world patterns and best practices:

- **`@contentstack/apps-cli`** - [GitHub](https://github.com/contentstack/contentstack-apps-cli) - Manages Contentstack Apps with multiple commands
- **`@contentstack/cli-tsgen`** - [GitHub](https://github.com/contentstack/contentstack-cli-tsgen) - Generates TypeScript types from stacks
- **`@contentstack/cli-content-type`** - [GitHub](https://github.com/contentstack/contentstack-cli-content-type) - Content type management utilities
- **`@contentstack/cli-plugin-openapi`** - [GitHub](https://github.com/timbenniks/contentstack-cli-plugin-openapi) - This repository - OpenAPI spec generation

---

## Prerequisites

Before you start developing a Contentstack CLI plugin, ensure you have:

1. **Node.js v20.x or higher** (Only active or LTS versions are supported for security reasons)
2. **Familiarity with TypeScript** (recommended) or JavaScript
3. **Basic understanding of the oclif CLI framework**
4. **Contentstack CLI installed globally**:
   ```bash
   npm install -g @contentstack/cli
   ```
5. **A Contentstack account** with access to stacks for testing

---

## Plugin Structure

### Recommended Directory Layout

Based on patterns from official plugins like `@contentstack/apps-cli`, here's the recommended structure:

```
my-plugin/
├── bin/                       # Executable scripts (optional)
│   └── run                    # Entry point script
├── src/
│   ├── commands/
│   │   └── myplugin/
│   │       ├── do.ts          # Command implementation
│   │       └── list.ts        # Additional commands
│   ├── services/              # Service layer (optional, for complex plugins)
│   │   ├── api-service.ts     # API integration logic
│   │   └── data-service.ts    # Data processing logic
│   ├── utils/
│   │   ├── helper.ts          # Utility functions
│   │   ├── api-client.ts      # API client wrappers
│   │   └── validators.ts      # Input validation utilities
│   └── index.ts               # Plugin entry point
├── test/
│   └── commands/
│       └── myplugin/
│           └── do.test.ts     # Command tests
├── .github/                   # GitHub Actions workflows (CI/CD)
│   └── workflows/
│       └── ci.yml
├── lib/                       # Compiled output (auto-generated)
├── .editorconfig              # Editor configuration
├── .eslintrc.js               # ESLint configuration
├── .eslintignore               # ESLint ignore patterns
├── .gitignore
├── .mocharc.json              # Mocha test configuration
├── .snyk                       # Snyk security configuration (optional)
├── package.json
├── tsconfig.json
├── README.md
├── SECURITY.md                # Security policy (recommended)
└── oclif.manifest.json        # Auto-generated manifest
```

**Key additions from official plugins:**

- **`bin/`** - Executable scripts (used in `apps-cli`)
- **`src/services/`** - Service layer for complex business logic separation
- **`.github/workflows/`** - CI/CD automation
- **`.editorconfig`** - Consistent code formatting
- **`.mocharc.json`** - Mocha test configuration
- **`.snyk`** - Security vulnerability scanning
- **`SECURITY.md`** - Security policy and reporting

### Key Files Explained

- **`src/commands/`** - Contains all command implementations
- **`src/services/`** - Service layer for complex business logic (separates concerns)
- **`src/utils/`** - Shared utility functions and helpers
- **`src/index.ts`** - Plugin entry point that exports commands
- **`bin/`** - Executable scripts (if needed for direct execution)
- **`test/`** - Unit and integration tests
- **`lib/`** - Compiled JavaScript output (do not edit directly)
- **`.github/workflows/`** - CI/CD automation workflows
- **`oclif.manifest.json`** - Auto-generated command manifest

### Service Layer Pattern

For complex plugins (like `apps-cli`), consider organizing business logic into a service layer:

```
src/
├── commands/
│   └── myplugin/
│       └── do.ts              # Thin command layer
├── services/
│   ├── api-service.ts          # API calls and data fetching
│   ├── transform-service.ts   # Data transformation logic
│   └── validation-service.ts  # Business rule validation
└── utils/
    └── helpers.ts             # Pure utility functions
```

This pattern separates:

- **Commands** - Handle CLI I/O, flags, user interaction
- **Services** - Contain business logic and API interactions
- **Utils** - Pure, reusable helper functions

---

## Development Steps

### Step 1: Bootstrap the Plugin

You have two options for creating a new plugin:

#### Option A: Using Contentstack CLI (Recommended)

The Contentstack CLI provides an interactive plugin generator:

```bash
csdx plugins:create
```

This command will prompt you for:

- **Plugin Name**: A unique name for your plugin
- **Description**: Brief description of functionality
- **Version**: Initial version (default: `0.0.0`)
- **License**: License type (default: `MIT`)
- **GitHub Repository**: Owner and repository name
- **Preferences**:
  - Use TypeScript? (recommended)
  - Use ESLint?
  - Use Mocha for testing?

This generates a complete boilerplate with proper structure, configuration files, and a sample command.

#### Option B: Using oclif directly

Alternatively, use oclif directly:

```bash
npx oclif generate @contentstack/myplugin
cd myplugin
```

This creates a basic plugin structure with:

- TypeScript configuration
- Basic command scaffolding
- Test setup
- Package.json with oclif configuration

**Note**: The `csdx plugins:create` command is preferred as it includes Contentstack-specific configurations and best practices.

### Step 2: Configure package.json

Update your `package.json` with the correct plugin configuration:

```json
{
  "name": "@contentstack/myplugin",
  "version": "1.0.0",
  "description": "Your plugin description",
  "author": "Your Name",
  "license": "MIT",
  "main": "lib/index.js",
  "types": "lib/index.d.ts",
  "oclif": {
    "commands": "./lib/commands",
    "bin": "csdx",
    "plugins": []
  },
  "files": ["/lib", "/oclif.manifest.json"],
  "scripts": {
    "build": "tsc",
    "prepack": "npm run build && oclif manifest",
    "test": "mocha --forbid-only \"test/**/*.test.ts\"",
    "lint": "eslint . --ext .ts",
    "lint:fix": "eslint . --ext .ts --fix"
  },
  "dependencies": {
    "@contentstack/cli-command": "^1.6.1",
    "@contentstack/cli-utilities": "^1.14.4",
    "@contentstack/management": "^1.20.0",
    "@oclif/core": "^3.0.0"
  },
  "devDependencies": {
    "@oclif/test": "^3.0.0",
    "@types/node": "^20.11.0",
    "@typescript-eslint/eslint-plugin": "^6.19.0",
    "@typescript-eslint/parser": "^6.19.0",
    "eslint": "^8.56.0",
    "eslint-config-oclif": "^5.0.0",
    "mocha": "^10.3.0",
    "ts-node": "^10.9.2",
    "typescript": "^5.3.3"
  },
  "engines": {
    "node": ">=20.0.0"
  }
}
```

**Key points:**

- Use `@contentstack/` namespace for official plugins
- Set `bin: "csdx"` to integrate with Contentstack CLI
- Include `prepack` script to build and generate manifest
- Add Contentstack dependencies: `@contentstack/cli-command` and `@contentstack/cli-utilities`

### Step 3: Install Dependencies

```bash
npm install
```

### Step 4: Generate a Command

Create a new command:

```bash
npx oclif command myplugin:do
```

This creates `src/commands/myplugin/do.ts` with a basic command structure.

### Step 5: Configure TypeScript

Create or update `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "lib": ["ES2022"],
    "declaration": true,
    "outDir": "./lib",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "types": ["node", "mocha"]
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "lib", "test"]
}
```

---

## Plugin Registration

### Local Development Testing

Link your plugin locally for development:

```bash
# From your plugin directory
csdx plugins:link .
```

Verify your command is registered:

```bash
csdx myplugin:do --help
```

### Unlinking

To unlink a plugin:

```bash
csdx plugins:unlink myplugin
```

---

## Commands & Flags

### Basic Command Structure

Commands should extend `Command` from `@contentstack/cli-command` (not `@oclif/core`) to get access to Contentstack-specific features:

```typescript
import { Command } from "@contentstack/cli-command";
import { flags, FlagInput } from "@contentstack/cli-utilities";

export default class MyCommand extends Command {
  static description = "Does something cool with Contentstack";

  static examples = [
    "$ csdx myplugin:do -s blt1234567890abcdef --token-alias my-token",
    "$ csdx myplugin:do -s blt1234567890abcdef -t my-token -r eu",
  ];

  static flags: FlagInput = {
    stack: flags.string({
      char: "s",
      description: "Stack API key",
      required: true,
    }),
    "token-alias": flags.string({
      char: "t",
      description: "Management token alias",
      required: true,
    }),
    region: flags.string({
      char: "r",
      description: "Contentstack region",
    }),
    // Add more flags as needed
  };

  async run(): Promise<void> {
    const { flags } = await this.parse(MyCommand);

    try {
      // Your command logic here
      this.log(`Working with stack: ${flags.stack}`);

      // Access Contentstack utilities
      const managementToken = this.getToken(flags["token-alias"]);
      const region = this.region;
      const cmaHost = this.cmaHost;
      const cdaHost = this.cdaHost;
    } catch (error) {
      this.error(
        `Failed: ${error instanceof Error ? error.message : String(error)}`,
        { exit: 1 }
      );
    }
  }
}
```

### Using Contentstack CLI Command Base Class

The `@contentstack/cli-command` base class provides:

- **`this.region`** - Current region configuration
- **`this.cmaHost`** - Management API host for current region
- **`this.cdaHost`** - Delivery API host for current region
- **`this.getToken(alias)`** - Get management token by alias
- **`this.log(message)`** - Logging with proper formatting
- **`this.warn(message)`** - Warning messages
- **`this.error(message, options)`** - Error handling

### Interactive Prompts

For commands that need user input, use `ux` from `@oclif/core`:

```typescript
import { ux } from '@oclif/core';

async run(): Promise<void> {
  const { flags } = await this.parse(MyCommand);

  // Prompt for missing required values
  const stackKey = flags.stack || await ux.prompt('Enter stack API key');
  const tokenAlias = flags['token-alias'] || await ux.prompt('Enter token alias');

  // Confirmation prompts
  const confirmed = await ux.confirm('Are you sure you want to proceed?');
  if (!confirmed) {
    this.log('Operation cancelled');
    return;
  }

  // Select from options
  const mode = await ux.prompt('Select mode', {
    type: 'select',
    options: ['delivery', 'management']
  });

  // Password input (hidden)
  const password = await ux.prompt('Enter password', { type: 'hide' });
}
```

**Best Practice**: Always allow flags to override prompts for automation and CI/CD compatibility.

### Flag Types

Common flag types from `@contentstack/cli-utilities`:

```typescript
import { flags, FlagInput } from '@contentstack/cli-utilities';

static flags: FlagInput = {
  // String flag
  stack: flags.string({
    char: 's',
    description: 'Stack API key',
    required: true
  }),

  // Boolean flag
  verbose: flags.boolean({
    char: 'v',
    description: 'Verbose output'
  }),

  // Enum flag (options)
  mode: flags.string({
    char: 'm',
    description: 'API mode',
    options: ['delivery', 'management'],
    default: 'delivery'
  }),

  // Integer flag
  limit: flags.integer({
    char: 'l',
    description: 'Limit results',
    default: 100
  }),
};
```

### Use Namespacing

**Always prefix your commands** to avoid collisions:

**Good**: `myplugin:do`, `myplugin:list`, `myplugin:generate`
**Bad**: `do`, `list`, `generate`

The namespace should match your plugin name (without the `@contentstack/cli-plugin-` prefix).

---

## Authentication & Configuration

### Management Token Handling

Contentstack CLI plugins use management token aliases for authentication. Here's how to handle them:

```typescript
import { configHandler } from "@contentstack/cli-utilities";

function getManagementToken(command: Command, tokenAlias?: string): string {
  if (tokenAlias) {
    try {
      const tokenData = command.getToken(tokenAlias);
      if (typeof tokenData === "object" && tokenData !== null) {
        return (
          (tokenData as { token?: string; authtoken?: string }).authtoken ||
          (tokenData as { token?: string; authtoken?: string }).token ||
          ""
        );
      }
      if (typeof tokenData === "string") return tokenData;
    } catch {
      // Fall through to default
    }
  }

  // Fallback to environment variables or config
  return (
    configHandler.get("authtoken") ||
    process.env.CS_AUTHTOKEN ||
    process.env.CONTENTSTACK_MANAGEMENT_TOKEN ||
    ""
  );
}
```

### Region Configuration

The `@contentstack/cli-command` base class automatically handles region configuration and endpoints. It uses the official [Contentstack Regions JSON](https://artifacts.contentstack.com/regions.json) as the source of truth internally, so you don't need to manually fetch or manage region data.

Simply use the built-in properties:

```typescript
async run(): Promise<void> {
  const { flags } = await this.parse(MyCommand);

  // Region is automatically configured from CLI config
  // Access region info
  const region = this.region; // Current region configuration

  // Endpoints are automatically set based on region
  const cmaHost = this.cmaHost; // Management API host for current region
  const cdaHost = this.cdaHost; // Delivery API host for current region

  // Use endpoints directly
  const client = createCMAClient(flags.stack, managementToken, cmaHost);
}
```

**Supported Regions**: The CLI supports all Contentstack regions including:

- AWS: `na` (default), `eu`, `au`
- Azure: `azure-na`, `azure-eu`
- GCP: `gcp-na`, `gcp-eu`

**Note**: The base class handles all region aliases (e.g., `us`, `na`, `aws-na` all map correctly) and endpoint resolution automatically. You only need to manually handle regions if you're overriding the default behavior or need to support a custom region flag.

### Using the Management API Client

```typescript
import { client } from "@contentstack/management";
import type { Stack } from "@contentstack/management/types/stack";

// Create a CMA client
function createCMAClient(
  apiKey: string,
  authtoken: string,
  cmaHost: string
): Stack {
  return client({
    authtoken,
    host: cmaHost,
  }).stack({
    api_key: apiKey,
    management_token: authtoken,
  });
}

// Use in your command
const stack = createCMAClient(flags.stack, managementToken, cmaHost);
const contentTypes = await stack.contentType().query().find();
```

---

## Testing

### Testing Setup

Use `@oclif/test` with Mocha for testing:

```typescript
import { expect } from "chai";
import { test } from "@oclif/test";
import MyCommand from "../../src/commands/myplugin/do";

describe("myplugin:do", () => {
  test
    .stdout()
    .command(["myplugin:do", "--stack", "dummy_key", "--token-alias", "test"])
    .it("runs myplugin:do", (ctx) => {
      expect(ctx.stdout).to.contain("Working with stack: dummy_key");
    });

  test
    .stdout()
    .stderr()
    .command(["myplugin:do"])
    .catch(/required/i)
    .it("requires stack flag");
});
```

### Running Tests

```bash
npm test
```

### Testing Patterns

**1. Test flag validation:**

```typescript
test
  .command(["myplugin:do"])
  .catch(/required/i)
  .it("requires required flags");
```

**2. Test successful execution:**

```typescript
test
  .stdout()
  .stub(MyCommand.prototype, "run", () => Promise.resolve())
  .command(["myplugin:do", "--stack", "test"])
  .it("executes successfully");
```

**3. Test error handling:**

```typescript
test
  .stderr()
  .command(["myplugin:do", "--stack", "invalid"])
  .catch(/error/i)
  .it("handles errors gracefully");
```

### Production Testing Checklist

Before publishing, test your plugin:

1. **Install CLI globally:**

   ```bash
   npm i -g @contentstack/cli
   ```

2. **Configure region:**

   ```bash
   csdx config:set:region <region-name>
   ```

3. **Login:**

   ```bash
   csdx login
   ```

4. **Link plugin:**

   ```bash
   csdx plugins:link <plugin-local-path>
   ```

5. **Test commands:**
   ```bash
   csdx myplugin:do --help
   csdx myplugin:do -s <stack-key> -t <token-alias>
   ```

---

## Plugin Entry Point

Your `src/index.ts` should export your commands:

```typescript
// Simple export
export { default } from "./commands/myplugin/do";

// Or export multiple commands
export { default as Do } from "./commands/myplugin/do";
export { default as List } from "./commands/myplugin/list";
```

---

## Publishing the Plugin

### Pre-Publishing Checklist

1. Build the project: `npm run build`
2. Run tests: `npm test`
3. Lint code: `npm run lint`
4. Update version in `package.json`
5. Update `README.md` with usage examples
6. Ensure `oclif.manifest.json` is generated (happens automatically in `prepack`)
7. Add `SECURITY.md` file (recommended for public packages)
8. Review and update dependencies for security vulnerabilities
9. Test installation: `npm pack` and verify contents

### Security Considerations

Before publishing, ensure security best practices:

1. **Add SECURITY.md**:

   ```markdown
   # Security Policy

   ## Supported Versions

   | Version | Supported          |
   | ------- | ------------------ |
   | 1.x.x   | :white_check_mark: |
   | < 1.0   | :x:                |

   ## Reporting a Vulnerability

   Please report vulnerabilities to security@contentstack.com
   ```

2. **Use Snyk for vulnerability scanning**:

   ```bash
   npm install -g snyk
   snyk test
   snyk wizard  # Interactive setup
   ```

   This creates a `.snyk` file for continuous monitoring.

3. **Review dependencies**:
   ```bash
   npm audit
   npm audit fix
   ```

### Publishing Steps

1. **Login to npm:**

   ```bash
   npm login
   ```

2. **Publish:**

   ```bash
   npm publish --access public
   ```

   Note: For scoped packages (`@contentstack/...`), you may need `--access public`.

3. **Install via CLI:**
   ```bash
   csdx plugins:install @contentstack/myplugin
   ```

### Version Management

Follow semantic versioning:

- **Patch** (`1.0.1`): Bug fixes
- **Minor** (`1.1.0`): New features, backward compatible
- **Major** (`2.0.0`): Breaking changes

---

## Best Practices

### Do's

| Practice                    | Description                                          |
| --------------------------- | ---------------------------------------------------- |
| **Use namespacing**         | Prefix commands like `myplugin:action`               |
| **Follow oclif standards**  | Maintain command/flag conventions                    |
| **Use proper CLI feedback** | Use `this.log`, `this.error`, `ux.prompt`            |
| **Validate inputs**         | Check required flags/args early                      |
| **Add tests**               | Include basic tests for every command                |
| **Document commands**       | Add descriptions, usage, and examples                |
| **Use Contentstack SDKs**   | Prefer official SDKs like `@contentstack/management` |
| **Respect user configs**    | Use `~/.csdx/config.json` when needed                |
| **Log errors gracefully**   | Use clear error messages and hints                   |
| **Handle regions properly** | Support all Contentstack regions                     |
| **Use TypeScript**          | Better type safety and developer experience          |

### Don'ts

| Practice                              | Reason                          |
| ------------------------------------- | ------------------------------- |
| **Don't overwrite global configs**    | Avoid altering shared state     |
| **Don't hardcode values**             | Make plugins configurable       |
| **Don't break existing flows**        | Avoid side effects in CLI       |
| **Don't ignore security**             | Never log sensitive info        |
| **Don't bypass CLI output patterns**  | Ensure UX consistency           |
| **Don't skip error handling**         | Always handle errors gracefully |
| **Don't ignore region configuration** | Support all regions properly    |

---

## Common Patterns

### Pattern 1: Fetching Content Types

```typescript
import type { ContentType } from "@contentstack/management/types/stack/contentType";
import type { ContentstackCollection } from "@contentstack/management/types/contentstackCollection";

export async function fetchContentTypes(stack: Stack): Promise<ContentType[]> {
  const contentTypes: ContentType[] = [];
  let skip = 0;
  const limit = 100;
  let hasMore = true;

  while (hasMore) {
    const response: ContentstackCollection<ContentType> = await stack
      .contentType()
      .query({
        skip,
        limit,
      })
      .find();

    if (response.items && response.items.length > 0) {
      contentTypes.push(...response.items);
      skip += response.items.length;
      hasMore = response.items.length === limit;
    } else {
      hasMore = false;
    }
  }

  return contentTypes;
}
```

### Pattern 2: Progress Feedback

```typescript
async run(): Promise<void> {
  this.log('Starting operation...');

  this.log('Fetching data...');
  const data = await fetchData();
  this.log(`Found ${data.length} item(s)`);

  this.log('Processing...');
  await processData(data);

  this.log('Operation completed successfully');
}
```

### Pattern 3: Error Handling

```typescript
async run(): Promise<void> {
  try {
    // Your logic
  } catch (error) {
    if (error instanceof Error) {
      if (error.message.includes('authentication')) {
        this.error(
          'Authentication failed. Please check your token alias.',
          { exit: 1 }
        );
      } else if (error.message.includes('not found')) {
        this.error(
          `Resource not found: ${error.message}`,
          { exit: 1 }
        );
      } else {
        this.error(
          `Failed: ${error.message}`,
          { exit: 1 }
        );
      }
    } else {
      this.error(
        `Failed: ${String(error)}`,
        { exit: 1 }
      );
    }
  }
}
```

### Pattern 4: File Operations

```typescript
import * as fs from "fs-extra";
import * as path from "path";

// Ensure directory exists
const outputPath = path.resolve(flags.out);
await fs.ensureDir(path.dirname(outputPath));

// Write JSON
await fs.writeJson(outputPath, data, { spaces: 2 });

// Read JSON
const data = await fs.readJson(inputPath);
```

### Pattern 5: Region Handling

Use the built-in region properties from `@contentstack/cli-command`:

```typescript
async run(): Promise<void> {
  const { flags } = await this.parse(MyCommand);

  // Region and endpoints are automatically available
  const cmaHost = this.cmaHost; // Already configured for current region
  const cdaHost = this.cdaHost; // Already configured for current region
  const region = this.region;   // Current region info

  // Use endpoints directly - no manual fetching needed
  const client = createCMAClient(flags.stack, managementToken, cmaHost);
}
```

**Key Points:**

- `@contentstack/cli-command` handles all region resolution internally
- Endpoints are automatically set based on CLI region configuration
- No need to manually fetch or parse regions.json
- The base class uses the official [regions.json](https://artifacts.contentstack.com/regions.json) as source of truth

### Pattern 6: Configuration File Handling

Many plugins (like `apps-cli`) support external configuration files:

```typescript
import * as fs from "fs-extra";
import * as path from "path";
import { flags } from "@contentstack/cli-utilities";

static flags: FlagInput = {
  config: flags.string({
    char: "c",
    description: "Path to configuration file",
  }),
};

async run(): Promise<void> {
  const { flags } = await this.parse(MyCommand);

  let config = {};

  // Load config from file if provided
  if (flags.config) {
    const configPath = path.resolve(flags.config);
    if (await fs.pathExists(configPath)) {
      config = await fs.readJson(configPath);
    } else {
      this.error(`Config file not found: ${configPath}`, { exit: 1 });
    }
  }

  // Merge config with flags (flags take precedence)
  const finalConfig = { ...config, ...flags };
}
```

### Pattern 7: Service Layer Organization

For complex plugins, separate business logic into services:

```typescript
// src/services/api-service.ts
import { Stack } from "@contentstack/management/types/stack";

export class ApiService {
  constructor(private stack: Stack) {}

  async fetchContentTypes() {
    // API logic here
  }

  async createContentType(data: any) {
    // API logic here
  }
}

// src/commands/myplugin/do.ts
import { ApiService } from "../../services/api-service";

async run(): Promise<void> {
  const { flags } = await this.parse(MyCommand);

  const apiService = new ApiService(stack);
  const contentTypes = await apiService.fetchContentTypes();

  // Command logic here
}
```

### Pattern 8: Progress Indicators

For long-running operations, show progress:

```typescript
import { ux } from "@oclif/core";

async run(): Promise<void> {
  const items = await fetchItems();

  // Show progress bar
  ux.action.start("Processing items");

  for (let i = 0; i < items.length; i++) {
    await processItem(items[i]);
    ux.action.status = `Processing ${i + 1}/${items.length}`;
  }

  ux.action.stop("Complete");

  // Or use spinner for indeterminate progress
  const spinner = ux.spinner();
  spinner.start("Fetching data");
  const data = await fetchData();
  spinner.stop("Data fetched");
}
```

---

## Continuous Integration (CI/CD)

### GitHub Actions Setup

Official plugins use GitHub Actions for CI/CD. Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [20.x]

    steps:
      - uses: actions/checkout@v3

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Build
        run: npm run build

      - name: Test
        run: npm test

      - name: Security audit
        run: npm audit --audit-level=moderate
        continue-on-error: true
```

### Pre-commit Hooks

Consider using `husky` for pre-commit checks:

```bash
npm install --save-dev husky
npx husky install
npx husky add .husky/pre-commit "npm run lint && npm test"
```

---

## Resources

### Official Documentation

- [oclif Documentation](https://oclif.io/docs)
- [Contentstack CLI Docs](https://www.contentstack.com/docs/developers/cli)
- [Configure Regions in CLI](https://www.contentstack.com/docs/developers/cli/configure-regions)
- [CLI Authentication Guide](https://www.contentstack.com/docs/developers/cli/authentication)

### SDKs and Libraries

- [Contentstack Management SDK](https://www.npmjs.com/package/@contentstack/management)
- [Contentstack CLI Command](https://www.npmjs.com/package/@contentstack/cli-command)
- [Contentstack CLI Utilities](https://www.npmjs.com/package/@contentstack/cli-utilities)

### Example Plugins (Study These!)

- **[@contentstack/apps-cli](https://github.com/contentstack/contentstack-apps-cli)** - Complex plugin with multiple commands, services layer, and interactive prompts
- **[@contentstack/cli-tsgen](https://github.com/contentstack/contentstack-cli-tsgen)** - TypeScript generation plugin with file operations
- **[@contentstack/cli-content-type](https://github.com/contentstack/contentstack-cli-content-type)** - Content type management utilities
- **[@contentstack/cli-plugin-openapi](https://github.com/contentstack/contentstack-cli-plugin-openapi)** - This repository - OpenAPI spec generation

### API Documentation

- [Contentstack Management API](https://www.contentstack.com/docs/developers/apis/content-management-api)
- [Contentstack Delivery API](https://www.contentstack.com/docs/developers/apis/content-delivery-api)

---

## Troubleshooting

### Common Issues

#### Issue: Command not found after linking

**Solution:**

```bash
# Rebuild the plugin
npm run build

# Relink
csdx plugins:unlink myplugin
csdx plugins:link .
```

#### Issue: Authentication errors

**Solution:**

```bash
# Check token alias exists
csdx config:get tokens

# Set token via environment variable (fallback)
export CS_AUTHTOKEN=your-management-token
```

#### Issue: Region not found

**Solution:**

```bash
# Set region explicitly (supports aliases like 'us', 'na', 'eu', 'azure-na', etc.)
csdx config:set:region eu

# Or pass via flag if your command supports it
csdx myplugin:do -s <key> -t <alias> -r eu
```

**Note**: The `@contentstack/cli-command` base class handles all region aliases automatically. The CLI uses the [official regions.json](https://artifacts.contentstack.com/regions.json) internally, so region resolution happens automatically.

#### Issue: TypeScript compilation errors

**Solution:**

- Ensure all dependencies are installed: `npm install`
- Check `tsconfig.json` configuration
- Verify Node.js version: `node --version` (should be >= 20.0.0)

#### Issue: Manifest not generated

**Solution:**

```bash
# Generate manifest manually
npx oclif manifest

# Or run prepack
npm run prepack
```

---

## Checklist for New Plugins

Use this checklist when creating a new plugin:

### Initial Setup

- [ ] Project bootstrapped with `csdx plugins:create` or `oclif generate`
- [ ] `package.json` configured with correct name and oclif settings
- [ ] TypeScript configured properly (`tsconfig.json`)
- [ ] ESLint configured (`.eslintrc.js`, `.eslintignore`)
- [ ] Editor config added (`.editorconfig`)
- [ ] Git ignore configured (`.gitignore`)

### Code Structure

- [ ] Commands use `@contentstack/cli-command` base class
- [ ] Commands are properly namespaced (e.g., `myplugin:do`)
- [ ] Service layer created for complex logic (if needed)
- [ ] Utilities organized in `src/utils/`
- [ ] Plugin entry point (`src/index.ts`) exports commands

### Functionality

- [ ] Authentication handled via token aliases
- [ ] Region support implemented
- [ ] Error handling in place with helpful messages
- [ ] Interactive prompts for missing inputs (if needed)
- [ ] Configuration file support (if needed)

### Testing & Quality

- [ ] Tests written and passing (`test/` directory)
- [ ] Mocha configured (`.mocharc.json`)
- [ ] Code linted and formatted
- [ ] No security vulnerabilities (`npm audit`)
- [ ] Snyk configured (`.snyk` file, optional)

### Documentation

- [ ] README.md with usage examples
- [ ] Command descriptions and examples added
- [ ] SECURITY.md file added (for public packages)
- [ ] Code comments for complex logic

### CI/CD

- [ ] GitHub Actions workflow configured (`.github/workflows/ci.yml`)
- [ ] Pre-commit hooks set up (optional, using husky)

### Publishing

- [ ] Plugin links and works locally (`csdx plugins:link .`)
- [ ] Tested with real Contentstack stack
- [ ] Version updated in `package.json`
- [ ] Ready for publishing

---

## Tips & Learnings from Official Plugins

### Development Tips

1. **Start Simple**: Begin with a basic command and add complexity gradually
2. **Test Early**: Write tests as you develop, not after
3. **Use Examples**: Include clear examples in command descriptions
4. **Handle Edge Cases**: Consider empty stacks, missing data, network errors
5. **Provide Feedback**: Use `this.log()` and progress indicators to keep users informed
6. **Follow Conventions**: Match the patterns used in official plugins
7. **Document Everything**: Clear documentation helps users and maintainers

### Patterns from Official Plugins

**From `apps-cli`:**

- Use service layer for complex business logic
- Support external configuration files
- Implement interactive prompts for better UX
- Use `bin/` directory for executable scripts when needed

**From `cli-tsgen`:**

- Organize file operations in utilities
- Use proper TypeScript types throughout
- Handle large data sets efficiently (pagination, streaming)

**From `cli-content-type`:**

- Keep commands focused and single-purpose
- Provide clear validation and error messages
- Use consistent naming conventions

### Common Pitfalls to Avoid

1. **Don't hardcode API endpoints** - Always use `this.cmaHost` and `this.cdaHost` from the base class
2. **Don't skip error handling** - Users need helpful error messages
3. **Don't ignore security** - Never log tokens or sensitive data
4. **Don't forget pagination** - Handle large datasets properly
5. **Don't skip tests** - Even basic tests catch regressions
6. **Don't ignore CI/CD** - Automate quality checks
7. **Don't manually fetch regions.json** - Use the built-in `this.region`, `this.cmaHost`, and `this.cdaHost` properties from `@contentstack/cli-command`

---

## Contributing

When contributing to Contentstack CLI plugins:

1. Follow the coding standards and patterns outlined in this guide
2. Write tests for new features
3. Update documentation
4. Ensure backward compatibility when possible
5. Submit pull requests with clear descriptions

## Next Steps

- Review [Base Concepts](00-contentstack-base-concepts.md) to understand Contentstack fundamentals
- Learn about [API Functionalities](01-api-functionalities.md) for Management API usage
- Explore [SDK Functionalities](02-sdk-functionalities.md) for Delivery API patterns
- Check [Practical Examples](05-practical-examples.md) for implementation patterns

---

Made with ❤️ for the Contentstack community
