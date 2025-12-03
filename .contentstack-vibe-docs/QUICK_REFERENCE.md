# Contentstack Quick Reference for AI Agents

Condensed patterns for common Contentstack operations. Use this for quick lookups.

## Stack Initialization (Use Once)

### Basic (Manual Region)

```typescript
// lib/contentstack.ts - Import this everywhere, don't repeat
import contentstack from "@contentstack/delivery-sdk";

export const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION || "us", // us, eu, au, azure-na, azure-eu, gcp-na, gcp-eu
  live_preview: {
    enable: true,
    preview_token: process.env.CONTENTSTACK_PREVIEW_TOKEN,
    host: "rest-preview.contentstack.com", // Manual - must match region!
  },
});
```

### With Endpoints Package (Recommended)

```typescript
import contentstack from "@contentstack/delivery-sdk";
import { getContentstackEndpoints } from "@timbenniks/contentstack-endpoints";

// omitHttps: true removes https:// prefix (needed for SDK host parameter)
const endpoints = getContentstackEndpoints(
  process.env.CONTENTSTACK_REGION || "us",
  true // omitHttps = true
);

export const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION || "us",
  live_preview: {
    enable: true,
    preview_token: process.env.CONTENTSTACK_PREVIEW_TOKEN,
    host: endpoints.contentPreview, // Already without https://
  },
});
```

---

## Fetching Entries

### Single Entry by UID

```typescript
const entry = await stack.contentType("blog_post").entry("entry_uid").fetch();
```

### Single Entry with References

```typescript
const entry = await stack
  .contentType("blog_post")
  .entry("entry_uid")
  .includeReference(["author", "category"])
  .fetch();
```

### Entry by URL

```typescript
import { QueryOperation } from "@contentstack/delivery-sdk";

const result = await stack
  .contentType("page")
  .entry()
  .query()
  .where("url", QueryOperation.EQUALS, "/about")
  .find();

const entry = result.entries[0];
```

### Multiple Entries with Pagination

```typescript
const result = await stack
  .contentType("blog_post")
  .entry()
  .query()
  .skip(0)
  .limit(10)
  .includeCount()
  .descending("published_date")
  .find();

// result.entries = array of entries
// result.count = total count
```

### Entry in Specific Locale

```typescript
const entry = await stack
  .contentType("page")
  .entry("entry_uid")
  .language("fr-fr")
  .fetch();
```

---

## Query Operations

```typescript
import { QueryOperation } from "@contentstack/delivery-sdk";

// Equals
.where("status", QueryOperation.EQUALS, "published")

// Not equals
.where("status", QueryOperation.NOT_EQUAL, "draft")

// Greater/Less than
.where("price", QueryOperation.GREATER_THAN, 100)
.where("price", QueryOperation.LESS_THAN_OR_EQUAL, 500)

// In array
.where("category", QueryOperation.IN, ["tech", "design"])

// Contains
.where("title", QueryOperation.CONTAINS, "tutorial")

// Complex query
.addQuery({
  $or: [
    { category: { $in: ["tech", "design"] } },
    { tags: { $regex: "javascript", $options: "i" } }
  ]
})
```

---

## Sorting

```typescript
// Ascending
.ascending("published_date")

// Descending
.descending("published_date")

// Multiple
.ascending("category").descending("published_date")
```

---

## Field Selection

```typescript
// Only specific fields
.only(["title", "url", "published_date"])

// Exclude fields
.except(["internal_notes", "draft_content"])
```

---

## Live Preview

**Note**: Live Preview is for development/preview only. Never enabled in production.

### The `ssr` Setting

- `ssr: false` → Uses postMessage. Client re-fetches via `onEntryChange()`. No page refresh.
- `ssr: true` → CMS refreshes iframe with query params. Server reads params to fetch preview data.

### Initialize (Client-Side)

