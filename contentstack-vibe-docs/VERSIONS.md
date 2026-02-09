# Package Versions & Compatibility

This documentation is tested and verified with the following package versions.

## Core Packages

| Package                            | Recommended Version | Notes                   |
| ---------------------------------- | ------------------- | ----------------------- |
| `@contentstack/delivery-sdk`       | `^4.10.3`           | TypeScript Delivery SDK |
| `@contentstack/live-preview-utils` | `^4.1.2`            | Live Preview SDK v4     |
| `@contentstack/utils`              | `^1.6.2`            | Utility functions       |
| `@contentstack/app-sdk`            | `^2.3.4`            | Developer Hub App SDK   |

## CLI & Plugin Development

| Package                       | Recommended Version | Notes                  |
| ----------------------------- | ------------------- | ---------------------- |
| `@contentstack/cli`           | `^1.52.0`           | Contentstack CLI       |
| `@contentstack/cli-command`   | `^1.6.1`            | CLI command base class |
| `@contentstack/cli-utilities` | `^1.14.4`           | CLI utility functions  |
| `@contentstack/management`    | `^1.25.1`           | Management SDK         |
| `@oclif/core`                 | `^4.8.0`            | oclif framework        |

## Framework Integrations

| Package                      | Recommended Version | Notes                |
| ---------------------------- | ------------------- | -------------------- |
| `gatsby-source-contentstack` | `^5.4.0`            | Gatsby source plugin |

## Utilities

| Package                              | Recommended Version | Notes                             |
| ------------------------------------ | ------------------- | --------------------------------- |
| `@timbenniks/contentstack-endpoints` | `^2.1.0`            | Region endpoint resolution helper |

## Runtime Requirements

| Requirement | Version    |
| ----------- | ---------- |
| Node.js     | `>=20.0.0` |
| npm         | `>=10.0.0` |

## Installation Commands

### For Content Delivery (Websites/Apps)

```bash
npm install @contentstack/delivery-sdk @contentstack/live-preview-utils
```

### For CLI Plugin Development

```bash
npm install @contentstack/cli-command @contentstack/cli-utilities @contentstack/management @oclif/core
```

### For Developer Hub Apps

```bash
npm install @contentstack/app-sdk
```

## Version Compatibility Notes

### Live Preview Utils v4

Version 4.x features:

- `mode` property (`"preview"` or `"builder"`)
- Improved Edit button configuration
- `cleanCslpOnProduction` option
- Better TypeScript support
- Enhanced postMessage handling

### Delivery SDK v4

Version 4.x features:

- Full TypeScript support
- Improved query builder
- Better error handling
- `QueryOperation` enum for type-safe queries
- Enhanced Live Preview integration

## Checking Installed Versions

```bash
# Check all Contentstack packages
npm list | grep contentstack

# Check specific package
npm list @contentstack/delivery-sdk
```

## Updating Packages

```bash
# Update to latest compatible versions
npm update @contentstack/delivery-sdk @contentstack/live-preview-utils

# Check for outdated packages
npm outdated
```
