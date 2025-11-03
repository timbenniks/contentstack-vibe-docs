# Contentstack Vibe Coding Documentation for AI Agents

This repository contains comprehensive documentation for building applications with Contentstack CMS. The guides are specifically written for AI coding assistants to help them implement Contentstack features quickly and accurately.

## Agent Instructions

When helping developers implement Contentstack features, follow these guidelines:

### How to Use This Documentation

1. **Start with Context**: Always read the relevant documentation file before implementing. Don't assume - verify patterns match the latest SDK/API versions.
2. **Follow the Numbered Order**: Files are numbered (`00-`, `01-`, etc.) for a reason. Read prerequisites before advanced topics.
3. **Reference Practical Examples**: When writing code, check `05-practical-examples.md` for real-world patterns that match the developer's framework.
4. **Verify Against Multiple Sources**: Cross-reference patterns between SDK docs, API docs, and examples to ensure consistency.

### Questions to Ask Developers

Before implementing Contentstack features, gather this information:

#### Basic Setup

- **What is the Contentstack region?** (US, EU, AU, Azure, GCP) - Required for correct endpoints
- **Do you have API credentials?** (API Key, Delivery Token, Preview Token if needed)
- **What environment are you using?** (production, staging, preview, development)
- **Is this a new project or existing project?** - Determines setup approach

#### Framework & Architecture

- **What framework are you using?** (Next.js, Nuxt, Gatsby, React, Vue, etc.)
- **Are you using SSR or CSR?** - Critical for Live Preview implementation
- **Do you have a routing setup?** - Needed for Live Preview URL handling
- **What's your hosting setup?** (Vercel, Netlify, self-hosted) - Affects Live Preview configuration

#### Content Requirements

- **What content types do you need?** - Helps understand query patterns
- **Do you need references included?** - Determines `.includeReference()` usage
- **Are you using modular blocks?** - Affects rendering patterns
- **Do you need multiple locales?** - Requires locale handling
- **Do you need image transformations?** - Asset handling patterns

#### Live Preview (if applicable)

- **Do you need Live Preview?** - Yes/No determines setup complexity
- **Is your site publicly accessible via HTTPS?** - Required for Live Preview
- **Do you have a Preview Token?** - Required for Live Preview
- **Is Live Preview enabled in Contentstack?** - Must be configured in CMS

### Decision Trees

#### Choosing API vs SDK

- **Use SDK** (`@contentstack/delivery-sdk`) when:
  - Building with JavaScript/TypeScript
  - Need type safety
  - Want built-in error handling
  - Need Live Preview support
- **Use REST/GraphQL API directly** when:
  - Using non-JavaScript languages
  - Need fine-grained control
  - Building custom integrations

#### Choosing CSR vs SSR for Live Preview

- **Use CSR** (`ssr: false`) when:
  - Client-side rendered application (React SPA, Vue SPA)
  - Want instant updates without page refresh
  - Using client-side routing
- **Use SSR** (`ssr: true`) when:
  - Server-side rendered application (Next.js SSR, Nuxt SSR)
  - Need SEO benefits
  - Using server-side routing

### Implementation Workflow

1. **Gather Requirements** - Ask the questions above
2. **Read Relevant Docs** - Start with Base Concepts if new, then SDK/API docs
3. **Check Examples** - Find matching patterns in Practical Examples
4. **Implement Step-by-Step** - Follow documentation patterns exactly
5. **Add Error Handling** - Always include try-catch and proper error messages
6. **Use Environment Variables** - Never hardcode credentials
7. **Test Thoroughly** - Verify with actual Contentstack stack

### Common Scenarios

#### Scenario: "I need to fetch content from Contentstack"

**Ask**: What framework? Do you have credentials? What content type?
**Action**: Read `02-sdk-functionalities.md`, check `05-practical-examples.md` for framework-specific patterns

#### Scenario: "I want to add Live Preview"

**Ask**: CSR or SSR? Is site HTTPS? Do you have Preview Token?
**Action**: Read `03-live-preview-guide.md` completely, find framework pattern in Section 12

#### Scenario: "I'm getting errors"

