# Contentstack Live Preview Integration Guide for Next.js SSR

This guide provides focused instructions for AI coding assistants to integrate Contentstack Live Preview into an existing Next.js application using Server-Side Rendering (SSR) mode.

## Prerequisites

- Existing Next.js application with App Directory structure
- Contentstack stack with Live Preview enabled
- Delivery token with preview scope generated
- Website served over HTTPS and iframe-compatible

## Required Dependencies

Install these Contentstack-specific packages:

```bash
npm install @contentstack/delivery-sdk @contentstack/live-preview-utils isomorphic-dompurify
```

**Optional but recommended**: Install the endpoints helper package for easier region management:

```bash
npm install @timbenniks/contentstack-endpoints
```

This package provides all Contentstack endpoints for different regions and cloud providers, eliminating the need to manually maintain endpoint URLs.

## Environment Variables

Create a `.env.local` file in your Next.js root directory:

```env
CONTENTSTACK_API_KEY=your_api_key
CONTENTSTACK_DELIVERY_TOKEN=your_delivery_token
CONTENTSTACK_ENVIRONMENT=your_environment
CONTENTSTACK_REGION=us
CONTENTSTACK_LIVE_PREVIEW=your_live_preview_token
CONTENTSTACK_LIVE_EDIT_PREVIEW=your_preview_token
```

## Stack Configuration

Configure your Contentstack SDK client with region-specific endpoints. You have two options:

### Option 1: Using @timbenniks/contentstack-endpoints (Recommended)

This package provides all official Contentstack endpoints and is automatically updated:

```typescript
// lib/contentstack.ts
import { Region, Stack, addEditableTags } from "@contentstack/delivery-sdk";
import { getContentstackEndpoints } from "@timbenniks/contentstack-endpoints";

const stackConfig = {
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region:
    Region[
      process.env.CONTENTSTACK_REGION?.toUpperCase() as keyof typeof Region
    ] || Region.US,
  live_preview: {
    enable: true,
    preview_token: process.env.CONTENTSTACK_LIVE_PREVIEW!,
    host: getContentstackEndpoints(process.env.CONTENTSTACK_REGION || "us")
      .rest_api_url,
  },
};

export const stack = new Stack(stackConfig);

// Enable live edit tags for SSR
addEditableTags(stack, "data-cslp");
```

### Option 2: Manual Endpoint Configuration

If you prefer not to use the endpoints package, configure manually:

```typescript
// lib/contentstack.ts
import { Region, Stack, addEditableTags } from "@contentstack/delivery-sdk";

const getContentstackHost = (region: string) => {
  const hosts = {
    us: "https://api.contentstack.io",
    eu: "https://eu-api.contentstack.com",
    au: "https://au-api.contentstack.com",
    azure_na: "https://azure-na-api.contentstack.com",
    azure_eu: "https://azure-eu-api.contentstack.com",
    gcp_na: "https://gcp-na-api.contentstack.com",
  };
  return hosts[region as keyof typeof hosts] || hosts.us;
};

const stackConfig = {
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region:
    Region[
      process.env.CONTENTSTACK_REGION?.toUpperCase() as keyof typeof Region
    ] || Region.US,
  live_preview: {
    enable: true,
    preview_token: process.env.CONTENTSTACK_LIVE_PREVIEW!,
    host: getContentstackHost(process.env.CONTENTSTACK_REGION || "us"),
  },
};

export const stack = new Stack(stackConfig);

// Enable live edit tags for SSR
addEditableTags(stack, "data-cslp");
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

// Initialize Live Preview functionality for SSR
export function initLivePreview() {
  ContentstackLivePreview.init({
    ssr: true, // SSR mode - CRITICAL for server-side rendering
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
// Example: Fetch page data with Live Preview support
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

### 3. Client-Side Live Preview Component

Create a client-side component to initialize Live Preview:

```typescript
// components/ContentstackLivePreview.tsx
"use client";

import { initLivePreview } from "@/lib/contentstack";
import React, { useEffect } from "react";

export function ContentstackLivePreview({
  children,
}: {
  children?: React.ReactNode;
}) {
  const livePreviewEnabled =
    process.env.NEXT_PUBLIC_CONTENTSTACK_PREVIEW === "true";

  useEffect(() => {
    if (livePreviewEnabled) {
      initLivePreview();
    }
  }, [livePreviewEnabled]);

  return <>{children}</>;
}
```

### 4. Server-Side Page Component Integration

Integrate Live Preview into your server-side page components:

```typescript
// app/page.tsx or any server component
import DOMPurify from "isomorphic-dompurify";
import { getPage, stack } from "@/lib/contentstack";
import { headers } from "next/headers";
import Image from "next/image";

