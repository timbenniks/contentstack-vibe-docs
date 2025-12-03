# Contentstack Vibe Coding Documentation for AI Agents

Comprehensive documentation for building applications with Contentstack CMS. Specifically optimized for AI coding assistants to implement Contentstack features quickly and accurately.

## Quick Start

| I want to...                       | Read this                                                                    |
| ---------------------------------- | ---------------------------------------------------------------------------- |
| **Quickly look up a pattern**      | [QUICK_REFERENCE.md](.contentstack-vibe-docs/QUICK_REFERENCE.md)             |
| **Check package versions**         | [VERSIONS.md](.contentstack-vibe-docs/VERSIONS.md)                           |
| **Understand Contentstack basics** | [Base Concepts](.contentstack-vibe-docs/concepts/base-concepts.md)           |
| **Configure regions**              | [Regions Guide](.contentstack-vibe-docs/concepts/regions.md)                 |
| **Use the REST API**               | [REST API](.contentstack-vibe-docs/api/rest-api.md)                          |
| **Use GraphQL**                    | [GraphQL API](.contentstack-vibe-docs/api/graphql-api.md)                    |
| **Use the TypeScript SDK**         | [Delivery SDK](.contentstack-vibe-docs/sdk/delivery-sdk.md)                  |
| **Implement Live Preview**         | [Live Preview Concepts](.contentstack-vibe-docs/live-preview/concepts.md)    |
| **Build with Next.js**             | [Next.js Patterns](.contentstack-vibe-docs/frameworks/nextjs.md)             |
| **Build with Nuxt**                | [Nuxt Patterns](.contentstack-vibe-docs/frameworks/nuxt.md)                  |
| **Build with Gatsby**              | [Gatsby Patterns](.contentstack-vibe-docs/frameworks/gatsby.md)              |
| **Create a CLI plugin**            | [CLI Plugins](.contentstack-vibe-docs/extensions/cli-plugins.md)             |
| **Build a Developer Hub app**      | [DevHub Apps](.contentstack-vibe-docs/extensions/devhub-apps.md)             |
| **See code examples**              | [Practical Examples](.contentstack-vibe-docs/examples/practical-examples.md) |

---

## Agent Instructions

### Questions to Ask Developers First

#### Basic Setup

- **What Contentstack region?** (US, EU, AU, Azure, GCP) - Required for endpoints
- **Do you have credentials?** (API Key, Delivery Token, Preview Token)
- **What environment?** (production, staging, preview)

#### Framework & Architecture

- **What framework?** (Next.js, Nuxt, Gatsby, React, Vue)
- **Does your site fetch data client-side or server-side?** - Determines Live Preview SDK mode
- **What's the hosting?** (Vercel, Netlify, self-hosted)

#### Content Requirements

- **What content types?** - Query patterns
- **Need references included?** - `.includeReference()` usage
- **Using modular blocks?** - Rendering patterns
- **Multiple locales?** - Locale handling
- **Need image transforms?** - Asset handling

### Decision Trees

#### API vs SDK

- **Use SDK** when: JavaScript/TypeScript, need type safety, need Live Preview
- **Use REST/GraphQL directly** when: Non-JS languages, fine-grained control

#### Live Preview SDK Mode (`ssr` setting)

**Important**: This setting controls how the preview updates inside the CMS iframe - NOT your website's rendering strategy. Live Preview is never enabled in production.

- **`ssr: false`**: Uses postMessage. CMS sends data changes to iframe, client JS re-fetches and updates UI instantly (no page refresh)
- **`ssr: true`**: CMS refreshes the iframe with query params (`?live_preview=hash&entry_uid=...`). Server reads params and fetches preview data

### Implementation Workflow

1. **Gather requirements** - Ask questions above
2. **Read relevant docs** - Start with concepts if new
3. **Check examples** - Find matching patterns
4. **Implement step-by-step** - Follow patterns exactly
5. **Add error handling** - Always try-catch
6. **Use environment variables** - Never hardcode
7. **Test thoroughly** - Verify with real stack

