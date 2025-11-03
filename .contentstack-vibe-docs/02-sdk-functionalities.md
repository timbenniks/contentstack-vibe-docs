# Contentstack SDK Functionalities for AI Agents

This guide covers Contentstack's JavaScript/TypeScript SDK (`@contentstack/delivery-sdk`) functionality, including initialization, querying, filtering, and best practices for AI coding assistants.

## Overview

The Contentstack Delivery SDK provides a developer-friendly interface to interact with Contentstack's REST API. It handles authentication, request building, error handling, and provides TypeScript support.

**Package**: `@contentstack/delivery-sdk`  
**Installation**: `npm install @contentstack/delivery-sdk`

## SDK Initialization

### Basic Stack Configuration

```typescript
import contentstack from "@contentstack/delivery-sdk";

const stack = contentstack.stack({
  apiKey: "YOUR_API_KEY",
  deliveryToken: "YOUR_DELIVERY_TOKEN",
  environment: "production",
  region: "us", // or "eu", "au", "azure-na", "azure-eu", "gcp-na", "gcp-eu"
});
```

### With Live Preview Support

```typescript
import contentstack from "@contentstack/delivery-sdk";

const stack = contentstack.stack({
  apiKey: "YOUR_API_KEY",
  deliveryToken: "YOUR_DELIVERY_TOKEN",
  environment: "preview",
  region: "us",
  live_preview: {
    enable: true,
    preview_token: "YOUR_PREVIEW_TOKEN",
    host: "rest-preview.contentstack.com", // or region-specific preview host
  },
});
```

### Using Region Endpoints Package (Recommended)

```typescript
import contentstack from "@contentstack/delivery-sdk";
import {
  getContentstackEndpoints,
  getRegionForString,
} from "@timbenniks/contentstack-endpoints";

const region = getRegionForString(process.env.CONTENTSTACK_REGION || "us");
const endpoints = getContentstackEndpoints(region, true); // true = omit https://

const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: region,
  live_preview: {
    enable: process.env.CONTENTSTACK_PREVIEW === "true",
    preview_token: process.env.CONTENTSTACK_PREVIEW_TOKEN,
    host: endpoints.preview,
  },
});
```

## Fetching Entries

### Single Entry by UID

```typescript
const entry = await stack.contentType("blog_post").entry("entry_uid").fetch();

console.log(entry.title);
console.log(entry.uid);
```

### Single Entry with Locale

```typescript
const entry = await stack
  .contentType("blog_post")
  .entry("entry_uid")
  .language("fr-fr")
  .fetch();
```

### Single Entry with References

```typescript
const entry = await stack
  .contentType("blog_post")
  .entry("entry_uid")
  .includeReference(["author", "category"])
  .fetch();

// Access referenced entries
console.log(entry.author.title);
console.log(entry.category.title);
```

### Multiple Entries

```typescript
const result = await stack.contentType("blog_post").entry().query().find();

console.log(result.entries); // Array of entries
console.log(result.count); // Total count (if includeCount() was called)
```

## Query Building

### Basic Query

```typescript
const result = await stack.contentType("blog_post").entry().query().find();
```

### Filtering with Where Clauses

```typescript
import { QueryOperation } from "@contentstack/delivery-sdk";

const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .where("title", QueryOperation.EQUALS, "My Post")
  .find();
```

### Multiple Where Clauses (AND)

```typescript
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .where("category", QueryOperation.EQUALS, "tech")
  .where("published_date", QueryOperation.LESS_THAN_OR_EQUAL, "2024-01-01")
  .find();
```

### Complex Queries

```typescript
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .addQuery({
    $or: [
      { category: { $in: ["tech", "design"] } },
      { tags: { $regex: "javascript", $options: "i" } },
    ],
  })
  .find();
```

### Query Operations

The SDK provides `QueryOperation` enum for common operations:

```typescript
import { QueryOperation } from "@contentstack/delivery-sdk";

// Available operations:
QueryOperation.EQUALS; // ==
QueryOperation.NOT_EQUAL; // !=
QueryOperation.GREATER_THAN; // >
QueryOperation.GREATER_THAN_OR_EQUAL; // >=
QueryOperation.LESS_THAN; // <
QueryOperation.LESS_THAN_OR_EQUAL; // <=
QueryOperation.IN; // in array
QueryOperation.NOT_IN; // not in array
QueryOperation.EXISTS; // field exists
QueryOperation.CONTAINS; // contains string
QueryOperation.NOT_CONTAINS; // does not contain
```

