**What Contentstack Live Preview is**

Contentstack's **Live Preview** feature lets content managers see changes instantly before publishing. Whenever an editor opens an entry, the CMS generates a _Live Preview hash_ that identifies the preview session and links the entry editor with the website loaded in an iframe. The Live Preview Utils SDK running on the site uses `postMessage()` events to communicate with the entry editor, fetches the latest content and injects it into the preview panel without reloading. Live Preview works for both client-side rendering (CSR) and server-side rendering (SSR): CSR sites use the SDK to inject updated content into the DOM immediately, whereas SSR sites listen for updates and then reload the page. Editors can also open the preview in different device sizes and locales to check omnichannel experiences.

**Key components and terms**

| Concept                  | What it means (summarised)                                                                                                                                                                                                                                                                  |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| _Live Preview hash_      | A session-specific ID appended to preview URLs. It identifies the editor session and ensures the preview shows the latest unpublished changes.                                                                                                                                              |
| _Preview token_          | A special API token (generated from **Settings → Tokens → Delivery Tokens**) that allows the site to call preview endpoints. It's associated with a delivery token and should be used for Live Preview instead of a management token.                                                       |
| _Live Preview Utils SDK_ | JavaScript library that connects the CMS and the front-end. It listens for entry-change events, injects content, adds edit buttons and supports edit tags. Version 3 introduces modes (`preview` vs `builder`), configurable Edit/Start Editing buttons and improved configuration options. |
| _Live Edit tags_         | Special `data-cslp` attributes pointing to the path of a field (e.g. `content_type.entry.locale.field`). The SDK reads these tags to highlight fields and show Edit buttons. Tags can be added manually or generated automatically with `addEditableTags()`.                                |

**How to set up Live Preview**

Below is a consolidated step-by-step outline. The exact code varies by framework, but the high-level process is consistent.

1. **Enable Live Preview in the stack**.

   - Navigate to **Settings → Environments**, set the base URL for each locale and save.
   - Under **Live Preview**, tick **Enable Live Preview**, choose a default preview environment and save. When enabled, an eye icon appears in the entry editor.

2. **Generate a Preview token**.

   - Go to **Settings → Tokens → Delivery Tokens**. Select an existing delivery token or create a new one and toggle **Create preview token**; copy the preview token.

3. **Build your site with a Contentstack SDK** (GraphQL, REST or Gatsby). The SDK will be configured differently for CSR or SSR:

   - **GraphQL - CSR**: initialise `@contentstack/live-preview-utils` with `ssr: false`, `enable: true`, stack details and client URL parameters. When the preview hash is present, change the GraphQL host from `graphql.contentstack.com` to `graphql-preview.contentstack.com` and add `live_preview`, `preview_token` and `include_applied_variants` headers. Use `onEntryChange()` inside `useEffect` to refetch data when an entry changes.
   - **GraphQL - SSR**: similar to CSR but set `ssr: true`. On the server, extract the hash from the query parameters and pass it to the GraphQL request headers; re-initialise the SDK for each request to avoid cross-session data.
   - **Gatsby (uses GraphQL)**: create a `live-preview.js` module that instantiates `ContentstackGatsby` with `preview_token`, environment and host; install and initialise the Live Preview Utils SDK; connect `onLiveEdit()` to fetch updated data and update component state. Add `__typename` and `uid` to GraphQL queries; use `addEditableTags()` to generate edit tags.
   - **REST - CSR**: configure the Delivery SDK with a `live_preview` object containing `preview_token`, `enable: true` and `host: 'rest-preview.contentstack.com'`. Upgrade to the new preview service to improve performance. Initialise the Live Preview Utils SDK (`ssr: false`) and use `onEntryChange()` to refetch data.
   - **REST - SSR**: same as CSR but set `ssr: true`. On each server request, create a fresh stack instance and call `Stack.livePreviewQuery(req.query)` to inject the hash into API calls. Always create a new SDK instance per request to avoid showing wrong preview data across users.

4. **Host the site** on a secure domain. Contentstack's **Launch** service or any other hosting provider can be used. The site must be iframe-compatible and served over HTTPS.
5. **Implement Live Edit tags (optional but recommended)**.

   - For custom HTML/templating languages, manually add `data-cslp` attributes with the field path: `{content_type_uid}.{entry_uid}.{locale}.{field_uid}`; nested fields use `{modular_block_field_UID}.{block_UID}.{field_UID}`.
   - For JavaScript frameworks using Contentstack SDKs, import `addEditableTags` from `@contentstack/utils` and call `addEditableTags(entry, content_type_uid, tagsAsObject, locale)` after fetching data. Set `tagsAsObject=true` when using JSX so the tags can be spread as props.

6. **Configure edit buttons**. The SDK provides an `editButton` object to enable or disable the button, choose positions (top, bottom-right, etc.) and decide whether to show it inside or outside the preview panel.

**Additional features**

- **Preview Service**: Contentstack has introduced a new Preview Service that replaces the Content Management API (CMA) for preview requests. To migrate, update the API base URL from `api.contentstack.io` to `rest-preview.contentstack.com` and use your preview token instead of the management token. For non-SDK implementations, add the preview hash and token to the headers and switch the hostname only when the hash is present.
- **SDK versions**: Version 3 of the Live Preview Utils SDK adds a `mode` property (`preview` for Live Preview and `builder` for Visual Builder) and a `start editing` button for quickly switching to edit mode in Visual Builder. It also lets you clean all `data-cslp` attributes in production using `cleanCslpOnProduction`.
- **Event-based communication**: The SDK relies on events such as `init-ack` (sent when the session starts) and `client-data-send` (triggers content update) to keep the preview in sync.
