# Contentstack with Next.js

Complete patterns for Next.js App Router with Contentstack and Live Preview.

## Installation

```bash
npm install @contentstack/delivery-sdk @contentstack/live-preview-utils
```

## Stack Configuration

```typescript
// lib/contentstack.ts
import contentstack from "@contentstack/delivery-sdk";
import ContentstackLivePreview, {
  IStackSdk,
} from "@contentstack/live-preview-utils";

export const stack = contentstack.stack({
  apiKey: process.env.NEXT_PUBLIC_CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.NEXT_PUBLIC_CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.NEXT_PUBLIC_CONTENTSTACK_ENVIRONMENT!,
  region: "us",
  live_preview: {
    enable: true,
    preview_token: process.env.NEXT_PUBLIC_CONTENTSTACK_PREVIEW_TOKEN,
    host: "rest-preview.contentstack.com",
  },
});

// ssr: false = postMessage updates, client re-fetches via onEntryChange()
// ssr: true = iframe refresh with query params, server reads them
export function initLivePreview(ssrMode: boolean = false) {
  ContentstackLivePreview.init({
    ssr: ssrMode,
    enable: true,
    mode: "preview",
    stackSdk: stack.config as IStackSdk,
    stackDetails: {
      apiKey: process.env.NEXT_PUBLIC_CONTENTSTACK_API_KEY!,
      environment: process.env.NEXT_PUBLIC_CONTENTSTACK_ENVIRONMENT!,
    },
    editButton: { enable: true },
  });
}
```

## Environment Variables

```bash
# .env.local
NEXT_PUBLIC_CONTENTSTACK_API_KEY=your_api_key
NEXT_PUBLIC_CONTENTSTACK_DELIVERY_TOKEN=your_delivery_token
NEXT_PUBLIC_CONTENTSTACK_PREVIEW_TOKEN=your_preview_token
NEXT_PUBLIC_CONTENTSTACK_ENVIRONMENT=preview
```

---

## `ssr: false` Pattern (Client-Side Data Fetching)

Best for: Pages that fetch data client-side. Preview updates instantly via postMessage without iframe refresh.

```typescript
// app/page.tsx
"use client";
import { useEffect, useState } from "react";
import contentstack from "@contentstack/delivery-sdk";
import ContentstackLivePreview from "@contentstack/live-preview-utils";
import { stack, initLivePreview } from "@/lib/contentstack";

interface PageEntry {
  title: string;
  description: string;
  $?: Record<string, any>;
}

export default function HomePage() {
  const [page, setPage] = useState<PageEntry | null>(null);

  useEffect(() => {
    initLivePreview(false); // postMessage mode - client re-fetches on changes

    async function fetchPage() {
      const entry = await stack.contentType("page").entry("home").fetch();
      contentstack.Utils.addEditableTags(entry, "page", true);
      setPage(entry as PageEntry);
    }

    ContentstackLivePreview.onEntryChange(fetchPage);
    fetchPage();
  }, []);

  if (!page) return <div>Loading...</div>;

  return (
    <main>
      <h1 {...(page?.$ && page?.$.title)}>{page.title}</h1>
      <p {...(page?.$ && page?.$.description)}>{page.description}</p>
    </main>
  );
}
```

---

## `ssr: true` Pattern (Server-Side Data Fetching)

Best for: Pages that fetch data server-side. CMS refreshes iframe with query params (`?live_preview=...`).

### Server Component (Page)

```typescript
// app/[slug]/page.tsx
import contentstack from "@contentstack/delivery-sdk";
import { stack } from "@/lib/contentstack";

interface PageProps {
  params: Promise<{ slug: string }>;
  searchParams: Promise<{
    live_preview?: string;
    entry_uid?: string;
    content_type_uid?: string;
  }>;
}

export default async function Page({ params, searchParams }: PageProps) {
  const { slug } = await params;
  const { live_preview, entry_uid, content_type_uid } = await searchParams;

  // Apply preview query if hash exists
  if (live_preview) {
    stack.livePreviewQuery({
      live_preview,
      contentTypeUid: content_type_uid || "page",
      entryUid: entry_uid || "",
    });
  }

  const entry = await stack.contentType("page").entry(slug).fetch();
  contentstack.Utils.addEditableTags(entry, "page", true);

  return (
    <main>
      <h1 {...(entry?.$ && entry?.$.title)}>{entry.title}</h1>
      <p {...(entry?.$ && entry?.$.description)}>{entry.description}</p>
    </main>
  );
}
```

### Client Component (SDK Init)