### Sorting

```typescript
// Ascending
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .ascending("published_date")
  .find();

// Descending
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .descending("published_date")
  .find();

// Multiple sorts
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .ascending("category")
  .descending("published_date")
  .find();
```

### Pagination

```typescript
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .skip(0) // Skip first N entries
  .limit(10) // Return maximum N entries
  .find();
```

### Include Count

```typescript
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .includeCount()
  .find();

console.log(result.entries.length); // Current page entries
console.log(result.count); // Total matching entries
```

### Locale Filtering

```typescript
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .language("fr-fr")
  .find();
```

### Include Specific Fields Only

```typescript
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .only(["title", "url", "published_date"])
  .find();
```

### Exclude Specific Fields

```typescript
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .except(["internal_notes", "draft_content"])
  .find();
```

## Working with Assets

### Fetch Single Asset

```typescript
const asset = await stack.asset("asset_uid").fetch();

console.log(asset.url);
console.log(asset.title);
console.log(asset.filename);
```

### Query Assets

```typescript
const result = await stack
  .asset()
  .query()
  .where("content_type", QueryOperation.EQUALS, "image/jpeg")
  .find();

console.log(result.assets);
```

## Live Preview Integration

### Apply Live Preview Query

```typescript
// For SSR applications
if (livePreviewHash) {
  stack.livePreviewQuery({
    live_preview: livePreviewHash,
    contentTypeUid: "blog_post",
    entryUid: "entry_uid",
  });
}

const entry = await stack.contentType("blog_post").entry("entry_uid").fetch();
```

### Add Editable Tags

```typescript
import contentstack from "@contentstack/delivery-sdk";

const entry = await stack.contentType("blog_post").entry("entry_uid").fetch();

// Add editable tags for Live Preview
contentstack.Utils.addEditableTags(entry, "blog_post", true, "en-us");

// In JSX/React, use the $ property:
// <h1 {...(entry?.$ && entry?.$.title)}>{entry.title}</h1>
```

**Parameters:**

- `entry`: The entry object
- `contentTypeUid`: Content type UID
- `tagsAsObject`: `true` for JSX, `false` for HTML strings
- `locale`: Locale code (optional)

## Error Handling

### Try-Catch Pattern

```typescript
try {
  const entry = await stack.contentType("blog_post").entry("entry_uid").fetch();
} catch (error) {
  if (error instanceof Error) {
    console.error("Error fetching entry:", error.message);
  }
  // Handle error appropriately
}
```

### Error Types

```typescript
import { ContentstackError } from "@contentstack/delivery-sdk";

try {
  const entry = await stack
    .contentType("blog_post")
    .entry("invalid_uid")
    .fetch();
} catch (error) {
  if (error instanceof ContentstackError) {
    console.error("Contentstack Error:", error.errorCode, error.errorMessage);
    console.error("Errors:", error.errors);
  }
}
```

## Advanced Features

### Custom Headers

```typescript
const stack = contentstack.stack({
  apiKey: "YOUR_API_KEY",
  deliveryToken: "YOUR_DELIVERY_TOKEN",
  environment: "production",
  region: "us",
  headers: {
    "X-Custom-Header": "custom-value",
  },
});
```

### Retry Configuration

```typescript
const stack = contentstack.stack({
  apiKey: "YOUR_API_KEY",
  deliveryToken: "YOUR_DELIVERY_TOKEN",
  environment: "production",
  region: "us",
  retryOptions: {
    retryDelayOptions: {
      base: 1000, // Base delay in ms
    },
    maxRetries: 3,
  },
});
```

### Cache Configuration

```typescript
const stack = contentstack.stack({
  apiKey: "YOUR_API_KEY",
  deliveryToken: "YOUR_DELIVERY_TOKEN",
  environment: "production",
  region: "us",
  cachePolicy: {
    cacheProvider: "memory", // or "redis"
    ttl: 3600, // Time to live in seconds
  },
});
```

## TypeScript Support

The SDK provides full TypeScript support:

