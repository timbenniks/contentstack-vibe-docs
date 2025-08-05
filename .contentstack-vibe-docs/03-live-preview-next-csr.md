# Contentstack Live Preview Integration Guide for Next.js CSR

This guide provides focused instructions for AI coding assistants to integrate Contentstack Live Preview into an existing Next.js application using Client-Side Rendering (CSR) mode.

## Prerequisites

- Existing Next.js application with App Directory structure
- Contentstack stack with Live Preview enabled
- Delivery token with preview scope generated
- Website served over HTTPS and iframe-compatible

## Required Dependencies

Install these Contentstack-specific packages:

```bash
npm install @contentstack/delivery-sdk @contentstack/live-preview-utils
```

**Optional but recommended**: Install the endpoints helper package for easier region management:

```bash
npm install @timbenniks/contentstack-endpoints
```

This package provides all Contentstack endpoints for different regions and cloud providers, eliminating the need to manually maintain endpoint URLs.

## Environment Variables

Add these variables to your `.env.local` file:

```env
NEXT_PUBLIC_CONTENTSTACK_API_KEY=<YOUR_API_KEY>
NEXT_PUBLIC_CONTENTSTACK_DELIVERY_TOKEN=<YOUR_DELIVERY_TOKEN>
NEXT_PUBLIC_CONTENTSTACK_PREVIEW_TOKEN=<YOUR_PREVIEW_TOKEN>
NEXT_PUBLIC_CONTENTSTACK_REGION=EU
NEXT_PUBLIC_CONTENTSTACK_ENVIRONMENT=preview
NEXT_PUBLIC_CONTENTSTACK_PREVIEW=true
```

## Core Implementation

### 1. Contentstack SDK Configuration with Live Preview

Create or update your Contentstack configuration file:

```typescript
// lib/contentstack.ts
import contentstack, { QueryOperation } from "@contentstack/delivery-sdk";
import ContentstackLivePreview, {
  IStackSdk,
} from "@contentstack/live-preview-utils";

// Option 1: Use the @timbenniks/contentstack-endpoints package (recommended)
// Install: npm install @timbenniks/contentstack-endpoints
import {
  getContentstackEndpoints,
  getRegionForString,
} from "@timbenniks/contentstack-endpoints";

const region = getRegionForString(
  process.env.NEXT_PUBLIC_CONTENTSTACK_REGION || "EU"
);
const endpoints = getContentstackEndpoints(region, true); // true = omit https://

// Option 2: Manual helper function (if you prefer not to use the package)
function getRegionEndpoints(region: string) {
  const endpoints = {
    EU: {
      preview: "eu-rest-preview.contentstack.com",
      application: "eu-app.contentstack.com",
    },
    US: {
      preview: "rest-preview.contentstack.com",
      application: "app.contentstack.com",
    },
  };
  return endpoints[region as keyof typeof endpoints] || endpoints.US;
}

// If using Option 2, uncomment these lines and comment out Option 1:
// const region = process.env.NEXT_PUBLIC_CONTENTSTACK_REGION || "EU";
// const endpoints = getRegionEndpoints(region);

// Configure Contentstack SDK with Live Preview support
export const stack = contentstack.stack({
  apiKey: process.env.NEXT_PUBLIC_CONTENTSTACK_API_KEY as string,
  deliveryToken: process.env.NEXT_PUBLIC_CONTENTSTACK_DELIVERY_TOKEN as string,
  environment: process.env.NEXT_PUBLIC_CONTENTSTACK_ENVIRONMENT as string,
  region: region as any,
  live_preview: {
    enable: process.env.NEXT_PUBLIC_CONTENTSTACK_PREVIEW === "true",
    preview_token: process.env.NEXT_PUBLIC_CONTENTSTACK_PREVIEW_TOKEN,
    host: endpoints.preview,
  },
});

// Initialize Live Preview functionality for CSR
export function initLivePreview() {
  ContentstackLivePreview.init({
    ssr: false, // CSR mode - CRITICAL for client-side rendering
    enable: process.env.NEXT_PUBLIC_CONTENTSTACK_PREVIEW === "true",
    mode: "builder", // Supports both Live Preview and Visual Builder
    stackSdk: stack.config as IStackSdk,
    stackDetails: {
      apiKey: process.env.NEXT_PUBLIC_CONTENTSTACK_API_KEY as string,
      environment: process.env.NEXT_PUBLIC_CONTENTSTACK_ENVIRONMENT as string,
    },
    clientUrlParams: {
      host: endpoints.application,
    },
    editButton: {
      enable: true,
      exclude: ["outsideLivePreviewPortal"],
    },
  });
}
```

