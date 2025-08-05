# Contentstack Live Preview Integration Guide for Nuxt 4 CSR

This guide provides focused instructions for AI coding assistants to integrate Contentstack Live Preview into an existing Nuxt 4 application using Client-Side Rendering (CSR) mode.

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

  // CRITICAL: Initialize Live Preview only on client-side for CSR mode
  if (preview && import.meta.client) {
    ContentstackLivePreview.init({
      ssr: false, // CRITICAL: CSR mode - enables real-time updates without page refresh
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

Create a composable for fetching page data with Live Preview support:

```typescript
// composables/useGetPage.ts
import contentstack, {
  QueryOperation,
  type LivePreviewQuery,
} from "@contentstack/delivery-sdk";

/**
 * Retrieve Contentstack Page Data with Live Preview Support (CSR Mode)
 *
 * This composable fetches page data from Contentstack CMS using Nuxt's composables.
 * It supports both regular content delivery and live preview functionality with
 * real-time content updates without page refresh.
 *
 * Features:
 * - Fetches page content based on URL parameter
 * - Supports Contentstack Live Preview mode
 * - Handles data fetching with Nuxt's useAsyncData
 * - Implements type-safe Contentstack queries
 * - Adds editable tags for content management
 * - Provides refresh function for real-time updates
 *
 * Usage:
 * const { data, status, refresh } = await useGetPage("/about");
 */
export const useGetPage = async (url: string) => {
  // Use the useAsyncData hook to fetch data and manage its state
  const { data, status, refresh } = await useAsyncData(
    `page-${url}`,
    async () => {
      // Get the Nuxt app instance and the current route
      const { $preview, $stack } = useNuxtApp();
      const route = useRoute();
      const qs = { ...toRaw(route.query) }; // Convert the route query to a raw object

      // CRITICAL: Check if preview mode is enabled and if live preview query parameter is present
      if ($preview && qs?.live_preview) {
        $stack.livePreviewQuery(qs as unknown as LivePreviewQuery);
      }

      // Fetch the page data from Contentstack
      const result = await $stack
        .contentType("page") // Specify the content type as 'page'
        .entry() // Access the entries of the content type
        .query() // Create a query to filter the entries
        .where("url", QueryOperation.EQUALS, url) // Filter entries where the 'url' field matches the provided URL
        .find(); // Execute the query

      // Check if there are any entries in the result
      if (result.entries) {
        const entry = result.entries[0]; // Get the first entry from the result

        // CRITICAL: If preview mode is enabled, add editable tags to the entry
        if ($preview) {
          contentstack.Utils.addEditableTags(entry, "page", true);
        }

        return entry; // Return the entry as the data
      }
    }
  );

  // Return the data, status, and refresh function from useAsyncData
  return { data, status, refresh };
};
```

## CSR App Component Implementation

```vue
<!-- app.vue (CSR version) -->
<script lang="ts" setup>
import VB_EmptyBlockParentClass from "@contentstack/live-preview-utils";
import ContentstackLivePreview from "@contentstack/live-preview-utils";

// Fetch the page data and refresh function using the useGetPage composable
const { data: page, refresh } = await useGetPage("/");

// Set up live preview functionality when the component is mounted
onMounted(() => {
  // Get preview utilities from Nuxt app context
  const { $preview, $ContentstackLivePreview } = useNuxtApp();
  // Type assertion for ContentstackLivePreview
  const livePreview =
    $ContentstackLivePreview as unknown as typeof ContentstackLivePreview;

  // CRITICAL: If preview mode is active, set up live content updates
  // This enables real-time content updates without page refresh in CSR mode
  $preview && livePreview.onEntryChange(refresh);
});
</script>

<template>
  <main class="max-w-screen-md mx-auto">
    <section class="p-4">
      <!-- CRITICAL: Page title with live preview binding -->
      <h1
        v-if="page?.title"
        class="text-4xl font-bold mb-4 text-center"
        v-bind="page?.$ && page?.$.title"
      >
        {{ page?.title }} with Nuxt
      </h1>

      <!-- Page description with live preview binding -->
      <p
        v-if="page?.description"
        class="mb-4 text-center"
        v-bind="page?.$ && page?.$.description"
      >
        {{ page?.description }}
      </p>

      <!-- Featured image with responsive dimensions and live preview binding -->
      <img
        v-if="page?.image"
        class="mb-4"
        width="768"
        height="414"
        :src="page?.image.url"
        :alt="page?.image.title"
        v-bind="page?.image?.$ && page?.image?.$.url"
      />

      <!-- Rich text content with HTML rendering and live preview binding -->
      <div
        v-if="page?.rich_text"
        v-bind="page?.$ && page?.$.rich_text"
        v-html="page?.rich_text"
      />

      <!-- Dynamic blocks container with live preview support -->
      <div
        :class="[
          'space-y-8 max-w-full mt-4',
          // Apply empty block class for live preview when no blocks exist
          !page?.blocks || page.blocks.length === 0
            ? VB_EmptyBlockParentClass
            : '',
        ]"
        v-bind="page?.$ && page?.$.blocks"
      >
        <!-- Iterate through content blocks -->
        <div
          v-for="(item, index) in page?.blocks"
          :key="item.block._metadata.uid"
          v-bind="page?.$ && page?.$[`blocks__${index}`]"
          :class="[
            'flex flex-col md:flex-row items-center space-y-4 md:space-y-0 bg-white',
            // Dynamic layout class based on image position preference
            item.block.layout === 'image_left'
              ? 'md:flex-row'
              : 'md:flex-row-reverse',
          ]"
        >
          <!-- Block image section -->
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

          <!-- Block text content section -->
          <div class="w-full md:w-1/2 p-4">
            <!-- Block title with live preview binding -->
            <h2
              v-if="item.block.title"
              class="text-2xl font-bold"
              v-bind="item.block.$ && item.block.$.title"
            >
              {{ item.block.title }}
            </h2>
            <!-- Block copy/content with HTML rendering -->
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

<style lang="css">
body {
  background: #e9e9e9;
}

a {
  text-decoration: underline;
}
</style>
```

## CSR Live Preview Integration Patterns

### Pattern 1: Basic Page Integration

```vue
<script lang="ts" setup>
const { data: page, refresh } = await useGetPage("/");

// CSR - Set up live preview refresh for real-time updates
onMounted(() => {
  const { $preview, $ContentstackLivePreview } = useNuxtApp();
  const livePreview =
    $ContentstackLivePreview as unknown as typeof ContentstackLivePreview;
  $preview && livePreview.onEntryChange(refresh);
});
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

const { data: page, refresh } = await useGetPage(slug);

// CSR Live Preview setup for real-time content updates
onMounted(() => {
  const { $preview, $ContentstackLivePreview } = useNuxtApp();
  const livePreview =
    $ContentstackLivePreview as unknown as typeof ContentstackLivePreview;
  $preview && livePreview.onEntryChange(refresh);
});
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
// For multiple entries with CSR live updates
const { data: entries, refresh } = await useAsyncData(
  "blog-posts",
  async () => {
    const { $preview, $stack } = useNuxtApp();
    const route = useRoute();
    const qs = { ...toRaw(route.query) };

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
  }
);

// CSR Live Preview setup for collection updates
onMounted(() => {
  const { $preview, $ContentstackLivePreview } = useNuxtApp();
  const livePreview =
    $ContentstackLivePreview as unknown as typeof ContentstackLivePreview;
  $preview && livePreview.onEntryChange(refresh);
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

## Critical CSR Implementation Notes

### CSR Mode Characteristics

- **Real-time Updates**: Content updates instantly without page refresh
- **Event-driven**: Uses `onEntryChange()` event handler for live updates
- **Client-side Refresh**: Requires refresh function from composables
- **Interactive Experience**: Better for dynamic user experiences and immediate feedback

### CSR Live Preview Requirements

1. **Plugin Configuration**: Set `ssr: false` in Live Preview initialization
2. **Client-side Only**: Initialize Live Preview only on client-side (`import.meta.client`)
3. **Event Handling**: Use `onEntryChange(refresh)` in `onMounted()` lifecycle
4. **Query Parameter Handling**: Apply `$stack.livePreviewQuery()` when `live_preview` parameter is present
5. **Editable Tags**: Use `contentstack.Utils.addEditableTags()` and bind with `v-bind="entry?.$ && entry?.$.field"`

### Data Fetching Best Practices for CSR

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

// CRITICAL: Return refresh function for CSR updates
return { data, status, refresh };
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

## Testing CSR Live Preview Integration

1. **Enable Live Preview in Contentstack**: Settings → Live Preview → Enable and select Preview environment
2. **Set Environment Variables**: Ensure `NUXT_CONTENTSTACK_PREVIEW=true` is set
3. **Test in CMS**: Open any entry → Click Live Preview icon → Make changes
4. **Verify CSR Behavior**: Content should update instantly without page refresh
5. **Check Edit Buttons**: Ensure edit buttons appear on tagged elements in the preview

## Troubleshooting CSR Live Preview

### Common CSR Integration Issues

1. **Preview not updating**: Check that `onEntryChange(refresh)` is set up in `onMounted()`
2. **Missing edit buttons**: Verify that editable tags are properly bound with `v-bind`
3. **Page refreshing instead of updating**: Ensure `ssr: false` is set in plugin configuration
4. **Content not refreshing**: Verify refresh function is properly returned from composable

### CSR Debug Checklist

- [ ] Environment variables properly configured
- [ ] Preview token has correct scopes in Contentstack
- [ ] Plugin initializes only on client-side (`import.meta.client`)
- [ ] `ssr: false` set in Live Preview initialization
- [ ] `onEntryChange(refresh)` set up in `onMounted()` hook
- [ ] Editable tags bound using `v-bind="entry?.$ && entry?.$.field"` pattern
- [ ] `@timbenniks/contentstack-endpoints` transpiled in build configuration
- [ ] Live Preview enabled in Contentstack Settings

This focused CSR guide enables AI agents to add real-time Live Preview functionality to existing Nuxt 4 applications with instant content updates without page refresh.