export default async function HomePage({
  searchParams,
}: {
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;
}) {
  // CRITICAL: Await headers() for Next.js compatibility
  await headers();

  // Extract Live Preview parameters from search params
  const { live_preview, entry_uid, content_type_uid } = await searchParams;

  // STEP 1: Apply Live Preview query parameters to stack
  if (live_preview) {
    stack.livePreviewQuery({
      live_preview: live_preview as string,
      contentTypeUid: (content_type_uid as string) || "",
      entryUid: (entry_uid as string) || "",
    });
  }

  // STEP 2: Fetch content with Live Preview context
  const page = await getPage("/");

  return (
    <main>
      <section>
        {/* Optional: Display Live Preview debug info */}
        {live_preview && (
          <div className="mb-4 p-2 bg-yellow-100 text-sm">
            <p>Live Preview Active</p>
            <p>Hash: {live_preview}</p>
            <p>Content Type: {content_type_uid}</p>
            <p>Entry: {entry_uid}</p>
          </div>
        )}

        {/* CRITICAL: Spread editable tags for Live Preview */}
        {page?.title && <h1 {...(page?.$ && page?.$.title)}>{page.title}</h1>}

        {page?.description && (
          <p {...(page?.$ && page?.$.description)}>{page.description}</p>
        )}

        {page?.image && (
          <Image
            src={page.image.url}
            alt={page.image.title}
            width={768}
            height={414}
            {...(page?.image?.$ && page?.image?.$.url)}
          />
        )}

        {page?.rich_text && (
          <div
            {...(page?.$ && page?.$.rich_text)}
            dangerouslySetInnerHTML={{
              __html: DOMPurify.sanitize(page.rich_text),
            }}
          />
        )}

        {/* For modular blocks/arrays */}
        <div {...(page?.$ && page?.$.blocks)}>
          {page?.blocks?.map((item, index) => (
            <div
              key={item.block._metadata?.uid}
              {...(page?.$ && page?.$[`blocks__${index}`])}
            >
              {/* Block content with editable tags */}
              {item.block.title && (
                <h2 {...(item.block?.$ && item.block?.$.title)}>
                  {item.block.title}
                </h2>
              )}
              {item.block.copy && (
                <div
                  {...(item.block?.$ && item.block?.$.copy)}
                  dangerouslySetInnerHTML={{
                    __html: DOMPurify.sanitize(item.block.copy),
                  }}
                />
              )}
            </div>
          ))}
        </div>
      </section>
    </main>
  );
}
```

### 5. Layout Integration

Add the Live Preview component to your root layout:

```typescript
// app/layout.tsx
import type { Metadata } from "next";
import "./globals.css";
import { ContentstackLivePreview } from "@/components/ContentstackLivePreview";

