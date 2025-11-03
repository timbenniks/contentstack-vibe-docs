# Contentstack Live Preview: Complete Guide for AI Agents

This guide consolidates everything needed to implement Contentstack's **Live Preview** feature.
It is written for AI coding assistants and assumes you can generate and insert code directly into a project.
The instructions are self-contained and avoid cross-referencing other documents.

## 1. What is Live Preview?

Contentstack's **Live Preview** feature lets content managers see changes instantly before publishing. Whenever an editor opens an entry, the CMS generates a _Live Preview hash_ that identifies the preview session and links the entry editor with the website loaded in an iframe. The Live Preview Utils SDK running on the site uses `postMessage()` events to communicate with the entry editor, fetches the latest content and injects it into the preview panel without reloading.

**How it works:**
- Editor opens an entry in Contentstack CMS
- CMS generates a Live Preview hash and appends it to the preview URL
- Website loads in an iframe with the hash in query parameters
- Live Preview Utils SDK detects the hash and connects to the CMS
- SDK listens for content changes and updates the preview in real-time
- For CSR: Content updates instantly without page reload
- For SSR: Page refreshes to show updated content

**Benefits:**
- Editors see changes immediately without publishing
- Preview works across different device sizes and locales
- Enables real-time content editing experience
- Reduces publishing errors and improves content quality

## 2. Key Concepts and Terminology

| Concept                  | Description                                                                                                                                                                                                                                                                  |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Live Preview hash**      | A session-specific ID appended to preview URLs (e.g., `?live_preview=abc123`). It identifies the editor session and ensures the preview shows the latest unpublished changes.                                                                                                                                              |
| **Preview token**          | A special API token (generated from **Settings → Tokens → Delivery Tokens**) that allows the site to call preview endpoints. It's associated with a delivery token and should be used for Live Preview instead of a management token.                                                       |
| **Live Preview Utils SDK** | JavaScript library (`@contentstack/live-preview-utils`) that connects the CMS and the front-end. It listens for entry-change events, injects content, adds edit buttons and supports edit tags. Version 3 introduces modes (`preview` vs `builder`), configurable Edit/Start Editing buttons and improved configuration options. |
| **Live Edit tags**         | Special `data-cslp` attributes pointing to the path of a field (e.g., `content_type.entry.locale.field`). The SDK reads these tags to highlight fields and show Edit buttons. Tags can be added manually or generated automatically with `addEditableTags()`.                                |
| **CSR Mode**              | Client-Side Rendering mode - Content updates instantly without page refresh. Use `ssr: false` when initializing the SDK.                                                                                                                                    |
| **SSR Mode**              | Server-Side Rendering mode - Page refreshes when content changes. Use `ssr: true` when initializing the SDK.                                                                                                                                                                    |
| **Preview Service**       | New service that replaces the Content Management API (CMA) for preview requests. Uses `rest-preview.contentstack.com` or `graphql-preview.contentstack.com` endpoints.                                                                                                           |

## 3. Prerequisites and Setup

Before adding Live Preview support, ensure the following:

### 3.1 Enable Live Preview in Contentstack

1. Navigate to **Settings → Environments** in your Contentstack stack
2. Set the **base URL** for each locale (e.g., `https://your-site.com`)
3. Under **Live Preview**, tick **Enable Live Preview**
4. Select a default preview environment (usually `preview` or `development`)
5. Save settings

When enabled, an eye icon (👁️) appears in the entry editor sidebar.

### 3.2 Generate a Preview Token