### 2. Data Fetching with Live Preview Support

Create data fetching functions that support Live Preview:

```typescript
// Example: Fetch page data for URL with Live Preview support
export async function getPage(url: string) {
  const result = await stack
    .contentType("page")
    .entry()
    .query()
    .where("url", QueryOperation.EQUALS, url)
    .find();

  if (result.entries) {
    const entry = result.entries[0];

    // CRITICAL: Add editable tags for Live Preview
    if (process.env.NEXT_PUBLIC_CONTENTSTACK_PREVIEW === "true") {
      contentstack.Utils.addEditableTags(entry, "page", true);
    }

    return entry;
  }
}

// Example: Fetch multiple entries with Live Preview support
export async function getBlogPosts() {
  const result = await stack.contentType("blog_post").entry().query().find();

  if (result.entries) {
    const entries = result.entries;

    // Add editable tags to all entries
    if (process.env.NEXT_PUBLIC_CONTENTSTACK_PREVIEW === "true") {
      entries.forEach((entry) => {
        contentstack.Utils.addEditableTags(entry, "blog_post", true);
      });
    }

    return entries;
  }
}
```

### 3. Component Integration with Live Preview

Integrate Live Preview into your React components:

```typescript
"use client";

import { useEffect, useState } from "react";
import ContentstackLivePreview, {
  VB_EmptyBlockParentClass,
} from "@contentstack/live-preview-utils";
import { getPage, initLivePreview } from "@/lib/contentstack";

export default function YourPageComponent() {
  const [content, setContent] = useState(null);

  // Fetch content function
  const fetchContent = async () => {
    const data = await getPage("/your-page-url");
    setContent(data);
  };

  useEffect(() => {
    // STEP 1: Initialize Live Preview once
    initLivePreview();

    // STEP 2: Set up real-time content updates
    ContentstackLivePreview.onEntryChange(fetchContent);

    // STEP 3: Initial content fetch
    fetchContent();
  }, []);

  return (
    <div>
      {/* CRITICAL: Spread editable tags for Live Preview */}
      {content?.title && (
        <h1 {...(content?.$ && content?.$.title)}>{content.title}</h1>
      )}

      {content?.description && (
        <p {...(content?.$ && content?.$.description)}>{content.description}</p>
      )}

      {/* For modular blocks/arrays */}
      <div
        className={!content?.blocks?.length ? VB_EmptyBlockParentClass : ""}
        {...(content?.$ && content?.$.blocks)}
      >
        {content?.blocks?.map((item, index) => (
          <div
            key={item._metadata?.uid}
            {...(content?.$ && content?.$[`blocks__${index}`])}
          >
            {/* Block content with editable tags */}
            {item.title && (
              <h2 {...(item?.$ && item?.$.title)}>{item.title}</h2>
            )}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Live Preview Integration Patterns

### Pattern 1: Single Entry Live Preview

```typescript
// For pages, articles, or single content entries
useEffect(() => {
  initLivePreview();

  const fetchData = async () => {
    const entry = await getSingleEntry("entry_uid");
    setContent(entry);
  };

  ContentstackLivePreview.onEntryChange(fetchData);
  fetchData();
}, []);
```

### Pattern 2: List/Collection Live Preview

```typescript
// For blog lists, product catalogs, etc.
useEffect(() => {
  initLivePreview();

  const fetchData = async () => {
    const entries = await getMultipleEntries();
    setContentList(entries);
  };

  ContentstackLivePreview.onEntryChange(fetchData);
  fetchData();
}, []);
```

### Pattern 3: Nested/Modular Content Live Preview

```typescript
// For complex page builders with modular blocks
useEffect(() => {
  initLivePreview();

  const fetchData = async () => {
    const page = await getPageWithModularBlocks(slug);
    setPageData(page);
  };

  ContentstackLivePreview.onEntryChange(fetchData);
  fetchData();
}, []);
```

## Essential Live Edit Tags Implementation

### Basic Field Tags

```jsx
// Text fields
<h1 {...(entry?.$ && entry?.$.title)}>{entry.title}</h1>

