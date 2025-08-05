# Contentstack Live Preview: Implementation Guide for AI Agents

This guide consolidates everything needed to implement Contentstack's **Live Preview** feature.
It is written for AI coding assistants and assumes you can generate and insert code directly into a project.
The instructions are self-contained and avoid cross-referencing other documents.

## 1. Overview

Contentstack's Live Preview lets editors view changes to content immediately, across devices and locales, without publishing.
When an entry is opened in the CMS, Contentstack appends a **Live Preview hash** to the preview URL and loads your website in an iframe.
A JavaScript library called the **Live Preview Utils SDK** runs in your website and communicates with the CMS using the `postMessage` API.
It listens for content-change events and updates the page accordingly. Live Preview supports both **client-side rendering (CSR)** and **server-side rendering (SSR)** frameworks.

## 2. Key Concepts

- **Preview token** - A special API token generated from **Settings → Tokens → Delivery Tokens**. It is associated with a delivery token and is used to fetch unpublished content. Do not use your management token for Live Preview.
- **Live Preview hash** - A session-specific identifier that Contentstack adds to preview URLs. When this hash is present, your application must fetch data from the preview endpoints and include the hash and preview token in API headers. When absent, use the normal delivery endpoints.
- **Live Preview Utils SDK** - A JavaScript package (`@contentstack/live-preview-utils`) that manages the connection between the CMS and your site. It works with Contentstack's GraphQL, REST and Gatsby SDKs. You initialise it once with configuration options and then register event handlers.
- **Live Edit tags** - HTML attributes (`data-cslp`) that map elements in your site to fields in the entry. When you click the edit button in the preview panel, the CMS uses these tags to focus the correct field. Tags can be generated automatically via `addEditableTags()` from the Contentstack Utils SDK.

## 3. Prerequisites

Before adding Live Preview support, ensure the following:

1. You have a Contentstack **stack** with at least one **environment**.
2. Live Preview is **enabled** on the stack:
   1. Navigate to **Settings → Environments** and enter the **base URL** for each locale.
   2. Under **Live Preview**, tick **Enable Live Preview** and select a default preview environment.
3. You have a **preview token** (generated from **Settings → Tokens → Delivery Tokens**). Copy this token; it will be used in code.
4. Your website is served over **HTTPS** and can be embedded in an iframe. Disable X-Frame-Options headers or set them appropriately.

## 4. Common Setup Steps

The following steps apply to all frameworks and rendering modes:

1. **Import the necessary SDKs** - use the Contentstack SDK (`@contentstack/delivery-sdk`) and install `@contentstack/live-preview-utils` via npm or a script tag.

- THIS IS VERY IMPORTANT: ALWAYS USE THE TYPESCRIPT DELIVERY SDK (`@contentstack/delivery-sdk`) WHEN YOU QUERY CONTENTSTACK!

2. **Configure the SDK** - always include a `live_preview` configuration when creating the SDK instance. Set `enable: true`, pass your `preview_token` and specify the correct `host` (see [Region Hosts](#region-hosts)).
3. **Initialise the Live Preview Utils SDK**. Call `ContentstackLivePreview.init()` once on the client side. Specify whether you are using SSR (`ssr: true`) or CSR (`ssr: false`), and provide:
   - `stackDetails`: object with `apiKey`, `environment` and optional `branch`.
   - `stackSdk`: reference to your Contentstack SDK instance (for Gatsby use `ContentstackGatsby.stackSdk`).
   - `editButton`: optional object to enable/disable the Edit button and set its position.
   - `clientUrlParams`: optional object with `protocol`, `host` and `port` of the Contentstack app (defaults to `https://app.contentstack.com`).
4. **React to content changes** - register event handlers to fetch fresh data when the editor updates an entry. Use `onEntryChange()` in CSR contexts and `onLiveEdit()` when you only want to update while editing.
5. **Add Live Edit tags** - either manually add `data-cslp="contentType.entry.locale.field"` to your HTML or call `addEditableTags(entry, content_type_uid, tagsAsObject, locale)` after fetching data to append tags automatically. When using JSX, pass tags as an object (`{ 'data-cslp': 'path' }`).

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

## 11. Migrating to the Preview Service

If you built Live Preview using the old Content Management API (CMA), switch to the Preview Service for improved performance:

1. **Change the API base URL** in your code from `api.contentstack.io` to `rest-preview.contentstack.com` (for REST) or from `graphql.contentstack.com` to `graphql-preview.contentstack.com` (for GraphQL).
2. **Replace the management token with your preview token**.
3. **When a preview hash is present**, include it as the `live_preview` header and set `preview_token` in the headers. Only switch the host to the preview host when the hash exists; otherwise use the normal CDN host.

## 12. Additional Notes

- Always initialise the Live Preview Utils SDK on the **client side** (browser) for CSR and on **each request** for SSR.
- Ensure your preview site uses **HTTPS** and allows iframe embedding.
- The **Edit button** can be customised via the `editButton` object in the SDK configuration. You can disable it entirely or choose positions like `top`, `top-right`, `bottom-left`, etc.
- Avoid caching preview responses—fetch fresh data for each update.
- For frameworks not covered here (Vue, Angular, Nuxt, Next, etc.), follow the same patterns: generate the preview token, configure your SDK with `live_preview`, initialise the Live Preview Utils SDK, detect the hash, and refetch data on change.
