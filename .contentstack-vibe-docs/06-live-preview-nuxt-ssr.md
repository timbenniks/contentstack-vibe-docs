# Contentstack Live Preview Integration Guide for Nuxt 4 SSR

This guide provides focused instructions for AI coding assistants to integrate Contentstack Live Preview into an existing Nuxt 4 application using Server-Side Rendering (SSR) mode.

## Prerequisites

- Existing Nuxt 4 application with latest configuration
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

Create a `.env` file in your Nuxt root directory:

```env
NUXT_CONTENTSTACK_API_KEY=your_api_key
NUXT_CONTENTSTACK_DELIVERY_TOKEN=your_delivery_token
NUXT_CONTENTSTACK_PREVIEW_TOKEN=your_preview_token
NUXT_CONTENTSTACK_ENVIRONMENT=your_environment
NUXT_CONTENTSTACK_REGION=eu
NUXT_CONTENTSTACK_PREVIEW=true
```

## Nuxt Configuration

Update your `nuxt.config.ts` file to support Contentstack Live Preview:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  compatibilityDate: "2024-04-03",
  devtools: { enabled: true },

  future: {
    compatibilityVersion: 4,
  },

  modules: ["@nuxtjs/tailwindcss"], // Add any other modules you need

  // CRITICAL: Transpile the endpoints package for proper SSR support
  build: {
    transpile: ["@timbenniks/contentstack-endpoints"],
  },

  // Runtime configuration for environment variables
  runtimeConfig: {
    public: {
      apiKey: process.env.NUXT_CONTENTSTACK_API_KEY,
      deliveryToken: process.env.NUXT_CONTENTSTACK_DELIVERY_TOKEN,
      previewToken: process.env.NUXT_CONTENTSTACK_PREVIEW_TOKEN,
      environment: process.env.NUXT_CONTENTSTACK_ENVIRONMENT,
      preview: process.env.NUXT_CONTENTSTACK_PREVIEW === "true",
      region: process.env.NUXT_CONTENTSTACK_REGION,
    },
  },
});
```

## Core Implementation

### 1. Contentstack Plugin Configuration

Create the main Contentstack plugin with region-specific endpoints:

```typescript
// plugins/contentstack.ts
import contentstack, { Region } from "@contentstack/delivery-sdk";
import ContentstackLivePreview, {
  type IStackSdk,
} from "@contentstack/live-preview-utils";

// Option 1: Using @timbenniks/contentstack-endpoints (Recommended)
import {
  getContentstackEndpoints,
  getRegionForString,
} from "@timbenniks/contentstack-endpoints";

export default defineNuxtPlugin((nuxtApp) => {
  const { apiKey, deliveryToken, previewToken, region, environment, preview } =
    nuxtApp.$config.public;

  // RECOMMENDED: Use the endpoints package for automatic endpoint resolution
  const regionEnum: Region = getRegionForString(region);
  const endpoints = getContentstackEndpoints(regionEnum, true); // true = omit https://

  // Alternative manual endpoint configuration (if you prefer not to use the package)
  const getManualEndpoints = (region: string) => {
    const endpointMap = {
      us: {
        preview: "rest-preview.contentstack.com",
        application: "app.contentstack.com",
      },
      eu: {
        preview: "eu-rest-preview.contentstack.com",
        application: "eu-app.contentstack.com",
      },
    };
    return endpointMap[region as keyof typeof endpointMap] || endpointMap.us;
  };

  // Initialize Contentstack SDK with Live Preview configuration
  const stack = contentstack.stack({
    apiKey,
    deliveryToken,
    environment,
    region: regionEnum,
    live_preview: {
      enable: preview ? true : false,
      preview_token: previewToken,
      host: endpoints.preview, // or getManualEndpoints(region).preview
    },
  });

  // CRITICAL: Initialize Live Preview only on client-side for SSR mode
  if (preview && import.meta.client) {
    ContentstackLivePreview.init({
      ssr: true, // CRITICAL: SSR mode - enables page refresh behavior
      mode: "builder", // Supports both Live Preview and Visual Builder
      enable: preview ? true : false,
      stackSdk: stack.config as IStackSdk,
      stackDetails: {
        apiKey: apiKey,
        environment: environment,
      },
      clientUrlParams: {
        host: endpoints.application, // or getManualEndpoints(region).application
      },
      editButton: {
        enable: true,
        exclude: ["outsideLivePreviewPortal"],
      },
    });
  }

  return {
    provide: {
      stack,
      preview,
      ContentstackLivePreview,
    },
  };
});
```

### 2. Data Fetching Composable

The SSR composable is simpler as it doesn't need client-side refresh handling:

```typescript
// composables/useGetPage.ts
import contentstack, {
  QueryOperation,
  type LivePreviewQuery,
} from "@contentstack/delivery-sdk";

