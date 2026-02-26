# Live Preview: `ssr: true` Mode

Iframe refresh with query parameters for server-side data fetching.

## What This Mode Does

When `ssr: true`:
1. Editor makes changes in the CMS
2. CMS **refreshes the iframe** with query parameters appended to the URL:
   - `?live_preview=<hash>&entry_uid=<uid>&content_type_uid=<type>`
3. Your server reads these query params
4. Server uses the `live_preview` hash to fetch the preview (unpublished) content
5. Server renders the page with preview data

## When to Use `ssr: true`

- Your preview pages fetch data **server-side** (getServerSideProps, server components, API routes)
- Your server needs to read query params to know which preview content to fetch
- You can't easily trigger client-side re-fetches

## Installation

```bash
npm install @contentstack/delivery-sdk @contentstack/live-preview-utils
```

## Key Differences from `ssr: false`

| Aspect | `ssr: false` | `ssr: true` |
|--------|--------------|-------------|
| Update mechanism | postMessage | Iframe refresh with query params |
| How you get preview hash | SDK provides via `ContentstackLivePreview.hash` | Read from URL query params |
| Data fetching | Client re-fetches via `onEntryChange()` | Server reads params and fetches |
| Page refresh | No | Yes (iframe reloads) |
| Stack instance | Can be shared | **New per request** (to avoid cross-session issues) |

## Implementation Steps

### 1. Configure Stack

```typescript
// lib/contentstack.ts
import contentstack from "@contentstack/delivery-sdk";

export function createStack() {
  return contentstack.stack({
    apiKey: process.env.CONTENTSTACK_API_KEY!,
    deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
    environment: process.env.CONTENTSTACK_ENVIRONMENT!,
    region: "us",
    live_preview: {
      enable: true,
      preview_token: process.env.CONTENTSTACK_PREVIEW_TOKEN,
      host: "rest-preview.contentstack.com",
    },
  });
}
```

### 2. Server-Side: Handle Preview Hash

**CRITICAL**: Create a new stack instance per request to avoid cross-session data.

```typescript
// Server-side handler
export async function getPageData(searchParams: URLSearchParams) {
  const stack = createStack(); // New instance per request
  
  const live_preview = searchParams.get("live_preview");
  const entry_uid = searchParams.get("entry_uid");
  const content_type_uid = searchParams.get("content_type_uid");

  // Apply preview query if hash exists
  if (live_preview) {
    stack.livePreviewQuery({
      live_preview,
      contentTypeUid: content_type_uid || "page",
      entryUid: entry_uid || "",
    });
  }

  const entry = await stack.contentType("page").entry("entry_uid").fetch();
  return entry;
}
```

### 3. Client-Side: Initialize SDK

```typescript
// components/LivePreviewInit.tsx
"use client";
import { useEffect } from "react";
import ContentstackLivePreview, { IStackSdk } from "@contentstack/live-preview-utils";
import { createStack } from "@/lib/contentstack";

export function LivePreviewInit() {
  useEffect(() => {
    const stack = createStack();
    
    ContentstackLivePreview.init({
      ssr: true, // SSR mode
      enable: true,
      mode: "preview",
      stackSdk: stack.config as IStackSdk,
      stackDetails: {
        apiKey: process.env.NEXT_PUBLIC_CONTENTSTACK_API_KEY!,
        environment: process.env.NEXT_PUBLIC_CONTENTSTACK_ENVIRONMENT!,
      },
      editButton: { enable: true },
    });
  }, []);

  return null;
}
```

### 4. Add to Layout

```tsx
// app/layout.tsx
import { LivePreviewInit } from "@/components/LivePreviewInit";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <LivePreviewInit />
        {children}
      </body>
    </html>
  );
}
```

## Express.js Example

