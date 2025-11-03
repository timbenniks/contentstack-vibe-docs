# Contentstack Practical Examples for AI Agents

This guide provides real-world implementation examples that AI coding assistants need when building applications with Contentstack.

**Note**: Each example includes stack initialization for clarity. In production, you should create a single stack instance and reuse it across your application. Consider creating a shared module like `lib/contentstack.ts`:

```typescript
// lib/contentstack.ts
import contentstack from "@contentstack/delivery-sdk";

export const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION || "us",
});
```

## Table of Contents

1. [Rendering Rich Text](#rendering-rich-text)
2. [Working with References](#working-with-references)
3. [Handling Modular Blocks](#handling-modular-blocks)
4. [Asset Transformations](#asset-transformations)
5. [Common Patterns](#common-patterns)
6. [TypeScript Types](#typescript-types)
7. [Locale Handling](#locale-handling)

## Rendering Rich Text

### React (with DOMPurify)

```typescript
import DOMPurify from "isomorphic-dompurify";

interface BlogPost {
  title: string;
  content: string; // richtext field
}

function BlogPost({ post }: { post: BlogPost }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <div
        dangerouslySetInnerHTML={{
          __html: DOMPurify.sanitize(post.content),
        }}
      />
    </article>
  );
}
```

### Vue 3

```vue
<script setup>
import DOMPurify from "dompurify";

const props = defineProps<{
  post: {
    title: string;
    content: string;
  };
}>();

const sanitizedContent = computed(() => {
  return DOMPurify.sanitize(props.post.content);
});
</script>

<template>
  <article>
    <h1>{{ post.title }}</h1>
    <div v-html="sanitizedContent"></div>
  </article>
</template>
```

### Next.js (Server Component)

```typescript
import DOMPurify from "isomorphic-dompurify";

export default async function BlogPost({ entry }: { entry: BlogPost }) {
  return (
    <article>
      <h1>{entry.title}</h1>
      <div
        dangerouslySetInnerHTML={{
          __html: DOMPurify.sanitize(entry.content),
        }}
      />
    </article>
  );
}
```

## Working with References

### Single Reference

```typescript
import contentstack from "@contentstack/delivery-sdk";

const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION || "us",
});

// Fetch entry with author reference
const entry = await stack
  .contentType("blog_post")
  .entry("entry_uid")
  .includeReference(["author"])
  .fetch();

// Render in React
function BlogPost({ entry }: { entry: BlogPost }) {
  return (
    <article>
      <h1>{entry.title}</h1>
      {entry.author && (
        <div className="author">
          <img src={entry.author.avatar?.url} alt={entry.author.name} />
          <span>{entry.author.name}</span>
        </div>
      )}
    </article>
  );
}
```

### Multiple References

```typescript
import contentstack from "@contentstack/delivery-sdk";

const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION || "us",
});

// Fetch entry with multiple references
const entry = await stack
  .contentType("blog_post")
  .entry("entry_uid")
  .includeReference(["author", "category", "tags"])
  .fetch();

// Render in React
function BlogPost({ entry }: { entry: BlogPost }) {
  return (
    <article>
      <h1>{entry.title}</h1>
      {entry.author && <Author author={entry.author} />}
      {entry.category && <Category category={entry.category} />}
      {entry.tags && (
        <div className="tags">
          {entry.tags.map((tag) => (
            <span key={tag.uid}>{tag.title}</span>
          ))}
        </div>
      )}
    </article>
  );
}
```

### Reference Arrays

```typescript
import contentstack from "@contentstack/delivery-sdk";

const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION || "us",
});

// Entry has a reference field that's an array
const entry = await stack
  .contentType("page")
  .entry("entry_uid")
  .includeReference(["related_posts"])
  .fetch();

// Render array of references
function Page({ entry }: { entry: Page }) {
  return (
    <div>
      <h1>{entry.title}</h1>
      {entry.related_posts && entry.related_posts.length > 0 && (
        <section>
          <h2>Related Posts</h2>
          {entry.related_posts.map((post) => (
            <article key={post.uid}>
              <h3>{post.title}</h3>
              <p>{post.excerpt}</p>
            </article>
          ))}
        </section>
      )}
    </div>
  );
}
```

## Handling Modular Blocks

### Basic Modular Blocks

```typescript
import contentstack from "@contentstack/delivery-sdk";
import DOMPurify from "isomorphic-dompurify";

const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION || "us",
});

// Fetch entry with modular blocks
const entry = await stack.contentType("page").entry("entry_uid").fetch();

// Entry structure:
// {
//   title: "Page Title",
//   blocks: [
//     { block: { title: "Hero", ... }, _metadata: { uid: "..." } },
//     { block: { title: "Content", ... }, _metadata: { uid: "..." } }
//   ]
// }

// Render modular blocks in React
function Page({ entry }: { entry: Page }) {
  return (
    <div>
      <h1>{entry.title}</h1>
      {entry.blocks?.map((blockItem) => {
        const block = blockItem.block;
        const blockType = block._content_type_uid;

        switch (blockType) {
          case "hero_block":
            return <HeroBlock key={blockItem._metadata.uid} block={block} />;
          case "content_block":
            return <ContentBlock key={blockItem._metadata.uid} block={block} />;
          case "cta_block":
            return <CTABlock key={blockItem._metadata.uid} block={block} />;
          default:
            return null;
        }
      })}
    </div>
  );
}

// Individual block components
function HeroBlock({ block }: { block: HeroBlock }) {
  return (
    <section className="hero">
      <h2>{block.title}</h2>
      {block.image && <img src={block.image.url} alt={block.image.title} />}
      {block.description && <p>{block.description}</p>}
    </section>
  );
}

function ContentBlock({ block }: { block: ContentBlock }) {
  return (
    <section className="content">
      <h2>{block.title}</h2>
      <div
        dangerouslySetInnerHTML={{
          __html: DOMPurify.sanitize(block.content),
        }}
      />
    </section>
  );
}
```

### Modular Blocks with Vue

```vue
<script setup>
const props = defineProps<{
  entry: {
    title: string;
    blocks?: Array<{
      block: any;
      _metadata: { uid: string };
    }>;
  };
}>();

function getBlockComponent(blockType: string) {
  switch (blockType) {
    case "hero_block":
      return "HeroBlock";
    case "content_block":
      return "ContentBlock";
    case "cta_block":
      return "CTABlock";
    default:
      return null;
  }
}
</script>

<template>
  <div>
    <h1>{{ entry.title }}</h1>
    <component
      v-for="blockItem in entry.blocks"
      :key="blockItem._metadata.uid"
      :is="getBlockComponent(blockItem.block._content_type_uid)"
      :block="blockItem.block"
    />
  </div>
</template>
```

## Asset Transformations

Contentstack provides comprehensive image transformation via the Images API. All transformations are performed on-the-fly and cached automatically.

### Image Transformation Parameters

The Images API supports various transformation parameters:

- `width`: Resize image width (in pixels)
- `height`: Resize image height (in pixels)
- `quality`: JPEG quality (1-100, default: 75)
- `format`: Output format (`auto`, `webp`, `jpeg`, `png`, `gif`)
- `fit`: Fit mode (`bounds`, `crop`, `pad`)
- `auto`: Automatic optimization (`webp`, `compress`)

### Basic Image URLs

```typescript
import contentstack from "@contentstack/delivery-sdk";

const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION || "us",
});

// Basic asset URL
const imageUrl = entry.image.url; // https://images.contentstack.io/v3/assets/...

// Transform image URL with width and height
function getImageUrl(asset: Asset, width?: number, height?: number) {
  let url = asset.url;
  const params = new URLSearchParams();

  if (width) params.append("width", width.toString());
  if (height) params.append("height", height.toString());

  if (params.toString()) {
    url += `?${params.toString()}`;
  }

  return url;
}

// Usage
const thumbnailUrl = getImageUrl(entry.image, 400, 300);
```

### Advanced Image Transformations

```typescript
import contentstack from "@contentstack/delivery-sdk";

interface ImageTransformOptions {
  width?: number;
  height?: number;
  quality?: number; // 1-100
  format?: "auto" | "webp" | "jpeg" | "png" | "gif";
  fit?: "bounds" | "crop" | "pad";
  auto?: "webp" | "compress";
}

function transformImageUrl(
  asset: Asset,
  options: ImageTransformOptions
): string {
  const params = new URLSearchParams();

  if (options.width) params.append("width", options.width.toString());
  if (options.height) params.append("height", options.height.toString());
  if (options.quality) params.append("quality", options.quality.toString());
  if (options.format) params.append("format", options.format);
  if (options.fit) params.append("fit", options.fit);
  if (options.auto) params.append("auto", options.auto);

  return params.toString() ? `${asset.url}?${params.toString()}` : asset.url;
}

// Examples
const optimizedWebp = transformImageUrl(entry.image, {
  width: 800,
  format: "webp",
  quality: 85,
  auto: "webp",
});

const croppedThumbnail = transformImageUrl(entry.image, {
  width: 400,
  height: 300,
  fit: "crop",
});

const autoOptimized = transformImageUrl(entry.image, {
  width: 1200,
  auto: "compress",
});
```

### Image Component (React)

```typescript
import Image from "next/image"; // Next.js Image component

function BlogPostImage({ asset }: { asset: Asset }) {
  return (
    <Image
      src={asset.url}
      alt={asset.title || asset.filename}
      width={asset.dimension?.width || 800}
      height={asset.dimension?.height || 600}
      // Next.js Image optimization works with Contentstack CDN
    />
  );
}
```

### Responsive Images

```typescript
function ResponsiveImage({ asset }: { asset: Asset }) {
  const srcSet = [
    `${asset.url}?width=400 400w`,
    `${asset.url}?width=800 800w`,
    `${asset.url}?width=1200 1200w`,
  ].join(", ");

  return (
    <img
      src={asset.url}
      srcSet={srcSet}
      sizes="(max-width: 768px) 400px, (max-width: 1200px) 800px, 1200px"
      alt={asset.title || asset.filename}
    />
  );
}
```

## Common Patterns

### Blog Listing Page

```typescript
import contentstack from "@contentstack/delivery-sdk";
import { useState, useEffect } from "react";
import Link from "next/link"; // or your routing library

const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION || "us",
});

// Fetch blog posts with pagination
async function getBlogPosts(page: number = 1, pageSize: number = 10) {
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
    posts: result.entries,
    pagination: {
      currentPage: page,
      pageSize,
      totalEntries: result.count,
      totalPages: Math.ceil(result.count / pageSize),
    },
  };
}

// React component
function BlogListing() {
  const [page, setPage] = useState(1);
  const [data, setData] = useState(null);

  useEffect(() => {
    getBlogPosts(page).then(setData);
  }, [page]);

  if (!data) return <div>Loading...</div>;

  return (
    <div>
      <h1>Blog Posts</h1>
      {data.posts.map((post) => (
        <article key={post.uid}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
          <Link href={`/blog/${post.url}`}>Read more</Link>
        </article>
      ))}
      <Pagination
        currentPage={data.pagination.currentPage}
        totalPages={data.pagination.totalPages}
        onPageChange={setPage}
      />
    </div>
  );
}
```

### Single Blog Post Page

```typescript
import contentstack, { QueryOperation } from "@contentstack/delivery-sdk";
import { useState, useEffect } from "react";
import DOMPurify from "isomorphic-dompurify";

const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION || "us",
});

// Fetch single post by URL
async function getBlogPostByUrl(url: string) {
  const result = await stack
    .contentType("blog_post")
    .entry()
    .query()
    .where("url", QueryOperation.EQUALS, url)
    .includeReference(["author", "category", "tags"])
    .find();

  return result.entries[0] || null;
}

// React component
function BlogPost({ url }: { url: string }) {
  const [post, setPost] = useState(null);

  useEffect(() => {
    getBlogPostByUrl(url).then(setPost);
  }, [url]);

  if (!post) return <div>Loading...</div>;

  return (
    <article>
      <h1>{post.title}</h1>
      {post.featured_image && (
        <img src={post.featured_image.url} alt={post.title} />
      )}
      {post.author && (
        <div className="author">
          <span>By {post.author.name}</span>
        </div>
      )}
      <div
        dangerouslySetInnerHTML={{
          __html: DOMPurify.sanitize(post.content),
        }}
      />
    </article>
  );
}
```

### Navigation Menu

```typescript
import contentstack from "@contentstack/delivery-sdk";
import { useState, useEffect } from "react";
import Link from "next/link"; // or your routing library

const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION || "us",
});

// Fetch navigation items
async function getNavigation() {
  const result = await stack
    .contentType("navigation_item")
    .entry()
    .query()
    .ascending("order")
    .find();

  return result.entries;
}

// React component
function Navigation() {
  const [items, setItems] = useState([]);

  useEffect(() => {
    getNavigation().then(setItems);
  }, []);

  return (
    <nav>
      <ul>
        {items.map((item) => (
          <li key={item.uid}>
            <Link href={item.url}>{item.title}</Link>
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

## TypeScript Types

### Define Entry Types

```typescript
import { Entry } from "@contentstack/delivery-sdk";

// Base entry interface
interface BaseEntry extends Entry {
  uid: string;
  title: string;
  url?: string;
  created_at: string;
  updated_at: string;
}

// Blog post entry
interface BlogPost extends BaseEntry {
  title: string;
  url: string;
  excerpt: string;
  content: string; // richtext
  published_date: string;
  featured_image?: Asset;
  author?: Author;
  category?: Category;
  tags?: Tag[];
}

// Author entry
interface Author extends BaseEntry {
  name: string;
  bio: string;
  avatar?: Asset;
  email?: string;
}

// Category entry
interface Category extends BaseEntry {
  title: string;
  slug: string;
  description?: string;
}

// Asset type
interface Asset {
  uid: string;
  url: string;
  title: string;
  filename: string;
  content_type: string;
  file_size: number;
  dimension?: {
    width: number;
    height: number;
  };
}

// Modular block types
interface HeroBlock {
  _content_type_uid: "hero_block";
  title: string;
  description?: string;
  image?: Asset;
  cta_text?: string;
  cta_url?: string;
}

interface ContentBlock {
  _content_type_uid: "content_block";
  title: string;
  content: string; // richtext
}

// Page with modular blocks
interface Page extends BaseEntry {
  title: string;
  url: string;
  blocks?: Array<{
    block: HeroBlock | ContentBlock | CTABlock;
    _metadata: { uid: string };
  }>;
}
```

### Type-Safe Fetch Functions

```typescript
import contentstack, { QueryOperation } from "@contentstack/delivery-sdk";

const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION || "us",
});

async function getBlogPost(uid: string): Promise<BlogPost | null> {
  try {
    const entry = await stack
      .contentType("blog_post")
      .entry(uid)
      .includeReference(["author", "category", "tags"])
      .fetch();

    return entry as BlogPost;
  } catch (error) {
    console.error("Error fetching blog post:", error);
    return null;
  }
}

async function getBlogPosts(filters?: {
  category?: string;
  limit?: number;
}): Promise<BlogPost[]> {
  const query = stack.contentType("blog_post").entry().query();

  if (filters?.category) {
    query.where("category", QueryOperation.EQUALS, filters.category);
  }

  if (filters?.limit) {
    query.limit(filters.limit);
  }

  const result = await query.includeReference(["author"]).find();
  return result.entries as BlogPost[];
}
```

## Locale Handling

### Fetch Entry in Specific Locale

```typescript
import contentstack from "@contentstack/delivery-sdk";

const stack = contentstack.stack({
  apiKey: process.env.CONTENTSTACK_API_KEY!,
  deliveryToken: process.env.CONTENTSTACK_DELIVERY_TOKEN!,
  environment: process.env.CONTENTSTACK_ENVIRONMENT!,
  region: process.env.CONTENTSTACK_REGION || "us",
});

// Fetch entry in French locale
const entry = await stack
  .contentType("page")
  .entry("entry_uid")
  .language("fr-fr")
  .fetch();

// Fetch with locale fallback
async function getEntryWithFallback(
  uid: string,
  preferredLocale: string = "en-us"
) {
  try {
    const entry = await stack
      .contentType("page")
      .entry(uid)
      .language(preferredLocale)
      .fetch();
    return entry;
  } catch (error) {
    // Fallback to default locale
    return await stack.contentType("page").entry(uid).fetch();
  }
}
```

### Locale Switcher Component

```typescript
import { useRouter } from "next/router"; // or your routing library

const SUPPORTED_LOCALES = ["en-us", "fr-fr", "de-de"];

function LocaleSwitcher({ currentLocale }: { currentLocale: string }) {
  const router = useRouter();

  function switchLocale(locale: string) {
    router.push(`/${locale}${router.pathname}`);
  }

  return (
    <select
      value={currentLocale}
      onChange={(e) => switchLocale(e.target.value)}
    >
      {SUPPORTED_LOCALES.map((locale) => (
        <option key={locale} value={locale}>
          {locale.toUpperCase()}
        </option>
      ))}
    </select>
  );
}
```

## Next Steps

- Review [SDK Functionalities](02-sdk-functionalities.md) for more query patterns
- Learn about [Live Preview Guide](03-live-preview-guide.md) for real-time updates
