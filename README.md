# Contentstack Vibe Docs

An [Agent Skill](https://agentskills.io/) that gives AI coding agents comprehensive Contentstack CMS knowledge. Works with Claude Code, Cursor, GitHub Copilot, VS Code, Gemini CLI, Roo Code, and [25+ other tools](https://agentskills.io/).

> **AI Agents: Read [SKILL.md](SKILL.md).** It contains routing rules that direct you to the 1-3 files you need. Do not read all files — this skill contains ~10,000 lines across 20 reference documents.

---

## Install

### Universal (all agents)

Install with the [skills CLI](https://skills.sh/docs/cli):

```bash
npx skills add timbenniks/contentstack-vibe-docs
```

This auto-detects your installed agents (Claude Code, Cursor, Copilot, Windsurf, Cline, etc.) and installs the skill for all of them.

Install for specific agents only:

```bash
npx skills add timbenniks/contentstack-vibe-docs -a claude-code
npx skills add timbenniks/contentstack-vibe-docs -a cursor
npx skills add timbenniks/contentstack-vibe-docs -a github-copilot
```

Install globally (available in all your projects):

```bash
npx skills add timbenniks/contentstack-vibe-docs -g
```

### Claude Code

Install as a personal skill (all projects):

```bash
claude skill add timbenniks/contentstack-vibe-docs
```

Or use the skills CLI:

```bash
npx skills add timbenniks/contentstack-vibe-docs -a claude-code -g
```

Once installed, Claude Code discovers the skill automatically when you work on Contentstack tasks. You can also invoke it directly:

```
/contentstack-vibe-docs fetch blog posts with author references
/contentstack-vibe-docs set up live preview in Next.js
/contentstack-vibe-docs optimize images for responsive layouts
```

### Gemini CLI

```bash
gemini skills install https://github.com/timbenniks/contentstack-vibe-docs.git
```

Install to workspace scope:

```bash
gemini skills install https://github.com/timbenniks/contentstack-vibe-docs.git --scope workspace
```

### Manage installed skills

```bash
npx skills list              # List all installed skills
npx skills check             # Check for updates
npx skills update            # Update all skills
```

### Claude Code Plugin

This repo is also a [Claude Code plugin](https://code.claude.com/docs/en/plugins). Install it directly:

```bash
claude plugin install https://github.com/timbenniks/contentstack-vibe-docs
```

Or test locally during development:

```bash
claude --plugin-dir ./contentstack-vibe-docs
```

The plugin includes:
- **Skills** — the full Contentstack documentation skill with routing and references

### Cursor Plugin

This repo is also a [Cursor plugin](https://cursor.directory/plugins/contentstack). Install it directly in Cursor:

1. Open Cursor Settings > Plugins
2. Add this repository URL: `https://github.com/timbenniks/contentstack-vibe-docs`

The plugin includes:
- **Skills** — the full Contentstack documentation skill with routing and references
- **Rules** — `rules/contentstack.mdc` with routing table, decision helpers, security guardrails, and quick patterns

---

## How It Works

This skill uses **progressive disclosure** to keep your agent's context window clean:

1. **Discovery** (~100 tokens) — The agent loads the skill name and description. Just enough to know "this is relevant for Contentstack tasks."
2. **Activation** (~3,500 tokens) — When a Contentstack task is detected, the agent reads `SKILL.md` with its routing table, decision helpers, and inline quick-start patterns.
3. **Execution** (on-demand) — The routing table directs the agent to read only the 1-3 specific reference files it needs. It never loads all 10,000 lines.

```
User: "Add live preview to my Next.js app"
  → Agent loads SKILL.md (routing table)
  → Reads references/live-preview/concepts.md (131 lines)
  → Reads references/live-preview/ssr-mode.md (315 lines)
  → Reads references/frameworks/nextjs.md (424 lines)
  → Implements with copy-paste ready code
```

---

## What's Covered

| Topic | Reference File |
|-------|---------------|
| Quick code patterns | [QUICK_REFERENCE.md](references/QUICK_REFERENCE.md) |
| CMS fundamentals | [base-concepts.md](references/concepts/base-concepts.md) |
| Region configuration | [regions.md](references/concepts/regions.md) |
| REST Content Delivery API | [rest-api.md](references/api/rest-api.md) |
| GraphQL Content Delivery API | [graphql-api.md](references/api/graphql-api.md) |
| Content Management API (CRUD) | [content-management-api.md](references/api/content-management-api.md) |
| Image Delivery API (transforms) | [image-delivery-api.md](references/api/image-delivery-api.md) |
| TypeScript Delivery SDK | [delivery-sdk.md](references/sdk/delivery-sdk.md) |
| Live Preview (concepts) | [concepts.md](references/live-preview/concepts.md) |
| Live Preview (CSR mode) | [csr-mode.md](references/live-preview/csr-mode.md) |
| Live Preview (SSR mode) | [ssr-mode.md](references/live-preview/ssr-mode.md) |
| Next.js patterns | [nextjs.md](references/frameworks/nextjs.md) |
| Nuxt patterns | [nuxt.md](references/frameworks/nuxt.md) |
| Gatsby patterns | [gatsby.md](references/frameworks/gatsby.md) |
| OAuth with Auth.js | [oauth.md](references/authentication/oauth.md) |
| CLI plugin development | [cli-plugins.md](references/extensions/cli-plugins.md) |
| Developer Hub apps | [devhub-apps.md](references/extensions/devhub-apps.md) |
| Real-world code examples | [practical-examples.md](references/examples/practical-examples.md) |
| Package version compatibility | [VERSIONS.md](references/VERSIONS.md) |

All regions supported: US, EU, AU, Azure NA/EU, GCP NA/EU.

---

## Security & Guardrails

This skill includes built-in security measures and red flags that instruct agents to handle credentials safely:

- **No secret leaking** — Agents are explicitly told to never ask for, output, log, or hardcode API keys, tokens, or secrets. All code uses `process.env.*` references.
- **Credential hygiene** — If a developer accidentally pastes a real token in the conversation, the agent is instructed to warn them and suggest they rotate it.
- **Frontend protection** — Management Tokens are flagged as server-side only. Agents are told to never expose them in frontend code.
- **`.env` safety** — Agents are instructed to never commit `.env` files or credentials to version control.
- **Red flags checklist** — A dedicated "Red Flags" section in `SKILL.md` lists specific anti-patterns agents must avoid, from hardcoding secrets to mixing SDK patterns incorrectly.
- **Questions before code** — Agents are guided to ask about region, framework, and environment setup before writing any code, ensuring the right patterns are used from the start.

---

## Documentation Structure

This repo follows the [Open Plugins](https://open-plugins.com/plugin-builders) standard:

```
.plugin/
└── plugin.json              # Open Plugins manifest (OpenCode, etc.)

.claude-plugin/
└── plugin.json              # Claude Code plugin manifest

.cursor-plugin/
└── plugin.json              # Cursor plugin manifest

skills/
└── contentstack-vibe-docs/
    ├── SKILL.md → ../../SKILL.md        # Symlink to root skill
    └── references → ../../references    # Symlink to root references

rules/
└── contentstack.mdc         # Cursor rules (routing, helpers, security)

references/
├── QUICK_REFERENCE.md          # Condensed patterns for quick lookup
├── VERSIONS.md                 # Package version compatibility
├── concepts/
│   ├── base-concepts.md        # CMS fundamentals
│   └── regions.md              # Region configuration
├── api/
│   ├── rest-api.md             # REST Content Delivery API
│   ├── graphql-api.md          # GraphQL Content Delivery API
│   ├── content-management-api.md  # Content Management API (CRUD)
│   └── image-delivery-api.md   # Image Delivery API (transforms)
├── sdk/
│   └── delivery-sdk.md         # TypeScript SDK guide
├── live-preview/
│   ├── concepts.md             # Live Preview overview
│   ├── csr-mode.md             # ssr:false — postMessage updates
│   └── ssr-mode.md             # ssr:true — iframe refresh
├── authentication/
│   └── oauth.md                # OAuth login with Auth.js
├── frameworks/
│   ├── nextjs.md               # Next.js patterns
│   ├── nuxt.md                 # Nuxt 4 patterns
│   └── gatsby.md               # Gatsby patterns
├── extensions/
│   ├── cli-plugins.md          # CLI plugin development
│   └── devhub-apps.md          # Developer Hub apps
└── examples/
    └── practical-examples.md   # Real-world code patterns
```

---

## Support

- [Contentstack Documentation](https://www.contentstack.com/docs)
- [Discord Community](https://community.contentstack.com)
- [Contentstack Support](https://www.contentstack.com/support)
- [Agent Skills Specification](https://agentskills.io/)

---

## License

This documentation is provided as-is for use with AI coding assistants.