```typescript
import express from "express";
import contentstack from "@contentstack/delivery-sdk";

const app = express();

app.get("*", async (req, res) => {
  // Create new stack per request
  const stack = contentstack.stack({
    apiKey: process.env.CONTENTSTACK_API_KEY!,
    deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
    environment: process.env.CONTENTSTACK_ENVIRONMENT!,
    region: "us",
    live_preview: {
      enable: true,
      preview_token: process.env.CONTENTSTACK_PREVIEW_TOKEN,
      host: "rest-preview.contentstack.com",
    },
  });

  // Apply preview hash from query params
  if (req.query.live_preview) {
    stack.livePreviewQuery(req.query);
  }

  const entry = await stack.contentType("page").entry("entry_uid").fetch();
  
  // Add editable tags
  contentstack.Utils.addEditableTags(entry, "page", true);

  // Render template with entry data
  res.render("page", { entry });
});
```

## Next.js App Router Example

```typescript
// app/[slug]/page.tsx
import contentstack from "@contentstack/delivery-sdk";
import { createStack } from "@/lib/contentstack";

export default async function Page({ 
  params, 
  searchParams 
}: { 
  params: { slug: string };
  searchParams: Promise<{ [key: string]: string | undefined }>;
}) {
  const stack = createStack();
  const { live_preview, entry_uid, content_type_uid } = await searchParams;

  // Apply preview query
  if (live_preview) {
    stack.livePreviewQuery({
      live_preview,
      contentTypeUid: content_type_uid || "page",
      entryUid: entry_uid || "",
    });
  }

  const entry = await stack.contentType("page").entry(params.slug).fetch();
  contentstack.Utils.addEditableTags(entry, "page", true);

  return (
    <div>
      <h1 {...(entry?.$ && entry?.$.title)}>{entry.title}</h1>
    </div>
  );
}
```

## GraphQL SSR Example

```typescript
import { request } from "graphql-request";

async function fetchPage(searchParams: URLSearchParams) {
  const hash = searchParams.get("live_preview");
  const isPreview = Boolean(hash);
  
  const host = isPreview 
    ? "graphql-preview.contentstack.com" 
    : "graphql.contentstack.com";

  const headers: Record<string, string> = {
    api_key: process.env.CONTENTSTACK_API_KEY!,
    access_token: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  };

  if (isPreview) {
    headers.live_preview = hash!;
    headers.preview_token = process.env.CONTENTSTACK_PREVIEW_TOKEN!;
    headers.include_applied_variants = "true";
  }

  const query = `
    query GetPage($url: String!) {
      allContentstackPage(where: { url: $url }) {
        nodes { uid title url content }
      }
    }
  `;

  return request(`https://${host}/graphql`, query, { url: "/" }, headers);
}
```

## Important Notes

### New Stack Per Request

**CRITICAL**: Always create a new stack instance for each request to avoid:
- Cross-session data leakage
- Cached preview content for wrong users
- Security issues

```typescript
// ❌ WRONG - Shared instance
const stack = createStack();

app.get("*", async (req, res) => {
  stack.livePreviewQuery(req.query); // Affects all requests!
});

// ✅ CORRECT - New instance per request
app.get("*", async (req, res) => {
  const stack = createStack();
  stack.livePreviewQuery(req.query);
});
```

### Edit Tags

Edit tags work the same as CSR:

```typescript
import contentstack from "@contentstack/delivery-sdk";

contentstack.Utils.addEditableTags(entry, "page", true, "en-us");
```

## Troubleshooting

### Page not refreshing on changes

1. Verify `ssr: true` is set in client initialization
2. Check SDK is initialized on client side
3. Verify CMS is configured with correct preview URL

### Query params not appearing

1. Check Live Preview is enabled in Contentstack environment settings
2. Verify the preview URL is correctly configured

### Wrong content shown

1. Ensure new stack instance per request (prevents cross-session data)
2. Check `livePreviewQuery` is called with the query params
3. Verify preview token is valid

### Edit buttons not showing

1. SDK must initialize on client side (even for SSR mode)
2. Editable tags must be added to entry
3. Check site loads correctly in iframe

## Important Note

**This is for preview only**. In production:
- Live Preview is disabled
- No `live_preview` query params are added
- Your site fetches published content normally

## Next Steps

- [CSR Mode](./csr-mode.md) - PostMessage-based alternative
- [Framework Patterns](../frameworks/) - Specific framework examples