**Ask**: What's the error message? What SDK version? What region?
**Action**: Check troubleshooting sections in relevant docs, verify credentials, check region endpoints

#### Scenario: "I need to query with filters"

**Ask**: What filters? Single or multiple conditions?
**Action**: Read `02-sdk-functionalities.md` Query Building section, check `QueryOperation` examples

### Best Practices for Agents

1. **Always Use TypeScript Types**: Import `Entry` and create proper interfaces
2. **Always Initialize Stack Properly**: Use environment variables, never hardcode
3. **Always Include Error Handling**: Wrap API calls in try-catch blocks
4. **Always Use `.includeReference()`**: When references are needed, explicitly include them
5. **Always Verify Credentials**: Remind developers to check API keys and tokens
6. **Always Check Region**: Verify correct region endpoints are used
7. **Always Follow Framework Patterns**: Match the developer's framework exactly
8. **Always Add Comments**: Explain Contentstack-specific code patterns

### Red Flags to Watch For

⚠️ **Never do these:**

- Hardcode API keys or tokens in code
- Use Management Tokens in frontend code
- Skip error handling
- Assume SDK versions (check package.json)
- Mix REST and GraphQL patterns incorrectly
- Forget to include references when needed
- Ignore region configuration
- Skip HTTPS requirement for Live Preview

### When to Ask for Help

Ask the developer for clarification when:

- Credentials are missing or unclear
- Framework/architecture is ambiguous
- Error messages are unclear
- Requirements conflict with documentation
- Custom setups deviate from standard patterns

## Quick Reference

All documentation files are located in `.contentstack-vibe-docs/`:

- `00-contentstack-base-concepts.md` - Start here for fundamentals
- `01-api-functionalities.md` - REST and GraphQL API usage
- `02-sdk-functionalities.md` - JavaScript/TypeScript SDK patterns
- `03-live-preview-guide.md` - Complete Live Preview implementation
- `05-practical-examples.md` - Real-world code examples

**File naming convention**: Files are numbered for recommended reading order. Missing numbers (e.g., `04-`) indicate removed or deprecated content.

## Documentation Structure

### Core Concepts

- **[Base Concepts](.contentstack-vibe-docs/00-contentstack-base-concepts.md)** - Fundamental CMS concepts, terminology, and architecture
- **[API Functionalities](.contentstack-vibe-docs/01-api-functionalities.md)** - REST API and GraphQL API usage, endpoints, and patterns
- **[SDK Functionalities](.contentstack-vibe-docs/02-sdk-functionalities.md)** - JavaScript/TypeScript SDK usage, querying, and best practices

### Feature Guides

- **[Live Preview Guide](.contentstack-vibe-docs/03-live-preview-guide.md)** - Complete Live Preview guide covering concepts, setup, CSR/SSR modes, and framework-specific patterns (Next.js, Nuxt 4, Gatsby)
- **[Practical Examples](.contentstack-vibe-docs/05-practical-examples.md)** - Real-world implementation examples for rendering content, references, modular blocks, asset transformations, and common patterns

## Key Features

### AI Agent Optimized

- **Step-by-step instructions** with clear implementation patterns
- **Critical requirements** highlighted throughout
- **Troubleshooting sections** with debug checklists
- **Copy-paste ready code** examples with proper error handling
- **TypeScript support** with type-safe examples

### Comprehensive Coverage

- **CMS Base Concepts** - Stack, Content Types, Entries, Environments, Locales
- **API Functionalities** - REST API, GraphQL API, authentication, querying
- **SDK Functionalities** - Delivery SDK usage, query building, error handling
- **Live Preview** - Setup, configuration, CSR/SSR modes, framework patterns

### Framework Support

The Live Preview implementation guide includes patterns for:

- **Next.js App Directory** (CSR and SSR modes)
- **Nuxt 4** (CSR and SSR modes)
- **Gatsby** (with Contentstack plugin)
- **Universal patterns** applicable to other frameworks (React, Vue, Express, etc.)

### Integration Options

