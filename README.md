# Contentstack Vibe Coding Documentation for AI Agents

This repository contains comprehensive documentation for building applications with Contentstack CMS. The guides are specifically written for AI coding assistants to help them implement Contentstack features quickly and accurately.

## Documentation Structure

### Core Concepts

- **[Base Concepts](.contentstack-vibe-docs/00-contentstack-base-concepts.md)** - Fundamental CMS concepts, terminology, and architecture
- **[API Functionalities](.contentstack-vibe-docs/01-api-functionalities.md)** - REST API and GraphQL API usage, endpoints, and patterns
- **[SDK Functionalities](.contentstack-vibe-docs/02-sdk-functionalities.md)** - JavaScript/TypeScript SDK usage, querying, and best practices

### Feature Guides

- **[Live Preview Base Concepts](.contentstack-vibe-docs/03-live-preview-base-concepts.md)** - Live Preview fundamentals, terminology, and setup process
- **[Live Preview Implementation](.contentstack-vibe-docs/03a-live-preview-implementation.md)** - Comprehensive Live Preview implementation guide with framework patterns
- **[Personalization Setup](.contentstack-vibe-docs/04-personalization-setup.md)** - Personalization features, personas, variants, and implementation patterns
- **[Practical Examples](.contentstack-vibe-docs/05-practical-examples.md)** - Real-world implementation examples for rendering content, references, modular blocks, and more

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
- **Personalization** - Personas, variants, A/B testing, user context

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

1. **Understand the CMS**: Start with [Base Concepts](.contentstack-vibe-docs/00-contentstack-base-concepts.md)
2. **Learn the APIs**: Review [API Functionalities](.contentstack-vibe-docs/01-api-functionalities.md)
3. **Set up SDK**: Follow [SDK Functionalities](.contentstack-vibe-docs/02-sdk-functionalities.md)
4. **Add Live Preview**: Follow [Live Preview Implementation](.contentstack-vibe-docs/03a-live-preview-implementation.md)
5. **Add Personalization**: Follow [Personalization Setup](.contentstack-vibe-docs/04-personalization-setup.md)

### For Existing Projects

1. **Review Live Preview**: Start with [Base Concepts](.contentstack-vibe-docs/03-live-preview-base-concepts.md)
2. **Choose rendering mode**: Decide between CSR (real-time updates) or SSR (page refresh)
3. **Follow implementation guide**: See framework patterns in [Live Preview Implementation](.contentstack-vibe-docs/03a-live-preview-implementation.md)
4. **Test integration**: Use the provided troubleshooting checklists to verify setup

## Documentation by Use Case

### I want to understand Contentstack basics

→ Start with [Base Concepts](.contentstack-vibe-docs/00-contentstack-base-concepts.md)

### I want to use Contentstack APIs directly

→ Read [API Functionalities](.contentstack-vibe-docs/01-api-functionalities.md)

### I want to use the Contentstack SDK

→ Follow [SDK Functionalities](.contentstack-vibe-docs/02-sdk-functionalities.md)

### I want to implement Live Preview

→ Read [Live Preview Base Concepts](.contentstack-vibe-docs/03-live-preview-base-concepts.md) then [Implementation Guide](.contentstack-vibe-docs/03a-live-preview-implementation.md)

### I want to implement Personalization

→ Follow [Personalization Setup](.contentstack-vibe-docs/04-personalization-setup.md)

### I need practical examples

→ See [Practical Examples](.contentstack-vibe-docs/05-practical-examples.md) for rendering patterns, references, modular blocks, and common use cases

### I'm building with Next.js

→ See framework patterns in [Live Preview Implementation](.contentstack-vibe-docs/03a-live-preview-implementation.md#12-framework-specific-patterns)

### I'm building with Nuxt 4

→ See framework patterns in [Live Preview Implementation](.contentstack-vibe-docs/03a-live-preview-implementation.md#12-framework-specific-patterns)

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