1. Go to **Settings → Tokens → Delivery Tokens**
2. Select an existing delivery token or create a new one
3. Toggle **Create preview token**
4. Copy the preview token (you'll need this in your code)

**Important**: The preview token is associated with a delivery token. Use this preview token (not the management token) for Live Preview functionality.

### 3.3 Host Requirements

Your website must meet these requirements:

- **HTTPS**: Site must be served over HTTPS (required for iframe embedding)
- **Iframe-Compatible**: Site must allow embedding in iframes
  - Remove or configure `X-Frame-Options` headers
  - Ensure `Content-Security-Policy` allows iframe embedding if used
- **Publicly Accessible**: Site must be accessible from the internet (Contentstack needs to load it in an iframe)

**Hosting Options:**
- Contentstack's **Launch** service (recommended for easy setup)
- Any hosting provider (Vercel, Netlify, AWS, etc.)
- Local development with tunneling service (ngrok, Cloudflare Tunnel, etc.)

## 4. How to Set Up Live Preview

Below is a consolidated step-by-step outline. The exact code varies by framework, but the high-level process is consistent.

### 4.1 High-Level Setup Process

1. **Enable Live Preview in the stack** (see [Prerequisites](#3-prerequisites-and-setup))
2. **Generate a Preview token** (see [Prerequisites](#3-prerequisites-and-setup))
3. **Build your site with a Contentstack SDK** - Configure SDK with Live Preview support
4. **Host the site** on a secure domain (HTTPS, iframe-compatible)
5. **Implement Live Edit tags** (optional but recommended)
6. **Configure edit buttons** via SDK configuration

### 4.2 Framework-Specific Setup Patterns

The SDK will be configured differently for CSR or SSR:

- **GraphQL - CSR**: Initialize `@contentstack/live-preview-utils` with `ssr: false`, `enable: true`, stack details and client URL parameters. When the preview hash is present, change the GraphQL host from `graphql.contentstack.com` to `graphql-preview.contentstack.com` and add `live_preview`, `preview_token` and `include_applied_variants` headers. Use `onEntryChange()` inside `useEffect` to refetch data when an entry changes.

- **GraphQL - SSR**: Similar to CSR but set `ssr: true`. On the server, extract the hash from the query parameters and pass it to the GraphQL request headers; re-initialize the SDK for each request to avoid cross-session data.

- **Gatsby (uses GraphQL)**: Create a `live-preview.js` module that instantiates `ContentstackGatsby` with `preview_token`, environment and host; install and initialize the Live Preview Utils SDK; connect `onLiveEdit()` to fetch updated data and update component state. Add `__typename` and `uid` to GraphQL queries; use `addEditableTags()` to generate edit tags.

- **REST - CSR**: Configure the Delivery SDK with a `live_preview` object containing `preview_token`, `enable: true` and `host: 'rest-preview.contentstack.com'`. Initialize the Live Preview Utils SDK (`ssr: false`) and use `onEntryChange()` to refetch data.

- **REST - SSR**: Same as CSR but set `ssr: true`. On each server request, create a fresh stack instance and call `Stack.livePreviewQuery(req.query)` to inject the hash into API calls. Always create a new SDK instance per request to avoid showing wrong preview data across users.

### 4.3 Common Implementation Steps

The following steps apply to all frameworks and rendering modes:

1. **Import the necessary SDKs** - Use the Contentstack SDK (`@contentstack/delivery-sdk`) and install `@contentstack/live-preview-utils` via npm.

   **CRITICAL**: ALWAYS USE THE TYPESCRIPT DELIVERY SDK (`@contentstack/delivery-sdk`) WHEN YOU QUERY CONTENTSTACK!

2. **Configure the SDK** - Always include a `live_preview` configuration when creating the SDK instance. Set `enable: true`, pass your `preview_token` and specify the correct `host` (see [Region Hosts](#10-region-hosts)).

3. **Initialize the Live Preview Utils SDK** - Call `ContentstackLivePreview.init()` once on the client side. Specify whether you are using SSR (`ssr: true`) or CSR (`ssr: false`), and provide:
   - `stackDetails`: object with `apiKey`, `environment` and optional `branch`.
   - `stackSdk`: reference to your Contentstack SDK instance (for Gatsby use `ContentstackGatsby.stackSdk`).
   - `editButton`: optional object to enable/disable the Edit button and set its position.
   - `clientUrlParams`: optional object with `protocol`, `host` and `port` of the Contentstack app (defaults to `https://app.contentstack.com`).

4. **React to content changes** - Register event handlers to fetch fresh data when the editor updates an entry. Use `onEntryChange()` in CSR contexts and `onLiveEdit()` when you only want to update while editing.

5. **Add Live Edit tags** - Either manually add `data-cslp="contentType.entry.locale.field"` to your HTML or call `addEditableTags(entry, content_type_uid, tagsAsObject, locale)` after fetching data to append tags automatically. When using JSX, pass tags as an object (`{ 'data-cslp': 'path' }`).

## 5. Choosing CSR vs SSR

- **Client-side rendering (CSR)**: The browser fetches content and renders pages dynamically. When a Live Preview event arrives, you can update the component state without reloading the page. Use `ssr: false` when initialising the Live Preview SDK.
- **Server-side rendering (SSR)**: Pages are rendered on the server (e.g. with Express, Next.js or Nuxt). When an entry changes, the preview panel refreshes the entire page to show the updated HTML. Use `ssr: true`, and ensure you create a **new** Contentstack SDK instance for each request (to avoid cross-session state). Extract the Live Preview hash from the query string and pass it into your requests.

## 6. REST Implementation

### 6.1 REST - Client-Side Rendering

1. **Install dependencies**

   ```bash
   npm install @contentstack/delivery-sdk @contentstack/live-preview-utils
   ```

2. **Create a Stack instance with Live Preview**

   ```js
   // ALWAYS USE THE TYPESCRIPT DELIVERY SDK (import contentstack from "@contentstack/delivery-sdk")

   import contentstack from "@contentstack/delivery-sdk";
   import ContentstackLivePreview from "@contentstack/live-preview-utils";

   const stack = contentstack.stack({
     apiKey: "YOUR_API_KEY",
     deliveryToken: "YOUR_DELIVERY_TOKEN",
     environment: "YOUR_ENV",
     region: "YOUR_REGION",
     live_preview: {
       enable: true,
       preview_token: "YOUR_PREVIEW_TOKEN",
       host: "rest-preview.contentstack.com",
     },
   });
   ```

3. **Initialise Live Preview Utils SDK**

   ```js
   ContentstackLivePreview.init({
    enable: true,
    ssr: false,
    mode: "preview", // or "builder" for Visual Builder
    stackSdk: stack.config as IStackSdk,
    stackDetails: {
      apiKey: "YOUR_API_KEY",
      environment: "YOUR_ENV",
    },
    editButton: { enable: true },
   });
   ```

4. **Fetch data and handle updates**

   ```js
   async function getEntry() {
     const result = await stack
       .contentType("content_type_uid")
       .entry("entry_uid")
       .fetch();
     return result;
   }

   function initLivePreview() {
     ContentstackLivePreview.onEntryChange(async () => {
       const updated = await getEntry();
       // update your UI with updated data
     });
   }
   ```

### 6.2 REST - Server-Side Rendering

1. **Create a new Stack per request** with `live_preview` configured.
2. **Extract the Live Preview hash** from the request's query parameters and call `Stack.livePreviewQuery(req.query)` before fetching data.
3. **Example Express route**

   ```js
   import contentstack from "@contentstack/delivery-sdk";
   import ContentstackLivePreview from "@contentstack/live-preview-utils";

   app.get("*", async (req, res) => {
     const stack = contentstack.stack({
       apiKey: "YOUR_API_KEY",
       deliveryToken: "YOUR_DELIVERY_TOKEN",
       environment: "YOUR_ENV",
       region: "YOUR_REGION",
       live_preview: {
         enable: true,
         preview_token: "YOUR_PREVIEW_TOKEN",
         host: "graphql-preview.contentstack.com",
       },
     });

     ContentstackLivePreview.init({
      enable: true,
      ssr: true,
      mode: "preview", // or "builder" for Visual Builder
      stackSdk: stack.config as IStackSdk,
      stackDetails: {
        apiKey: "YOUR_API_KEY",
        environment: "YOUR_ENV",
      },
      editButton: { enable: true },
    });

     // Attach the preview hash from query params
     stack.livePreviewQuery(req.query);
     const entry = await stack
       .contentType("content_type_uid")
       .entry("entry_uid")
       .fetch();

     // Render your template with entry data
   });
   ```

## 7. GraphQL Implementation

### 7.1 GraphQL - Client-Side Rendering

1. **Install dependencies**

```bash
npm install @contentstack/delivery-sdk @contentstack/live-preview-utils
npm install graphql-request  # or your preferred GraphQL client
```

2. **Create a Contentstack stack instance**

   ```js
   import contentstack from "@contentstack/delivery-sdk";
   import ContentstackLivePreview from "@contentstack/live-preview-utils";

   // Replace placeholders with your values (please use .env file, do NOT hardocde these)
   const stack = contentstack.stack({
     apiKey: "YOUR_API_KEY",
     deliveryToken: "YOUR_DELIVERY_TOKEN",
     environment: "YOUR_ENV",
     region: "YOUR_REGION",
     live_preview: {
       enable: true,
       preview_token: "YOUR_PREVIEW_TOKEN",
       host: "graphql-preview.contentstack.com",
     },
   });
   ```

3. **Initialise Live Preview Utils SDK**

   ```js
   ContentstackLivePreview.init({
     enable: true,
     ssr: false,
     mode: "preview", // or "builder" for Visual Builder
     stackSdk: stack.config as IStackSdk,
     stackDetails: {
       apiKey: "YOUR_API_KEY",
       environment: "YOUR_ENV",
     },
     editButton: { enable: true },
   });
   ```

4. **GraphQL request helper with preview detection**

   ```js
   import { request } from "graphql-request";

   const GRAPHQL_HOST = "graphql.contentstack.com";
   const PREVIEW_HOST = "graphql-preview.contentstack.com";

   async function fetchGraphQL(query, variables = {}) {
     const hash = ContentstackLivePreview.hash;
     const isPreview = Boolean(hash);
     const host = isPreview ? PREVIEW_HOST : GRAPHQL_HOST;

     const headers = {
       api_key: "YOUR_API_KEY",
       // Always send your delivery token; the SDK will override it when preview is active
       access_token: "YOUR_DELIVERY_TOKEN",
     };
     if (isPreview) {
       headers.live_preview = hash;
       headers.preview_token = "YOUR_PREVIEW_TOKEN";
       headers.include_applied_variants = "true";
     }
     return request(`https://${host}/graphql`, query, variables, headers);
   }
   ```

5. **React component example**

   ```jsx
   import React, { useState, useEffect } from "react";

   function Home() {
     const [entry, setEntry] = useState(null);
     const query = `
       query GetPage($url: String!) {
         allContentstackPage(where: { url: $url }) {
           nodes { uid title url }
         }
       }
     `;

     async function fetchData() {
       const data = await fetchGraphQL(query, { url: "/" });
       setEntry(data.allContentstackPage.nodes[0]);
     }

     useEffect(() => {
       fetchData();
       ContentstackLivePreview.onEntryChange(fetchData);
     }, []);

     if (!entry) return null;
     return <h1>{entry.title}</h1>;
   }
   export default Home;
   ```

### 7.2 GraphQL - Server-Side Rendering

1. **Create a new Stack per request** and initialise the Live Preview Utils SDK with `ssr: true`.
2. **Extract the Live Preview hash** from the request's query parameters (e.g. `req.query.live_preview`) and include it in your GraphQL request headers. Only switch the host to `graphql-preview.contentstack.com` when a hash is present.
3. **Example Express route**

   ```js
   app.get("*", async (req, res) => {
     const hash = req.query.live_preview;
     const isPreview = Boolean(hash);
     const host = isPreview
       ? "graphql-preview.contentstack.com"
       : "graphql.contentstack.com";

     const stack = contentstack.stack({
       apiKey: "YOUR_API_KEY",
       deliveryToken: "YOUR_DELIVERY_TOKEN",
       environment: "YOUR_ENV",
       region: "YOUR_REGION",
       live_preview: {
         enable: true,
         preview_token: "YOUR_PREVIEW_TOKEN",
         host: "graphql-preview.contentstack.com",
       },
     });

    // this is always rendered in the client side.
    ContentstackLivePreview.init({
      enable: true,
      ssr: true,
      mode: "preview", // or "builder" for Visual Builder
      stackSdk: stack.config as IStackSdk,
      stackDetails: {
        apiKey: "YOUR_API_KEY",
        environment: "YOUR_ENV",
      },
      editButton: { enable: true },
    });

     // Build GraphQL request as shown in CSR example, passing hash and preview token when available
     // Render your template with fetched data
   });
   ```

## 8. Gatsby Implementation

Gatsby's `gatsby-source-contentstack` plugin exposes a `ContentstackGatsby` class that simplifies Live Preview.

1. **Install dependencies**

   ```bash
   npm install gatsby-source-contentstack @contentstack/live-preview-utils
   ```

2. **Create a Live Preview helper** (e.g. `src/live-preview.js`)

   ```js
   import { ContentstackGatsby } from "gatsby-source-contentstack/live-preview";

   export const cs = new ContentstackGatsby({
     api_key: process.env.GATSBY_CONTENTSTACK_API_KEY,
     environment: process.env.GATSBY_CONTENTSTACK_ENVIRONMENT,
     live_preview: {
       enable: true,
       preview_token: process.env.GATSBY_CONTENTSTACK_PREVIEW_TOKEN,
       host: "rest-preview.contentstack.com",
     },
   });
   ```

3. **Initialise the Live Preview Utils SDK** in a component that loads once (e.g. in `gatsby-browser.js` or a custom component). Pass `stackSdk: cs.stackSdk` and `ssr: false`.

   ```js
   import ContentstackLivePreview from "@contentstack/live-preview-utils";
   import { cs } from "./src/live-preview";

   export const onClientEntry = () => {
     ContentstackLivePreview.init({
       enable: true,
       ssr: false,
       stackSdk: cs.stackSdk,
       stackDetails: {
         apiKey: process.env.GATSBY_CONTENTSTACK_API_KEY,
         environment: process.env.GATSBY_CONTENTSTACK_ENVIRONMENT,
       },
       editButton: { enable: true },
     });
   };
   ```

4. **Use `onLiveEdit()` to update state**. In your page components, call `cs.get()` to fetch updated data when a Live Edit event occurs.

   ```jsx
   import React, { useEffect, useState } from "react";
   import ContentstackLivePreview from "@contentstack/live-preview-utils";
   import { cs } from "../live-preview";

   export const pageQuery = graphql`
     query HomeQuery {
       allContentstackPage {
         nodes {
           uid
           title
           url
         }
       }
     }
   `;

   const HomePage = (props) => {
     const [data, setData] = useState(props.data.allContentstackPage.nodes[0]);
     const fetchUpdated = async () => {
       const updated = await cs.get(props.data.allContentstackPage.nodes[0]);
       setData(updated);
     };
     useEffect(() => {
       ContentstackLivePreview.onLiveEdit(fetchUpdated);
     }, []);
     return <h1>{data.title}</h1>;
   };
   export default HomePage;
   ```

5. **Generate Live Edit tags** using `addEditableTags()` from `@contentstack/utils` after fetching your entry data:

   ```js
   import { addEditableTags } from "@contentstack/utils";
   // After fetching entry
   addEditableTags(entry, "content_type_uid", false, "en-us");
   ```

## 9. Live Edit Tags

### 9.1 Manual tags

Add `data-cslp="{content_type_uid}.{entry_uid}.{locale}.{field_uid}"` to HTML elements representing your fields. For nested fields (e.g. modular blocks), use `{modular_block_field_UID}.{block_UID}.{field_UID}`.

```html
<h1 data-cslp="home.blt123456789.en-us.title">Welcome!</h1>
```

### 9.2 Automatic tags via `addEditableTags()`

After retrieving an entry via the Contentstack SDK, call `addEditableTags(entry, content_type_uid, tagsAsObject, locale)`. When `tagsAsObject` is `false`, tags are added as strings (good for templating languages). When `true`, tags are returned as objects (good for JSX). Example:

```js
import contentstack from "@contentstack/delivery-sdk";
const entry = await stack.contentType("blog").entry("entry_uid").fetch();
contentstack.Utils.addEditableTags(entry, "blog", true, "en-us");

// Now entry.$ holds objects to spread into JSX
```

In React:

```jsx
<h1 {...entry.$.title}>{entry.title}</h1>
```

## 10. Region Hosts

| Data centre                     | GraphQL preview host                        | REST preview host                        | App host (clientUrlParams)      |
| ------------------------------- | ------------------------------------------- | ---------------------------------------- | ------------------------------- |
| North America (default: AWS NA) | `graphql-preview.contentstack.com`          | `rest-preview.contentstack.com`          | `app.contentstack.com`          |
| AWS EU                          | `eu-graphql-preview.contentstack.com`       | `eu-rest-preview.contentstack.com`       | `eu-app.contentstack.com`       |
| Azure NA                        | `azure-na-graphql-preview.contentstack.com` | `azure-na-rest-preview.contentstack.com` | `azure-na-app.contentstack.com` |
| Azure EU                        | `azure-eu-graphql-preview.contentstack.com` | `azure-eu-rest-preview.contentstack.com` | `azure-eu-app.contentstack.com` |
| GCP NA                          | `gcp-na-graphql-preview.contentstack.com`   | `gcp-na-rest-preview.contentstack.com`   | `gcp-na-app.contentstack.com`   |
| GCP EU                          | `gcp-eu-graphql-preview.contentstack.com`   | `gcp-eu-rest-preview.contentstack.com`   | `gcp-eu-app.contentstack.com`   |

Select the correct host based on your stack's region. Use the same host for both the `live_preview.host` option and the `clientUrlParams.host` when initialising the SDK.

## 11. Additional Features

### Preview Service

Contentstack has introduced a new Preview Service that replaces the Content Management API (CMA) for preview requests. This provides improved performance and better reliability.

**Migrating to Preview Service:**

If you built Live Preview using the old Content Management API (CMA), switch to the Preview Service:

1. **Change the API base URL** in your code from `api.contentstack.io` to `rest-preview.contentstack.com` (for REST) or from `graphql.contentstack.com` to `graphql-preview.contentstack.com` (for GraphQL).
2. **Replace the management token with your preview token**.
3. **When a preview hash is present**, include it as the `live_preview` header and set `preview_token` in the headers. Only switch the host to the preview host when the hash exists; otherwise use the normal CDN host.

### SDK Version 3 Features

Version 3 of the Live Preview Utils SDK introduces:

- **Mode Property**: `preview` for Live Preview and `builder` for Visual Builder
- **Start Editing Button**: Quickly switch to edit mode in Visual Builder
- **Improved Configuration**: More flexible configuration options
- **Clean Production Tags**: Use `cleanCslpOnProduction` to remove all `data-cslp` attributes in production

### Event-Based Communication

The SDK relies on events to keep the preview in sync:

- **`init-ack`**: Sent when the session starts
- **`client-data-send`**: Triggers content update
- **`entry-change`**: Fired when entry content changes

These events enable real-time communication between the CMS and your website without requiring page refreshes (in CSR mode).

## 12. Framework-Specific Patterns

The patterns below apply to any framework. The key difference is CSR (real-time updates) vs SSR (page refresh).

### Next.js App Directory (CSR)

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

export function initLivePreview() {
  ContentstackLivePreview.init({
    ssr: false, // CSR mode
    enable: true,
    stackSdk: stack.config as IStackSdk,
    stackDetails: {
      apiKey: process.env.NEXT_PUBLIC_CONTENTSTACK_API_KEY!,
      environment: process.env.NEXT_PUBLIC_CONTENTSTACK_ENVIRONMENT!,
    },
    editButton: { enable: true },
  });
}
```

```typescript
// app/page.tsx
"use client";
import { useEffect, useState } from "react";
import contentstack from "@contentstack/delivery-sdk";
import ContentstackLivePreview from "@contentstack/live-preview-utils";
import { stack, initLivePreview } from "@/lib/contentstack";

export default function Page() {
  const [entry, setEntry] = useState(null);

  useEffect(() => {
    initLivePreview();

    const fetchData = async () => {
      const result = await stack.contentType("page").entry("entry_uid").fetch();
      contentstack.Utils.addEditableTags(result, "page", true);
      setEntry(result);
    };

    ContentstackLivePreview.onEntryChange(fetchData);
    fetchData();
  }, []);

  return entry && <h1 {...(entry?.$ && entry?.$.title)}>{entry.title}</h1>;
}
```

### Next.js App Directory (SSR)

```typescript
// app/page.tsx (Server Component)
import contentstack from "@contentstack/delivery-sdk";
import { stack } from "@/lib/contentstack";

export default async function Page({ searchParams }) {
  const { live_preview, entry_uid, content_type_uid } = await searchParams;

  if (live_preview) {
    stack.livePreviewQuery({
      live_preview: live_preview as string,
      contentTypeUid: content_type_uid as string,
      entryUid: entry_uid as string,
    });
  }

  const entry = await stack.contentType("page").entry("entry_uid").fetch();
  contentstack.Utils.addEditableTags(entry, "page", true);

  return <h1 {...(entry?.$ && entry?.$.title)}>{entry.title}</h1>;
}
```

```typescript
// app/layout.tsx - Initialize SDK on client
"use client";
import { useEffect } from "react";
import { initLivePreview } from "@/lib/contentstack";

export default function Layout({ children }) {
  useEffect(() => {
    initLivePreview(); // Use ssr: true for SSR mode
  }, []);

  return <>{children}</>;
}
```

### Nuxt 4 (CSR)

```typescript
// plugins/contentstack.ts
import contentstack from "@contentstack/delivery-sdk";
import ContentstackLivePreview, {
  type IStackSdk,
} from "@contentstack/live-preview-utils";

export default defineNuxtPlugin((nuxtApp) => {
  const stack = contentstack.stack({
    apiKey: process.env.NUXT_CONTENTSTACK_API_KEY!,
    deliveryToken: process.env.NUXT_CONTENTSTACK_DELIVERY_TOKEN!,
    environment: process.env.NUXT_CONTENTSTACK_ENVIRONMENT!,
    region: "us",
    live_preview: {
      enable: true,
      preview_token: process.env.NUXT_CONTENTSTACK_PREVIEW_TOKEN,
      host: "rest-preview.contentstack.com",
    },
  });

  if (import.meta.client) {
    ContentstackLivePreview.init({
      ssr: false, // CSR mode
      enable: true,
      stackSdk: stack.config as IStackSdk,
      stackDetails: {
        apiKey: process.env.NUXT_CONTENTSTACK_API_KEY!,
        environment: process.env.NUXT_CONTENTSTACK_ENVIRONMENT!,
      },
    });
  }

  return { provide: { stack } };
});
```

```vue
<!-- app.vue -->
<script setup>
import contentstack from "@contentstack/delivery-sdk";

const { data: page, refresh } = await useAsyncData("page", async () => {
  const { $stack } = useNuxtApp();
  const route = useRoute();

  if (route.query.live_preview) {
    $stack.livePreviewQuery(route.query);
  }

  const entry = await $stack.contentType("page").entry("entry_uid").fetch();
  contentstack.Utils.addEditableTags(entry, "page", true);
  return entry;
});

onMounted(() => {
  const { $ContentstackLivePreview } = useNuxtApp();
  $ContentstackLivePreview.onEntryChange(refresh);
});
</script>

<template>
  <h1 v-if="page?.title" v-bind="page?.$ && page?.$.title">
    {{ page.title }}
  </h1>
</template>
```

### Nuxt 4 (SSR)

For SSR, use the same plugin but set `ssr: true` and remove the `onMounted` refresh handler. The page will automatically refresh when content changes.

## 13. Additional Notes

- Always initialise the Live Preview Utils SDK on the **client side** (browser) for CSR and on **each request** for SSR.
- Ensure your preview site uses **HTTPS** and allows iframe embedding.
- The **Edit button** can be customised via the `editButton` object in the SDK configuration. You can disable it entirely or choose positions like `top`, `top-right`, `bottom-left`, etc.
- Avoid caching preview responses—fetch fresh data for each update.
- For other frameworks (Vue, Angular, Svelte, etc.), follow the same patterns: configure SDK with `live_preview`, initialise the Live Preview Utils SDK, detect the hash, and refetch data on change (CSR) or let page refresh (SSR).