- **[@contentstack/delivery-sdk](https://www.npmjs.com/package/@contentstack/delivery-sdk)** - Official TypeScript Delivery SDK
- **[@contentstack/live-preview-utils](https://www.npmjs.com/package/@contentstack/live-preview-utils)** - Live Preview utilities
- **[@timbenniks/contentstack-endpoints](https://www.npmjs.com/package/@timbenniks/contentstack-endpoints)** - Recommended package for endpoint management
- **Manual endpoint configuration** - Alternative approach for custom setups

### Region Support

All Contentstack regions are supported:

- **US** (North America - AWS)
- **EU** (Europe - AWS)
- **AU** (Australia - AWS)
- **Azure NA/EU** (Azure North America/Europe)
- **GCP NA/EU** (GCP North America/Europe)

## Quick Start

### For New Projects

**Recommended reading order:**

1. **Understand the CMS**: Start with [Base Concepts](.contentstack-vibe-docs/00-contentstack-base-concepts.md) - Learn Stack, Content Types, Entries, Environments, Locales
2. **Learn the APIs**: Review [API Functionalities](.contentstack-vibe-docs/01-api-functionalities.md) - REST and GraphQL endpoints, authentication, querying
3. **Set up SDK**: Follow [SDK Functionalities](.contentstack-vibe-docs/02-sdk-functionalities.md) - JavaScript/TypeScript SDK usage, query building, error handling
4. **Add Live Preview**: Follow [Live Preview Guide](.contentstack-vibe-docs/03-live-preview-guide.md) - Complete setup and implementation for all frameworks
5. **See Examples**: Reference [Practical Examples](.contentstack-vibe-docs/05-practical-examples.md) - Real-world code patterns for rendering content

### For Existing Projects

1. **Review Live Preview**: Read the [Live Preview Guide](.contentstack-vibe-docs/03-live-preview-guide.md)
2. **Choose rendering mode**: Decide between CSR (real-time updates) or SSR (page refresh)
3. **Follow implementation patterns**: See framework-specific examples in the guide
4. **Test integration**: Use the provided troubleshooting checklists to verify setup

## Documentation by Use Case

### I want to understand Contentstack basics

→ Start with [Base Concepts](.contentstack-vibe-docs/00-contentstack-base-concepts.md)

### I want to use Contentstack APIs directly

→ Read [API Functionalities](.contentstack-vibe-docs/01-api-functionalities.md)

### I want to use the Contentstack SDK

→ Follow [SDK Functionalities](.contentstack-vibe-docs/02-sdk-functionalities.md)

### I want to implement Live Preview

→ Follow the [Live Preview Guide](.contentstack-vibe-docs/03-live-preview-guide.md)

### I need practical examples

→ See [Practical Examples](.contentstack-vibe-docs/05-practical-examples.md) for rendering patterns, references, modular blocks, and common use cases

### I'm building with Next.js

→ See framework patterns in [Live Preview Guide - Section 12](.contentstack-vibe-docs/03-live-preview-guide.md#12-framework-specific-patterns) for Next.js App Directory patterns (CSR and SSR)

### I'm building with Nuxt 4

→ See framework patterns in [Live Preview Guide - Section 12](.contentstack-vibe-docs/03-live-preview-guide.md#12-framework-specific-patterns) for Nuxt 4 patterns (CSR and SSR)

## Best Practices

### Security

- **Never expose Management Tokens** in frontend code
- **Use Delivery Tokens** for public-facing applications
- **Use Preview Tokens** only in preview/staging environments
- **Store credentials** in environment variables, never hardcode

### Performance

- **Implement caching** for frequently accessed content
- **Use CDN** for asset delivery
- **Optimize queries** to fetch only needed fields
- **Implement pagination** for large datasets

### Code Quality

- **Use TypeScript** for type safety
- **Create reusable helper functions** for common operations
- **Handle errors gracefully** with proper error handling
- **Follow framework patterns** for consistency

## Contributing

This documentation is maintained to help AI coding assistants implement Contentstack features effectively. Each guide focuses on practical implementation with minimal setup overhead.

## Support

For Contentstack-specific issues:

- Refer to the [official documentation](https://www.contentstack.com/docs)
- Join the [Discord community](https://community.contentstack.com)
- Check [Contentstack Support](https://www.contentstack.com/support)

## License

This documentation is provided as-is for use with AI coding assistants.