/**
 * Retrieve Contentstack Page Data with Live Preview Support (SSR Mode)
 *
 * This composable fetches page data from Contentstack CMS using Nuxt's composables.
 * It supports both regular content delivery and live preview functionality with
 * page refresh behavior for content updates.
 *
 * Features:
 * - Fetches page content based on URL parameter
 * - Supports Contentstack Live Preview mode with SSR
 * - Handles data fetching with Nuxt's useAsyncData
 * - Implements type-safe Contentstack queries
 * - Adds editable tags for content management
 * - Works with query parameter-based Live Preview
 *
 * Usage:
 * const { data, status } = await useGetPage("/about");
 */
export const useGetPage = async (url: string) => {
  const { data, status } = await useAsyncData(`page-${url}`, async () => {
    const { $preview, $stack } = useNuxtApp();
    const route = useRoute();
    const qs = { ...toRaw(route.query) };

    // CRITICAL: Apply live preview query parameters for SSR
    if ($preview && qs?.live_preview) {
      $stack.livePreviewQuery(qs as unknown as LivePreviewQuery);
    }

    const result = await $stack
      .contentType("page")
      .entry()
      .query()
      .where("url", QueryOperation.EQUALS, url)
      .find();

    if (result.entries) {
      const entry = result.entries[0];

      // Add editable tags for Live Preview
      if ($preview) {
        contentstack.Utils.addEditableTags(entry, "page", true);
      }

      return entry;
    }
  });

  return { data, status };
};
```

## SSR App Component Implementation

```vue
<!-- app.vue (SSR version) -->
<script lang="ts" setup>
import VB_EmptyBlockParentClass from "@contentstack/live-preview-utils";

// Fetch page data - no client-side refresh needed in SSR mode
// Page will automatically refresh when content changes
const { data: page } = await useGetPage("/");
</script>

<template>
  <main class="max-w-screen-md mx-auto">
    <section class="p-4">
      <!-- Page content with live preview bindings -->
      <h1
        v-if="page?.title"
        class="text-4xl font-bold mb-4"
        v-bind="page?.$ && page?.$.title"
      >
        {{ page?.title }}
      </h1>

      <p
        v-if="page?.description"
        class="mb-4"
        v-bind="page?.$ && page?.$.description"
      >
        {{ page?.description }}
      </p>

      <img
        v-if="page?.image"
        class="mb-4"
        width="768"
        height="414"
        :src="page?.image.url"
        :alt="page?.image.title"
        v-bind="page?.image?.$ && page?.image?.$.url"
      />

      <div
        v-if="page?.rich_text"
        v-bind="page?.$ && page?.$.rich_text"
        v-html="page?.rich_text"
      />

      <!-- Dynamic blocks with SSR live preview support -->
      <div
        :class="[
          'space-y-8 max-w-full mt-4',
          !page?.blocks || page.blocks.length === 0
            ? VB_EmptyBlockParentClass
            : '',
        ]"
        v-bind="page?.$ && page?.$.blocks"
      >
        <div
          v-for="(item, index) in page?.blocks"
          :key="item.block._metadata.uid"
          v-bind="page?.$ && page?.$[`blocks__${index}`]"
          :class="[
            'flex flex-col md:flex-row items-center space-y-4 md:space-y-0 bg-white',
            item.block.layout === 'image_left'
              ? 'md:flex-row'
              : 'md:flex-row-reverse',
          ]"
        >
          <div class="w-full md:w-1/2">
            <img
              v-if="item.block.image"
              :src="item.block.image.url"
              :alt="item.block.image.title"
              width="200"
              height="112"
              class="w-full"
              v-bind="item.block.$ && item.block.$.image"
            />
          </div>
          <div class="w-full md:w-1/2 p-4">
            <h2
              v-if="item.block.title"
              class="text-2xl font-bold"
              v-bind="item.block.$ && item.block.$.title"
            >
              {{ item.block.title }}
            </h2>
            <div
              v-if="item.block.copy"
              v-html="item.block.copy"
              class="prose"
              v-bind="item.block.$ && item.block.$.copy"
            />
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

