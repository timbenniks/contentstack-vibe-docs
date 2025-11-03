# Contentstack CMS: Base Concepts for AI Agents

This document provides fundamental concepts about Contentstack CMS that AI coding assistants need to understand when building applications with Contentstack.

## What is Contentstack?

Contentstack is a headless Content Management System (CMS) that separates content management from content presentation. It provides a RESTful API and GraphQL API to deliver content to any platform or device.

## Core Concepts

### Stack

A **Stack** is the top-level container in Contentstack. It represents a project or application and contains all your content types, entries, assets, environments, and configurations.

- **API Key**: Unique identifier for your stack (required for all API calls)
- **Region**: Geographic location of your stack's data center (US, EU, AU, Azure, GCP)
- **Access Token**: Authentication token for API access (Delivery Token or Management Token)

### Content Types

A **Content Type** defines the structure of your content. It's like a database schema that defines:

- **UID**: Unique identifier for the content type (e.g., `blog_post`, `page`)
- **Fields**: The individual data fields (text, rich text, number, date, file, reference, etc.)
- **Schema**: JSON schema defining field types, validation rules, and relationships

**Common Field Types:**

- `text`: Single-line text input
- `textarea`: Multi-line text input
- `richtext`: Rich text editor with formatting
- `number`: Numeric values
- `date`: Date and time values
- `file`: File uploads (images, documents, etc.)
- `reference`: Reference to other entries
- `modular_block`: Reusable content blocks
- `group`: Group of related fields
- `json`: Raw JSON data

### Entries

An **Entry** is an instance of a content type. It contains the actual content data.

- **UID**: Unique identifier for the entry
- **Title**: Display name for the entry
- **Locale**: Language/variant of the content (e.g., `en-us`, `fr-fr`)
- **Created/Updated**: Timestamps for creation and modification
- **Published**: Status indicating if entry is published

### Environments

**Environments** represent different stages of your content lifecycle:

- **Development**: For testing and development
- **Staging**: For pre-production testing
- **Production**: Live environment for end users
- **Custom**: You can create custom environments

Each environment can have different content versions and configurations.

### Locales

**Locales** represent different languages or regional variations:

- Each entry can exist in multiple locales
- Locale codes follow ISO format (e.g., `en-us`, `fr-fr`, `de-de`)
- Default locale is set at stack level
- Entries can be translated or localized per locale

### Assets

**Assets** are files stored in Contentstack:

- Images, videos, documents, etc.
- Managed through the Asset Library
- Assets have URLs that can be referenced in entries
- Support for image transformations and CDN delivery
- Automatically served through Contentstack's global CDN
- Support for versioning and asset management

**Asset Features:**

- **CDN Delivery**: All assets are automatically cached and delivered via CDN
- **Image Transformations**: On-the-fly image resizing, cropping, and format conversion
- **Asset Library**: Centralized management of all files
- **Versioning**: Track asset versions and rollback if needed
- **Metadata**: Store title, description, alt text, and custom metadata

### References

**References** allow entries to link to other entries:

- Single reference: Links to one entry
- Multiple references: Links to multiple entries
- Cross-content-type references supported
- Enables content relationships and reuse

### Modular Blocks

**Modular Blocks** are reusable content components:

- Defined within a content type or as standalone blocks
- Can be added/removed dynamically
- Enable flexible page building without code changes
- Each block has its own schema and fields

### Delivery vs Management API

**Delivery API** (CDA - Content Delivery API):

- Read-only access to published content
- Uses Delivery Tokens
- Fast, cached, CDN-delivered
- Used in production applications

**Management API** (CMA - Content Management API):

- Full CRUD operations on content
- Uses Management Tokens
- Requires authentication and permissions
- Used for content administration

**Preview API**:

- Access to unpublished content
- Uses Preview Tokens
- For Live Preview and development
- Requires special preview endpoints

## Authentication & Tokens

### Delivery Token

- **Purpose**: Access published content via Delivery API
- **Scopes**: Can be restricted to specific content types, environments, or locales
- **Security**: Should be public-facing (used in frontend applications)
- **Usage**: Required for all delivery API calls