```typescript
import contentstack, { Entry } from "@contentstack/delivery-sdk";

interface BlogPost extends Entry {
  title: string;
  url: string;
  content: string;
  author: Entry;
  published_date: string;
}

const result = await stack.contentType("blog_post").entry().query().find();

const entries = result.entries as BlogPost[];
```

## Best Practices

### 1. Always Use Environment Variables

```typescript
const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION!,
});
```

### 2. Create Reusable Helper Functions

```typescript
// lib/contentstack.ts
export async function getBlogPost(uid: string) {
  return await stack
    .contentType("blog_post")
    .entry(uid)
    .includeReference(["author", "category"])
    .fetch();
}

export async function getBlogPosts(filters?: {
  category?: string;
  limit?: number;
  skip?: number;
}) {
  const query = stack.contentType("blog_post").entry().query();

  if (filters?.category) {
    query.where("category", QueryOperation.EQUALS, filters.category);
  }

  if (filters?.limit) {
    query.limit(filters.limit);
  }

  if (filters?.skip) {
    query.skip(filters.skip);
  }

  return await query.find();
}
```

### 3. Handle Errors Gracefully

```typescript
async function safeFetchEntry(uid: string) {
  try {
    const entry = await stack.contentType("blog_post").entry(uid).fetch();
    return { success: true, data: entry };
  } catch (error) {
    return { success: false, error: error.message };
  }
}
```

### 4. Use Query Builders for Complex Queries

```typescript
function buildBlogPostQuery(filters: BlogPostFilters) {
  const query = stack.contentType("blog_post").entry().query();

  if (filters.category) {
    query.where("category", QueryOperation.EQUALS, filters.category);
  }

  if (filters.dateRange) {
    query.where(
      "published_date",
      QueryOperation.GREATER_THAN_OR_EQUAL,
      filters.dateRange.start
    );
    query.where(
      "published_date",
      QueryOperation.LESS_THAN_OR_EQUAL,
      filters.dateRange.end
    );
  }

  if (filters.search) {
    query.addQuery({
      $or: [
        { title: { $regex: filters.search, $options: "i" } },
        { content: { $regex: filters.search, $options: "i" } },
      ],
    });
  }

  return query;
}
```

### 5. Implement Pagination Properly

```typescript
async function getPaginatedBlogPosts(page: number = 1, pageSize: number = 10) {
  const skip = (page - 1) * pageSize;

  const result = await stack
    .contentType("blog_post")
    .entry()
    .query()
    .skip(skip)
    .limit(pageSize)
    .includeCount()
    .descending("published_date")
    .find();

  return {
    entries: result.entries,
    pagination: {
      currentPage: page,
      pageSize: pageSize,
      totalEntries: result.count,
      totalPages: Math.ceil(result.count / pageSize),
    },
  };
}
```

### 6. Cache Frequently Accessed Content

```typescript
import NodeCache from "node-cache";

const cache = new NodeCache({ stdTTL: 3600 });

async function getCachedBlogPost(uid: string) {
  const cacheKey = `blog_post_${uid}`;
  const cached = cache.get(cacheKey);

  if (cached) {
    return cached;
  }

  const entry = await stack.contentType("blog_post").entry(uid).fetch();
  cache.set(cacheKey, entry);
  return entry;
}
```

## Common Patterns

### Fetch Entry by URL

```typescript
async function getPageByUrl(url: string) {
  const result = await stack
    .contentType("page")
    .entry()
    .query()
    .where("url", QueryOperation.EQUALS, url)
    .find();

  return result.entries[0] || null;
}
```

### Fetch Related Entries

```typescript
async function getRelatedPosts(currentPostUid: string, category: string) {
  const result = await stack
    .contentType("blog_post")
    .entry()
    .query()
    .where("category", QueryOperation.EQUALS, category)
    .addQuery({ uid: { $ne: currentPostUid } })
    .limit(3)
    .find();

  return result.entries;
}
```

### Fetch Recent Entries

```typescript
async function getRecentPosts(limit: number = 5) {
  const result = await stack
    .contentType("blog_post")
    .entry()
    .query()
    .descending("published_date")
    .limit(limit)
    .find();

  return result.entries;
}
```

## Next Steps

- Learn about [Live Preview Setup](03-live-preview-base-concepts.md)
- Learn about [Live Preview Implementation](03a-live-preview-implementation.md)
- Learn about [Personalization Setup](04-personalization-setup.md)