### Red Flags ⚠️

**Never do these:**

- Hardcode API keys or tokens
- Use Management Tokens in frontend
- Skip error handling
- Mix REST and GraphQL patterns incorrectly
- Forget to include references when needed

---

## Documentation Structure

```
.contentstack-vibe-docs/
├── QUICK_REFERENCE.md      # Condensed patterns for quick lookup
├── VERSIONS.md             # Package version compatibility
├── concepts/
│   ├── base-concepts.md    # CMS fundamentals
│   └── regions.md          # Region configuration guide
├── api/
│   ├── rest-api.md         # REST API usage
│   └── graphql-api.md      # GraphQL API usage
├── sdk/
│   └── delivery-sdk.md     # TypeScript SDK guide
├── live-preview/
│   ├── concepts.md         # Live Preview overview
│   ├── csr-mode.md         # ssr:false - postMessage updates
│   └── ssr-mode.md         # ssr:true - iframe refresh with query params
├── frameworks/
│   ├── nextjs.md           # Next.js patterns
│   ├── nuxt.md             # Nuxt 4 patterns
│   └── gatsby.md           # Gatsby patterns
├── extensions/
│   ├── cli-plugins.md      # CLI plugin development
│   └── devhub-apps.md      # Developer Hub apps
└── examples/
    └── practical-examples.md  # Real-world code patterns
```

---

## Reading Order

### For New Projects

1. [Base Concepts](.contentstack-vibe-docs/concepts/base-concepts.md) - Understand the CMS
2. [Delivery SDK](.contentstack-vibe-docs/sdk/delivery-sdk.md) - Set up SDK
3. Framework guide ([Next.js](.contentstack-vibe-docs/frameworks/nextjs.md), [Nuxt](.contentstack-vibe-docs/frameworks/nuxt.md), or [Gatsby](.contentstack-vibe-docs/frameworks/gatsby.md))
4. [Live Preview Concepts](.contentstack-vibe-docs/live-preview/concepts.md) - If needed
5. [Practical Examples](.contentstack-vibe-docs/examples/practical-examples.md) - Reference patterns

### For Existing Projects

1. [QUICK_REFERENCE.md](.contentstack-vibe-docs/QUICK_REFERENCE.md) - Quick patterns
2. Specific feature docs as needed

### For Extensions

- CLI plugins: [CLI Plugins Guide](.contentstack-vibe-docs/extensions/cli-plugins.md)
- DevHub apps: [DevHub Apps Guide](.contentstack-vibe-docs/extensions/devhub-apps.md)

---

## Key Features

### AI Agent Optimized

- **Modular files** - Each topic in focused document
- **Quick reference** - Condensed patterns for common tasks
- **Version info** - Package compatibility documented
- **Framework patterns** - Specific guides for Next.js, Nuxt, Gatsby
- **Copy-paste ready** - All code examples work as-is

### Comprehensive Coverage

- CMS concepts and terminology
- REST and GraphQL APIs
- TypeScript Delivery SDK
- Live Preview (CSR and SSR)
- Framework-specific patterns
- CLI plugin development
- Developer Hub apps

### Region Support

All regions supported: US, EU, AU, Azure NA/EU, GCP NA/EU

---

## Best Practices

### Security

- Never expose Management Tokens in frontend
- Use Delivery Tokens for public apps
- Use Preview Tokens only in preview environments
- Store credentials in environment variables

### Performance

- Implement caching for frequent content
- Use CDN for assets
- Optimize queries (fetch only needed fields)
- Implement pagination for large datasets

### Code Quality

- Use TypeScript for type safety
- Create reusable helper functions
- Handle errors gracefully
- Follow framework patterns

---

## Support

- [Official Documentation](https://www.contentstack.com/docs)
- [Discord Community](https://community.contentstack.com)
- [Contentstack Support](https://www.contentstack.com/support)

---

## License

This documentation is provided as-is for use with AI coding assistants.