### Preview Token

- **Purpose**: Access unpublished content for preview purposes
- **Associated**: Generated from a Delivery Token
- **Usage**: Required for Live Preview functionality
- **Security**: Should be used only in preview/staging environments

### Management Token

- **Purpose**: Full CRUD operations via Management API
- **Scopes**: Based on user roles and permissions
- **Security**: Should NEVER be exposed in frontend code
- **Usage**: Backend/server-side operations only
- **Access**: Full content management capabilities (create, update, delete entries, assets, content types)

### Management API (CMA) Usage

The Management API is used for:

- **Content Import/Export**: Bulk content operations
- **CLI Operations**: Contentstack CLI uses CMA for automation
- **Content Migration**: Moving content between stacks or environments
- **Automation**: Scripting content management tasks
- **Backend Services**: Server-side content administration

**Note**: Most application development uses the Delivery API. Management API is primarily for content administration and automation tasks.

## Architecture Overview

### Big Picture Architecture

Contentstack follows a headless architecture pattern:

1. **Content Management Layer**: Content creators use the Contentstack UI to create and manage content
2. **Content Delivery Layer**: Content is delivered via REST or GraphQL APIs through a global CDN
3. **Presentation Layer**: Your application (website, mobile app, etc.) consumes content via APIs

**Key Benefits:**

- **Separation of Concerns**: Content management is separate from presentation
- **Multi-Channel**: Same content can power websites, mobile apps, IoT devices
- **Developer Freedom**: Use any programming language or framework
- **Performance**: Global CDN ensures fast content delivery worldwide
- **Scalability**: CDN handles traffic spikes automatically

### Environments Architecture

Environments provide content isolation for different stages:

**Environment Workflow:**

```
Development → Staging → Production
```

**Best Practices:**

- **Development**: Local development and testing
- **Staging**: Pre-production testing with production-like data
- **Production**: Live environment serving end users
- **Custom Environments**: Additional environments for specific needs (e.g., `preview`, `qa`)

**Environment Configuration:**

- Each environment can have different content versions
- Environment-specific delivery tokens
- Environment-specific configurations
- Content must be published to each environment separately

**Using Environments in Code:**

```typescript
// Fetch from specific environment
const entry = await stack
  .contentType("page")
  .entry("entry_uid")
  .environment("production") // or "staging", "development"
  .fetch();
```

## Regions and Endpoints

Contentstack operates in multiple regions for global performance:

| Region              | Code       | REST API Endpoint              | GraphQL Endpoint                    |
| ------------------- | ---------- | ------------------------------ | ----------------------------------- |
| North America (AWS) | `us`       | `cdn.contentstack.io`          | `graphql.contentstack.com`          |
| Europe (AWS)        | `eu`       | `eu-cdn.contentstack.com`      | `eu-graphql.contentstack.com`       |
| Australia (AWS)     | `au`       | `au-cdn.contentstack.io`       | `au-graphql.contentstack.com`       |
| Azure North America | `azure-na` | `azure-na-cdn.contentstack.io` | `azure-na-graphql.contentstack.com` |
| Azure Europe        | `azure-eu` | `azure-eu-cdn.contentstack.io` | `azure-eu-graphql.contentstack.com` |
| GCP North America   | `gcp-na`   | `gcp-na-cdn.contentstack.io`   | `gcp-na-graphql.contentstack.com`   |
| GCP Europe          | `gcp-eu`   | `gcp-eu-cdn.contentstack.io`   | `gcp-eu-graphql.contentstack.com`   |

**Preview Endpoints** (when using Preview Token):

- REST: `rest-preview.contentstack.com` (or region-specific)
- GraphQL: `graphql-preview.contentstack.com` (or region-specific)

**Management API Endpoints** (for CMA operations):

- REST: `api.contentstack.io` (or region-specific)
- Regions: `eu-api.contentstack.com`, `au-api.contentstack.com`, etc.

**Choose the correct region** based on:

- Your stack's configured region
- Geographic location of your users
- Compliance requirements (e.g., EU data residency)

## Content Delivery Patterns

### REST API

- **Endpoint**: `https://{region}-cdn.contentstack.io/v3/`
- **Authentication**: Via headers (`api_key`, `access_token`)
- **Query Parameters**: For filtering, sorting, pagination
- **Response Format**: JSON
- **SDK**: `@contentstack/delivery-sdk`

### GraphQL API

- **Endpoint**: `https://{region}-graphql.contentstack.com/graphql`
- **Authentication**: Via headers (`api_key`, `access_token`)
- **Query Language**: GraphQL queries
- **Response Format**: JSON
- **SDK**: `@contentstack/delivery-sdk` (with GraphQL support)

### SDK Usage

Contentstack provides official SDKs for various platforms:

- **JavaScript/TypeScript**: `@contentstack/delivery-sdk`
- **Live Preview Utils**: `@contentstack/live-preview-utils`
- **Utilities**: `@contentstack/utils`
- **Gatsby**: `gatsby-source-contentstack`
- **Node.js**: Same as JavaScript SDK

## CDN and Caching

### Content Delivery Network (CDN)

Contentstack uses a global CDN to deliver content:

- **Fast Delivery**: Content cached at edge locations worldwide
- **Automatic Caching**: API responses are automatically cached
- **Cache Invalidation**: Content updates automatically invalidate cache
- **No Configuration**: CDN is enabled by default for all delivery API calls

### Caching Behavior

**API Responses:**

- Entry data is cached at CDN edge locations
- Cache is automatically invalidated when content is published
- Unpublished content (preview) is not cached

**Assets:**

- All assets are served through CDN
- Image transformations are cached
- Long cache times for optimal performance

**Best Practices:**

- Leverage CDN caching for better performance
- Use appropriate cache headers in your application
- Implement client-side caching for frequently accessed content
- Use preview endpoints only in development (not cached)

## Content Modeling Best Practices

1. **Use Descriptive UIDs**: Content type and field UIDs should be clear and consistent
2. **Leverage References**: Use references to avoid content duplication
3. **Modular Blocks**: Use modular blocks for flexible page building
4. **Group Related Fields**: Use groups to organize related fields
5. **Validate Content**: Set up validation rules at content type level
6. **Plan for Localization**: Design content types with multiple locales in mind
7. **Design for Performance**: Use references efficiently, avoid deep nesting
8. **Structure for Reusability**: Create reusable content types (e.g., reusable author profile)

## Common Workflows

### Content Creation Flow

1. Content manager creates/edits entry in Contentstack UI
2. Entry saved as draft
3. Entry can be previewed (using Live Preview)
4. Entry published to an environment
5. Published content available via Delivery API
6. Frontend application fetches and displays content

### Development Flow

1. Developer sets up SDK with API key and delivery token
2. Developer queries content via SDK
3. Content fetched from specified environment
4. Developer renders content in application
5. Live Preview enabled for real-time content updates during development

## Key Terms Summary

| Term                 | Description                                      |
| -------------------- | ------------------------------------------------ |
| **Stack**            | Top-level container for all content              |
| **Content Type**     | Schema definition for content structure          |
| **Entry**            | Instance of a content type with actual data      |
| **Environment**      | Stage of content lifecycle (dev, staging, prod)  |
| **Locale**           | Language or regional variation                   |
| **Asset**            | File stored in Contentstack (image, video, etc.) |
| **Reference**        | Link between entries                             |
| **Modular Block**    | Reusable content component                       |
| **Delivery Token**   | Token for accessing published content            |
| **Preview Token**    | Token for accessing unpublished content          |
| **Management Token** | Token for content administration                 |

## Next Steps

- Learn about [API Functionalities](01-api-functionalities.md)
- Learn about [SDK Functionalities](02-sdk-functionalities.md)
- Learn about [Live Preview Setup](03-live-preview-base-concepts.md)
- Learn about [Personalization Setup](04-personalization-setup.md)