// Rich text fields
<div
  {...(entry?.$ && entry?.$.rich_text)}
  dangerouslySetInnerHTML={{ __html: entry.rich_text }}
/>

// Image fields
<img
  src={entry.image?.url}
  alt={entry.image?.title}
  {...(entry?.image?.$ && entry?.image?.$.url)}
/>
```

### Modular Blocks Tags

```jsx
// Parent container for blocks
<div {...(entry?.$ && entry?.$.blocks)}>
  {entry?.blocks?.map((block, index) => (
    <div
      key={block._metadata?.uid}
      {...(entry?.$ && entry?.$[`blocks__${index}`])}
    >
      {/* Block fields with their own tags */}
      <h2 {...(block?.$ && block?.$.title)}>{block.title}</h2>
    </div>
  ))}
</div>
```

### Reference Fields Tags

```jsx
// Referenced entries
<div {...(entry?.$ && entry?.$.referenced_field)}>
  {entry.referenced_field?.map((ref, index) => (
    <div
      key={ref.uid}
      {...(entry?.$ && entry?.$[`referenced_field__${index}`])}
    >
      <h3 {...(ref?.$ && ref?.$.title)}>{ref.title}</h3>
    </div>
  ))}
</div>
```

## Critical Implementation Notes

### Live Preview Initialization Requirements

1. **Client-Side Only**: Always use `"use client"` directive and `ssr: false`
2. **Single Initialization**: Call `initLivePreview()` only once per page
3. **Event Handling**: Use `ContentstackLivePreview.onEntryChange()` for real-time updates
4. **Editable Tags**: Always call `contentstack.Utils.addEditableTags()` with third parameter `true`

### Data Fetching Best Practices

```typescript
// ALWAYS use the TypeScript Delivery SDK
import contentstack from "@contentstack/delivery-sdk";

// CRITICAL: Configure live_preview in stack initialization
const stack = contentstack.stack({
  // ... other config
  live_preview: {
    enable: process.env.NEXT_PUBLIC_CONTENTSTACK_PREVIEW === "true",
    preview_token: process.env.NEXT_PUBLIC_CONTENTSTACK_PREVIEW_TOKEN,
    host: "your-region-preview-host.com",
  },
});

// ALWAYS add editable tags after fetching
if (process.env.NEXT_PUBLIC_CONTENTSTACK_PREVIEW === "true") {
  contentstack.Utils.addEditableTags(entry, "content_type_uid", true);
}
```

### JSX Integration Requirements

```jsx
// REQUIRED: Spread editable tags for all fields
<h1 {...(entry?.$ && entry?.$.title)}>{entry.title}</h1>

// REQUIRED: Use VB_EmptyBlockParentClass for empty modular blocks
<div className={!content?.blocks?.length ? VB_EmptyBlockParentClass : ""}>
  {/* blocks content */}
</div>

// REQUIRED: Index-based tags for array fields
{content?.blocks?.map((item, index) => (
  <div {...(content?.$ && content?.$[`blocks__${index}`])}>
    {/* block content */}
  </div>
))}
```

## Testing Live Preview Integration

1. **Enable Live Preview in Contentstack**: Settings → Live Preview → Enable and select Preview environment
2. **Test in CMS**: Open any entry → Click Live Preview icon in sidebar → Make changes to see real-time updates
3. **Verify Edit Buttons**: Ensure edit buttons appear on tagged elements in the preview

## Troubleshooting Live Preview

### Common Integration Issues

1. **No real-time updates**: Verify `onEntryChange` is set up in `useEffect`
2. **Missing edit buttons**: Check that editable tags are properly spread in JSX
3. **Preview not loading**: Ensure HTTPS and iframe compatibility
4. **Wrong content shown**: Verify `contentstack.Utils.addEditableTags()` is called with correct content type UID

### Debug Checklist

- [ ] `ssr: false` set in Live Preview initialization
- [ ] Environment variables properly configured
- [ ] Preview token has correct scopes in Contentstack
- [ ] Editable tags spread using `{...entry?.$ && entry?.$.field}` pattern
- [ ] `initLivePreview()` called only once per component
- [ ] Browser console shows Live Preview connection messages

This focused integration guide enables AI agents to add Live Preview functionality to existing Next.js applications without rebuilding the entire app structure.