```typescript
// components/LivePreviewProvider.tsx
"use client";
import { useEffect } from "react";
import { initLivePreview } from "@/lib/contentstack";

export function LivePreviewProvider({
  children,
}: {
  children: React.ReactNode;
}) {
  useEffect(() => {
    initLivePreview(true); // iframe refresh mode - CMS adds query params
  }, []);

  return <>{children}</>;
}
```

### Layout

```typescript
// app/layout.tsx
import { LivePreviewProvider } from "@/components/LivePreviewProvider";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html>
      <body>
        <LivePreviewProvider>{children}</LivePreviewProvider>
      </body>
    </html>
  );
}
```

---

## Fetching Patterns

### Get Entry by URL

```typescript
// lib/contentstack.ts
import { QueryOperation } from "@contentstack/delivery-sdk";

export async function getPageByUrl(url: string) {
  const result = await stack
    .contentType("page")
    .entry()
    .query()
    .where("url", QueryOperation.EQUALS, url)
    .find();

  const entry = result.entries[0];
  if (entry) {
    contentstack.Utils.addEditableTags(entry, "page", true);
  }
  return entry || null;
}
```

### Get Blog Posts with Pagination

```typescript
export async function getBlogPosts(page: number = 1, pageSize: number = 10) {
  const skip = (page - 1) * pageSize;

  const result = await stack
    .contentType("blog_post")
    .entry()
    .query()
    .skip(skip)
    .limit(pageSize)
    .includeCount()
    .includeReference(["author"])
    .descending("published_date")
    .find();

  return {
    posts: result.entries,
    pagination: {
      currentPage: page,
      totalPages: Math.ceil(result.count / pageSize),
      totalEntries: result.count,
    },
  };
}
```

---

## Modular Blocks

```typescript
// components/BlockRenderer.tsx
import { HeroBlock } from "./blocks/HeroBlock";
import { ContentBlock } from "./blocks/ContentBlock";
import { CTABlock } from "./blocks/CTABlock";

interface Block {
  block: {
    _content_type_uid: string;
    [key: string]: any;
  };
  _metadata: { uid: string };
}

export function BlockRenderer({ blocks }: { blocks: Block[] }) {
  return (
    <>
      {blocks?.map((item) => {
        const blockType = item.block._content_type_uid;

        switch (blockType) {
          case "hero_block":
            return <HeroBlock key={item._metadata.uid} block={item.block} />;
          case "content_block":
            return <ContentBlock key={item._metadata.uid} block={item.block} />;
          case "cta_block":
            return <CTABlock key={item._metadata.uid} block={item.block} />;
          default:
            return null;
        }
      })}
    </>
  );
}
```

---

## Image Optimization

```typescript
// components/ContentstackImage.tsx
import Image from "next/image";

interface Asset {
  url: string;
  title?: string;
  filename: string;
  dimension?: { width: number; height: number };
}

export function ContentstackImage({
  asset,
  width,
  height,
  className,
}: {
  asset: Asset;
  width?: number;
  height?: number;
  className?: string;
}) {
  // Add Contentstack image params
  const src =
    width || height
      ? `${asset.url}?${width ? `width=${width}` : ""}${
          height ? `&height=${height}` : ""
        }`
      : asset.url;

  return (
    <Image
      src={src}
      alt={asset.title || asset.filename}
      width={width || asset.dimension?.width || 800}
      height={height || asset.dimension?.height || 600}
      className={className}
    />
  );
}
```

---

## TypeScript Types

```typescript
// types/contentstack.ts
import { Entry } from "@contentstack/delivery-sdk";

export interface Asset {
  uid: string;
  url: string;
  title: string;
  filename: string;
  dimension?: { width: number; height: number };
}

export interface PageEntry extends Entry {
  title: string;
  url: string;
  description?: string;
  featured_image?: Asset;
  blocks?: Array<{
    block: any;
    _metadata: { uid: string };
  }>;
  $?: Record<string, any>;
}

export interface BlogPost extends Entry {
  title: string;
  url: string;
  excerpt: string;
  content: string;
  published_date: string;
  author?: Author;
  featured_image?: Asset;
  $?: Record<string, any>;
}

export interface Author extends Entry {
  name: string;
  bio: string;
  avatar?: Asset;
}
```

---

## Next.js Config

```typescript
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "images.contentstack.io",
      },
      {
        protocol: "https",
        hostname: "**.contentstack.com",
      },
    ],
  },
};

export default nextConfig;
```

---

## Troubleshooting

### Hydration Errors

Ensure client components are properly marked with `"use client"` directive.

### Preview Not Working

1. Check HTTPS is enabled
2. Verify preview token is correct
3. Ensure site allows iframe embedding
4. Check `searchParams` is awaited (Next.js 15+)

### Images Not Loading

Add Contentstack domains to `next.config.ts` image configuration.