export const metadata: Metadata = {
  title: "Your App with Contentstack Live Preview",
  description: "Next.js app with Contentstack Live Preview SSR",
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body>
        {children}
        {/* CRITICAL: Add Live Preview component at the end */}
        <ContentstackLivePreview />
      </body>
    </html>
  );
}
```

## SSR Live Preview Integration Patterns

### Pattern 1: Single Entry Page

```typescript
// For individual pages (blog post, product page, etc.)
export default async function SinglePage({
  params,
  searchParams,
}: {
  params: Promise<{ slug: string }>;
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;
}) {
  await headers();
  const { slug } = await params;
  const { live_preview, entry_uid, content_type_uid } = await searchParams;

  // Apply Live Preview context
  if (live_preview) {
    stack.livePreviewQuery({
      live_preview: live_preview as string,
      contentTypeUid: (content_type_uid as string) || "",
      entryUid: (entry_uid as string) || "",
    });
  }

  const entry = await getSingleEntry(slug);

  return (
    <div>
      <h1 {...(entry?.$ && entry?.$.title)}>{entry?.title}</h1>
      {/* Rest of your content */}
    </div>
  );
}
```

### Pattern 2: List/Collection Page

```typescript
// For listing pages (blog index, product catalog, etc.)
export default async function ListPage({
  searchParams,
}: {
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;
}) {
  await headers();
  const { live_preview, entry_uid, content_type_uid } = await searchParams;

  if (live_preview) {
    stack.livePreviewQuery({
      live_preview: live_preview as string,
      contentTypeUid: (content_type_uid as string) || "",
      entryUid: (entry_uid as string) || "",
    });
  }

  const entries = await getMultipleEntries();

  return (
    <div>
      {entries?.map((entry, index) => (
        <article key={entry.uid} {...(entry?.$ && entry?.$[`${index}`])}>
          <h2 {...(entry?.$ && entry?.$.title)}>{entry.title}</h2>
        </article>
      ))}
    </div>
  );
}
```

### Pattern 3: Dynamic Route with Live Preview

```typescript
// For dynamic routes like [slug]/page.tsx
export default async function DynamicPage({
  params,
  searchParams,
}: {
  params: Promise<{ slug: string }>;
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;
}) {
  await headers();
  const { slug } = await params;
  const { live_preview, entry_uid, content_type_uid } = await searchParams;

  if (live_preview) {
    stack.livePreviewQuery({
      live_preview: live_preview as string,
      contentTypeUid: (content_type_uid as string) || "",
      entryUid: (entry_uid as string) || "",
    });
  }

  const pageData = await getPageBySlug(slug);

  return <div>{/* Your page content with editable tags */}</div>;
}
```

## Critical Implementation Notes

### SSR Live Preview Requirements

1. **Server-Side Only**: Use `ssr: true` in the Live Preview initialization
2. **Query Parameter Handling**: Always extract and apply `live_preview`, `entry_uid`, and `content_type_uid` from searchParams
3. **Stack Query Application**: Call `stack.livePreviewQuery()` BEFORE fetching content
4. **Client-Side Initialization**: Live Preview SDK must still be initialized on the client side
5. **Page Refresh Behavior**: SSR mode refreshes the entire page when content changes

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

// ALWAYS apply Live Preview query before fetching
if (live_preview) {
  stack.livePreviewQuery({
    live_preview: live_preview as string,
    contentTypeUid: content_type_uid || "",
    entryUid: entry_uid || "",
  });
}

// ALWAYS add editable tags after fetching
if (process.env.NEXT_PUBLIC_CONTENTSTACK_PREVIEW === "true") {
  contentstack.Utils.addEditableTags(entry, "content_type_uid", true);
}
```

### JSX Integration Requirements

```jsx
// REQUIRED: Spread editable tags for all fields
<h1 {...(entry?.$ && entry?.$.title)}>{entry.title}</h1>;

// REQUIRED: Handle arrays with index-based tags
{
  content?.blocks?.map((item, index) => (
    <div {...(content?.$ && content?.$[`blocks__${index}`])}>
      {/* block content */}
    </div>
  ));
}

// REQUIRED: Use isomorphic-dompurify for SSR compatibility
import DOMPurify from "isomorphic-dompurify";

<div
  dangerouslySetInnerHTML={{
    __html: DOMPurify.sanitize(content.rich_text),
  }}
/>;
```

## Testing Live Preview Integration

1. **Enable Live Preview in Contentstack**: Settings → Live Preview → Enable and select Preview environment
2. **Test in CMS**: Open any entry → Click Live Preview icon in sidebar → Make changes to see page refresh with updates
3. **Verify Parameters**: Check that live_preview, entry_uid, and content_type_uid appear in URL parameters
4. **Check Edit Buttons**: Ensure edit buttons appear on tagged elements in the preview

## Troubleshooting SSR Live Preview

### Common Integration Issues

1. **Page not refreshing**: Verify `stack.livePreviewQuery()` is called before content fetching
2. **Missing edit buttons**: Check that editable tags are properly spread in JSX
3. **Query parameters not working**: Ensure `searchParams` is properly awaited and destructured
4. **Client-side errors**: Verify `ContentstackLivePreview` component is added to layout

### Debug Checklist

- [ ] `ssr: true` set in Live Preview initialization
- [ ] Environment variables properly configured
- [ ] Preview token has correct scopes in Contentstack
- [ ] `stack.livePreviewQuery()` called before content fetching
- [ ] Editable tags spread using `{...entry?.$ && entry?.$.field}` pattern
- [ ] `await headers()` called in server components
- [ ] `ContentstackLivePreview` component added to layout

This focused integration guide enables AI agents to add SSR Live Preview functionality to existing Next.js applications with proper server-side rendering and page refresh behavior.