<style>
body {
  background: #e9e9e9;
}

a {
  text-decoration: underline;
}
</style>
```

## SSR Live Preview Integration Patterns

### Pattern 1: Basic Page Integration

```vue
<script lang="ts" setup>
// SSR - No client-side refresh handling needed
const { data: page } = await useGetPage("/");
// Page will automatically refresh when content changes in Live Preview
</script>

<template>
  <div>
    <!-- CRITICAL: Always bind editable tags using v-bind -->
    <h1 v-if="page?.title" v-bind="page?.$ && page?.$.title">
      {{ page?.title }}
    </h1>
  </div>
</template>
```

### Pattern 2: Dynamic Route Integration

```vue
<!-- pages/[...slug].vue -->
<script lang="ts" setup>
const route = useRoute();
const slug = Array.isArray(route.params.slug)
  ? `/${route.params.slug.join("/")}`
  : `/${route.params.slug}`;

// SSR - Query parameters are automatically handled by the composable
const { data: page } = await useGetPage(slug);
</script>

<template>
  <div>
    <h1 v-if="page?.title" v-bind="page?.$ && page?.$.title">
      {{ page?.title }}
    </h1>
    <!-- Rest of your page content -->
  </div>
</template>
```

### Pattern 3: Collection/List Page Integration

```vue
<script lang="ts" setup>
// For multiple entries with SSR live preview
const { data: entries } = await useAsyncData("blog-posts", async () => {
  const { $preview, $stack } = useNuxtApp();
  const route = useRoute();
  const qs = { ...toRaw(route.query) };

  // CRITICAL: Apply live preview query parameters
  if ($preview && qs?.live_preview) {
    $stack.livePreviewQuery(qs as unknown as LivePreviewQuery);
  }

  const result = await $stack.contentType("blog_post").entry().query().find();

  if (result.entries && $preview) {
    result.entries.forEach((entry) => {
      contentstack.Utils.addEditableTags(entry, "blog_post", true);
    });
  }

  return result.entries;
});
</script>

<template>
  <div>
    <article
      v-for="(entry, index) in entries"
      :key="entry.uid"
      v-bind="entry?.$ && entry?.$[`${index}`]"
    >
      <h2 v-if="entry.title" v-bind="entry?.$ && entry?.$.title">
        {{ entry.title }}
      </h2>
    </article>
  </div>
</template>
```

## Critical SSR Implementation Notes

### SSR Mode Characteristics

- **Page Refresh**: Page refreshes entirely when content changes
- **Query Parameter Driven**: Relies on query parameters for Live Preview context
- **Server-side Processing**: Content is processed on the server for each request
- **SEO Optimized**: Better for SEO and initial page load performance

### SSR Live Preview Requirements

1. **Plugin Configuration**: Set `ssr: true` in Live Preview initialization
2. **Query Parameter Handling**: Apply `$stack.livePreviewQuery()` when `live_preview` parameter is present
3. **No Client-side Refresh**: No need for `onEntryChange()` or refresh functions
4. **Editable Tags**: Use `contentstack.Utils.addEditableTags()` and bind with `v-bind="entry?.$ && entry?.$.field"`
5. **Automatic Updates**: Page refreshes automatically when content changes

### Data Fetching Best Practices for SSR

```typescript
// ALWAYS use the TypeScript Delivery SDK
import contentstack from "@contentstack/delivery-sdk";