```typescript
import ContentstackLivePreview, {
  IStackSdk,
} from "@contentstack/live-preview-utils";

ContentstackLivePreview.init({
  ssr: false, // false: postMessage updates, true: iframe refresh with query params
  enable: true,
  stackSdk: stack.config as IStackSdk,
  stackDetails: {
    apiKey: process.env.CONTENTSTACK_API_KEY!,
    environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  },
  editButton: { enable: true },
});
```

### Handle Entry Changes (`ssr: false`)

```typescript
// CMS sends postMessage, SDK triggers this callback
ContentstackLivePreview.onEntryChange(async () => {
  const updated = await fetchData();
  setEntry(updated);
});
```

### Handle Query Params (`ssr: true`)

```typescript
// CMS refreshes iframe with: ?live_preview=hash&entry_uid=...&content_type_uid=...
// Read from URL and apply to stack
const { live_preview, entry_uid, content_type_uid } = searchParams;

if (live_preview) {
  stack.livePreviewQuery({
    live_preview,
    contentTypeUid: content_type_uid || "page",
    entryUid: entry_uid || "",
  });
}
```

### Add Editable Tags

```typescript
import contentstack from "@contentstack/delivery-sdk";

contentstack.Utils.addEditableTags(entry, "content_type_uid", true, "en-us");

// In JSX:
<h1 {...(entry?.$ && entry?.$.title)}>{entry.title}</h1>;
```

---

## Assets

### Fetch Asset

```typescript
const asset = await stack.asset("asset_uid").fetch();
console.log(asset.url, asset.title, asset.filename);
```

### Image Transformations

```typescript
// Width and height
`${asset.url}?width=800&height=600` // WebP format with quality
`${asset.url}?format=webp&quality=85` // Crop fit
`${asset.url}?width=400&height=300&fit=crop` // Auto optimization
`${asset.url}?auto=webp`;
```

---

## Region Configuration

### Using Endpoints Package (Recommended)

```typescript
import { getContentstackEndpoints } from "@timbenniks/contentstack-endpoints";

// omitHttps: true for SDKs (removes https:// prefix)
// omitHttps: false for direct API calls (includes https://)
const endpoints = getContentstackEndpoints("us", true); // or "eu", "au", "azure-na", etc.

// Available endpoints:
endpoints.contentDelivery; // REST CDN
endpoints.contentManagement; // Management API
endpoints.graphqlDelivery; // GraphQL
endpoints.contentPreview; // REST Preview
endpoints.graphqlPreview; // GraphQL Preview
```

### Region Codes

| Code       | Full Name           | Aliases        |
| ---------- | ------------------- | -------------- |
| `us`       | AWS North America   | `na`, `aws-na` |
| `eu`       | AWS Europe          | `aws-eu`       |
| `au`       | AWS Australia       | `aws-au`       |
| `azure-na` | Azure North America | `azure_na`     |
| `azure-eu` | Azure Europe        | `azure_eu`     |
| `gcp-na`   | GCP North America   | `gcp_na`       |
| `gcp-eu`   | GCP Europe          | `gcp_eu`       |

See [Regions Guide](../concepts/regions.md) for complete details.

---

## TypeScript Types

```typescript
import { Entry } from "@contentstack/delivery-sdk";

interface BlogPost extends Entry {
  title: string;
  url: string;
  content: string;
  published_date: string;
  author?: Author;
  featured_image?: Asset;
}

interface Asset {
  uid: string;
  url: string;
  title: string;
  filename: string;
  dimension?: { width: number; height: number };
}
```

---

## Error Handling Pattern

```typescript
try {
  const entry = await stack.contentType("page").entry(uid).fetch();
  return entry;
} catch (error) {
  console.error(
    "Error fetching entry:",
    error instanceof Error ? error.message : error
  );
  return null;
}
```

---

## Environment Variables

```bash
CONTENTSTACK_API_KEY=your_api_key
CONTENTSTACK_DELIVERY_TOKEN=your_delivery_token
CONTENTSTACK_PREVIEW_TOKEN=your_preview_token
CONTENTSTACK_ENVIRONMENT=production
CONTENTSTACK_REGION=us
```