// CRITICAL: Configure live_preview in stack initialization
const stack = contentstack.stack({
  // ... other config
  live_preview: {
    enable: preview ? true : false,
    preview_token: previewToken,
    host: endpoints.preview,
  },
});

// ALWAYS apply Live Preview query when parameters are present
if ($preview && qs?.live_preview) {
  $stack.livePreviewQuery(qs as unknown as LivePreviewQuery);
}

// ALWAYS add editable tags after fetching in preview mode
if ($preview) {
  contentstack.Utils.addEditableTags(entry, "content_type_uid", true);
}

// SSR: Return data and status only (no refresh function needed)
return { data, status };
```

### Vue Template Integration Requirements

```vue
<!-- REQUIRED: Bind editable tags for all fields -->
<h1 v-if="entry?.title" v-bind="entry?.$ && entry?.$.title">
  {{ entry.title }}
</h1>

<!-- REQUIRED: Handle arrays with index-based tags -->
<div
  v-for="(item, index) in content?.blocks"
  :key="item.block._metadata.uid"
  v-bind="content?.$ && content?.$[`blocks__${index}`]"
>
  <!-- block content -->
</div>

<!-- REQUIRED: Bind nested field tags -->
<img
  v-if="entry?.image"
  :src="entry.image.url"
  :alt="entry.image.title"
  v-bind="entry?.image?.$ && entry?.image?.$.url"
/>
```

## Testing SSR Live Preview Integration

1. **Enable Live Preview in Contentstack**: Settings → Live Preview → Enable and select Preview environment
2. **Set Environment Variables**: Ensure `NUXT_CONTENTSTACK_PREVIEW=true` is set
3. **Test in CMS**: Open any entry → Click Live Preview icon → Make changes
4. **Verify SSR Behavior**: Page should refresh entirely when content changes
5. **Check Edit Buttons**: Ensure edit buttons appear on tagged elements in the preview
6. **Check Query Parameters**: Verify `live_preview`, `entry_uid`, and `content_type_uid` appear in URL

## Troubleshooting SSR Live Preview

### Common SSR Integration Issues

1. **Preview not working**: Check that `$stack.livePreviewQuery()` is called when query parameters are present
2. **Missing edit buttons**: Verify that editable tags are properly bound with `v-bind`
3. **Content not updating**: Ensure Live Preview is enabled in Contentstack settings
4. **Page not refreshing**: Verify `ssr: true` is set in plugin configuration

### SSR Debug Checklist

- [ ] Environment variables properly configured
- [ ] Preview token has correct scopes in Contentstack
- [ ] Plugin initializes only on client-side (`import.meta.client`)
- [ ] `ssr: true` set in Live Preview initialization
- [ ] `$stack.livePreviewQuery()` called when `live_preview` parameter is present
- [ ] Editable tags bound using `v-bind="entry?.$ && entry?.$.field"` pattern
- [ ] `@timbenniks/contentstack-endpoints` transpiled in build configuration
- [ ] Live Preview enabled in Contentstack Settings
- [ ] Query parameters (`live_preview`, `entry_uid`, `content_type_uid`) present in preview URLs

### SSR vs CSR Comparison

| Feature                | SSR Mode     | CSR Mode                  |
| ---------------------- | ------------ | ------------------------- |
| Content Updates        | Page refresh | Real-time without refresh |
| SEO Performance        | Excellent    | Good                      |
| Initial Load           | Faster       | Slower                    |
| Interactive Experience | Basic        | Enhanced                  |
| Server Load            | Higher       | Lower                     |
| Client-side JS         | Minimal      | More required             |

## Migration from CSR to SSR

1. **Change Plugin Configuration**: Set `ssr: true` instead of `ssr: false`
2. **Remove Client-side Refresh**: Remove `onMounted()` and `onEntryChange()` logic
3. **Simplify Composables**: Remove refresh function returns from data fetching
4. **Test Page Refresh**: Verify page refreshes when content changes in Live Preview

This focused SSR guide enables AI agents to add Live Preview functionality to existing Nuxt 4 applications with server-side rendering and page refresh behavior for content updates.
