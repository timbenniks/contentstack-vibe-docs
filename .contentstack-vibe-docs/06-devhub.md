# Contentstack Developer Hub & Custom Apps Guide for AI Agents

This guide covers building custom apps and extensions for Contentstack Developer Hub, including the App SDK, UI locations, and implementation patterns. It is written for AI coding assistants to help implement Developer Hub apps quickly and accurately.

## Table of Contents

1. [What is Contentstack Developer Hub?](#what-is-contentstack-developer-hub)
2. [Key Concepts](#key-concepts)
3. [How This App Was Built](#how-this-app-was-built)
4. [Architecture Overview](#architecture-overview)
5. [Implementation Details](#implementation-details)
6. [UI Locations](#ui-locations)
7. [Development Workflow](#development-workflow)

---

## What is Contentstack Developer Hub?

[Contentstack Developer Hub](https://www.contentstack.com/docs/developers/developer-hub) is a platform that allows developers to build custom applications that extend and enhance the Contentstack CMS experience. It provides APIs, SDKs, and tools to create apps that integrate seamlessly into Contentstack's user interface.

### Key Features

- **App Framework**: Complete framework for building custom apps with React, TypeScript, and modern tooling
- **UI Locations**: Multiple integration points within Contentstack's interface (sidebars, dashboards, custom fields, etc.)
- **SDK Integration**: JavaScript SDK (`@contentstack/app-sdk`) for interacting with Contentstack's APIs and UI
- **Marketplace**: Ability to publish apps to Contentstack's marketplace or keep them private
- **OAuth Integration**: Secure authentication and authorization for apps
- **App Hosting**: Options for external hosting or Contentstack-managed hosting

### Types of Apps

1. **Marketplace Apps**: Public apps available to all Contentstack users
2. **Private Apps**: Organization-specific apps for internal use
3. **Machine-to-Machine Apps**: Apps that interact with Contentstack APIs without user interface

---

## Key Concepts

### App Manifest

The app manifest is configured in the Contentstack Developer Hub platform (not in the codebase). It defines:

- App metadata (name, description, icon)
- UI locations where the app appears
- Routing configuration (paths to your app's routes)
- Hosting information (base URL)
- Visibility settings (public/private)

The manifest configuration is done through the Developer Hub UI when creating or managing your app.

### UI Locations

UI locations are specific places in Contentstack's interface where your app can appear:

- **Asset Sidebar** (`cs.cm.stack.asset_sidebar`): Sidebar in the Asset Library
- **Entry Sidebar** (`cs.cm.stack.sidebar`): Sidebar when editing content entries
- **Custom Field** (`cs.cm.stack.custom_field`): Custom field type for content types
- **Dashboard** (`cs.cm.stack.dashboard`): Widget on the stack dashboard
- **App Config** (`cs.cm.stack.config`): Configuration page for app settings
- **Full Page** (`cs.cm.stack.full_page`): Standalone full-page app
- **Field Modifier** (`cs.cm.stack.field_modifier`): Modifies field values
- **RTE Location** (`cs.cm.stack.rte`): Rich text editor extensions

### App SDK

The `@contentstack/app-sdk` provides:

- **SDK Initialization**: `ContentstackAppSDK.init()` - Initializes the SDK and establishes connection
- **Location Access**: Access to current UI location and its data
- **Asset Operations**: Methods to read, upload, and replace assets
- **Entry Operations**: Methods to read and modify content entries
- **Configuration**: Access to app configuration settings
- **Frame Management**: Control iframe dimensions and auto-resizing

**Basic SDK Usage:**

```typescript
import ContentstackAppSDK from "@contentstack/app-sdk";

// Initialize SDK
const appSdk = await ContentstackAppSDK.init();

// Access current location
const location = appSdk.location;

// Access configuration
const config = await appSdk.getConfig();

// Access stack data
const stack = appSdk.stack;
```

---

## How This App Was Built

### Overview

This app is an **Asset Sidebar Widget** that enables AI-powered image editing using Google's Gemini 2.5 Flash Image model. Users can modify images in Contentstack's Asset Library by providing text prompts.

### Boilerplate Foundation

This codebase is built on the [Contentstack Marketplace App Boilerplate](https://github.com/contentstack/marketplace-app-boilerplate), which provides:

- **Pre-configured SDK Setup**: Ready-to-use `MarketplaceAppProvider` for SDK initialization
- **Custom Hooks**: Pre-built hooks like `useAppSdk`, `useAppLocation`, `useAppConfig`, etc.
- **Routing Structure**: React Router setup with lazy loading for optimal performance
- **UI Location Templates**: Scaffolding for all available UI locations (Asset Sidebar, Custom Field, Dashboard, etc.)
- **Testing Infrastructure**: E2E testing setup with Playwright
- **Build Configuration**: Vite configuration optimized for Contentstack apps
- **TypeScript Support**: Full TypeScript setup with proper type definitions

The boilerplate provides a solid foundation, and this app extends it by implementing the Asset Sidebar location with Gemini AI integration for image editing functionality.

### Technology Stack

- **React 18**: UI framework
- **TypeScript**: Type safety
- **Vite**: Build tool and dev server
- **Contentstack App SDK**: Integration with Contentstack
- **Google Gemini API**: AI image generation/modification
- **React Router**: Client-side routing

### Project Structure

```
src/
├── containers/
│   ├── AssetSidebarWidget/     # Main feature - Asset sidebar widget
│   │   ├── AssetSidebar.tsx    # Core component
│   │   └── AssetSidebar.css    # Styles
│   ├── App/                     # Main app component with routing
│   └── AppConfiguration/        # App configuration page
├── common/
│   ├── providers/              # React context providers
│   │   └── MarketplaceAppProvider.tsx  # SDK initialization
│   ├── hooks/                   # Custom React hooks
│   │   ├── useAppSdk.tsx       # Access to SDK instance
│   │   └── useAppLocation.ts   # Current UI location
│   └── contexts/                # React contexts
│       └── marketplaceContext.ts
└── components/                  # Reusable components
    └── ErrorBoundary.tsx
```

---

## Architecture Overview

### Initialization Flow

1. **App Loads**: React app loads in Contentstack's iframe
2. **SDK Initialization**: `MarketplaceAppProvider` calls `ContentstackAppSDK.init()`
3. **Token Validation**: App verifies authentication token from URL
4. **Location Detection**: SDK identifies which UI location is active
5. **Context Setup**: SDK instance and config are provided via React Context

**Example Provider Implementation:**

```typescript
// MarketplaceAppProvider.tsx
import { useEffect, useState } from "react";
import ContentstackAppSDK from "@contentstack/app-sdk";

export function MarketplaceAppProvider({ children }) {
  const [appSdk, setAppSdk] = useState(null);
  const [config, setConfig] = useState(null);
  const [failed, setFailed] = useState(false);

  useEffect(() => {
    ContentstackAppSDK.init()
      .then(async (sdk) => {
        setAppSdk(sdk);
        const appConfig = await sdk.getConfig();
        setConfig(appConfig);
      })
      .catch(() => {
        setFailed(true);
      });
  }, []);

  if (failed) return <div>Failed to initialize SDK</div>;
  if (!appSdk) return <div>Loading...</div>;

  return (
    <MarketplaceContext.Provider value={{ appSdk, config }}>
      {children}
    </MarketplaceContext.Provider>
  );
}
```
6. **Component Rendering**: UI components access SDK via `useAppSdk()` hook

### Data Flow

```
Contentstack UI (iframe)
    ↓
MarketplaceAppProvider
    ↓
ContentstackAppSDK.init()
    ↓
React Context (appSdk, appConfig)
    ↓
AssetSidebar Component
    ↓
useAppSdk() hook
    ↓
Asset operations (getData, replaceAsset)
```

### Image Editing Workflow

1. **User Input**: User enters text prompt in the sidebar
2. **Asset Retrieval**: App gets current asset data via `assetSidebar.getData()`
3. **Image Conversion**: Fetches image URL and converts to base64
4. **API Request**: Sends prompt + base64 image to Google Gemini API
5. **Image Generation**: Gemini processes and returns modified image
6. **Asset Replacement**: Converts base64 response to File object
7. **Update Asset**: Calls `assetSidebar.replaceAsset(file)` to update in Contentstack

---

## Implementation Details

### 1. SDK Initialization

Located in `src/common/providers/MarketplaceAppProvider.tsx`:

```typescript
useEffect(() => {
  ContentstackAppSDK.init()
    .then(async (appSdk) => {
      setAppSdk(appSdk);
      const appConfig = await appSdk.getConfig();
      setConfig(appConfig);
    })
    .catch(() => {
      setFailed(true);
    });
}, []);
```

### 2. Asset Sidebar Integration

The main component (`AssetSidebar.tsx`) accesses the SDK:

```typescript
const appSdk = useAppSdk();
const assetSidebar = (appSdk?.location as any)?.AssetSidebarWidget;

// Get current asset
const assetData = assetSidebar.getData();

// Replace asset with new image
await assetSidebar.replaceAsset(file);
```

### 3. Gemini API Integration

The app uses Google's Gemini 2.5 Flash Image model through Contentstack's API proxy with advanced settings:

- **API Proxy**: Uses `appSdk.api()` to make requests through Contentstack's API proxy
- **Endpoint Path**: `/genai/gemini-2.5-flash-image:generateContent` (configured via API rewrites in Developer Hub)
- **Authentication**: API key is managed through Contentstack's Advanced Settings using variable placeholders (`{{var.API_KEY}}`)
- **Request Format**: REST API with base64 image data and text prompt
- **Response Handling**: Extracts base64 image from response and converts to File

The API proxy and variable substitution are configured in the Developer Hub's Advanced Settings, allowing secure API key management without exposing credentials in the codebase.

**Code Example:**

```typescript
const response = await appSdk.api(
  "/genai/gemini-2.5-flash-image:generateContent",
  {
    method: "POST",
    headers: {
      "x-goog-api-key": `{{var.API_KEY}}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify(requestBody),
  }
);
```

The `{{var.API_KEY}}` placeholder is automatically replaced by Contentstack with the actual API key value configured in Advanced Settings.

### 4. Error Handling

The app includes comprehensive error handling:

- API request failures
- Image conversion errors
- Asset operation failures
- User-friendly error messages with auto-hide

### 5. UI Features

- **Input Validation**: Disables button when input is empty
- **Loading States**: Shows "Generating Image..." during API calls
- **Success/Error Messages**: Color-coded messages with 5-second auto-hide
- **MIME Type Detection**: Automatically detects image format (PNG, JPEG, GIF, WebP)

---

## UI Locations

This app is configured to support multiple UI locations (configured in the Developer Hub platform):

### Active Location

- **Asset Sidebar** (`/asset-sidebar`): The main feature - image editing widget

### Available but Not Implemented

- Custom Field (`/custom-field`)
- Dashboard (`/stack-dashboard`)
- Content Type Sidebar (`/content-type-sidebar`)
- Entry Sidebar (`/entry-sidebar`)
- Full Page (`/full-page`)
- Field Modifier (`/field-modifier`)
- App Configuration (`/app-configuration`)
- RTE Location (`/json-rte.js`)

These locations can be configured in the Developer Hub platform, but only the Asset Sidebar has a full implementation in this app.

---

## Development Workflow

### Local Development

1. **Install Dependencies**:

   ```bash
   npm install
   ```

2. **Start Dev Server**:

   ```bash
   npm run dev
   ```

   Runs on `http://localhost:3000`

3. **Configure in Developer Hub**:
   - Create app in Contentstack Developer Hub
   - Set base URL to `http://localhost:3000`
   - Configure Asset Sidebar location in the manifest settings with path `/asset-sidebar`
   - Set up API Rewrites in Advanced Settings:
     - Add rewrite rule: `/genai/*` → `https://generativelanguage.googleapis.com/v1beta/models/*`
   - Configure Variables in Advanced Settings:
     - Add variable `API_KEY` with your Google Gemini API key value

### Building for Production

```bash
npm run build
```

Outputs to `dist/` directory for deployment.

### Testing

- **Unit Tests**: `npm test`
- **E2E Tests**: `npm run test:chrome` or `npm run test:firefox`
- **Type Checking**: `npm run typecheck`

### Deployment

1. Build the app: `npm run build`
2. Deploy `dist/` folder to hosting provider
3. Update app configuration in Developer Hub with production URL
4. Update manifest settings in Developer Hub if needed

---

## Key Files Reference

### Core Files

- **`src/main.tsx`**: Application entry point
- **`src/containers/App/App.tsx`**: Main app component with routing
- **`src/common/providers/MarketplaceAppProvider.tsx`**: SDK initialization
- **`src/containers/AssetSidebarWidget/AssetSidebar.tsx`**: Main feature component

### Configuration Files

- **`vite.config.ts`**: Vite build configuration
- **`tsconfig.json`**: TypeScript configuration
- **`package.json`**: Dependencies and scripts

---

## Resources

- [Contentstack Developer Hub Documentation](https://www.contentstack.com/docs/developers/developer-hub)
- [Contentstack App SDK](https://www.contentstack.com/docs/developers/developer-hub/contentstack-app-development)
- [UI Locations Reference](https://www.contentstack.com/docs/developers/developer-hub/managing-ui-locations)
- [Marketplace App Boilerplate (GitHub)](https://github.com/contentstack/marketplace-app-boilerplate) - The boilerplate this app is based on
- [Marketplace App Boilerplate Documentation](https://www.contentstack.com/docs/developers/developer-hub/marketplace-app-boilerplate)
- [Google Gemini API Documentation](https://ai.google.dev/docs)

## Next Steps

- Review [Base Concepts](00-contentstack-base-concepts.md) to understand Contentstack fundamentals
- Learn about [API Functionalities](01-api-functionalities.md) for API integration patterns
- Explore [SDK Functionalities](02-sdk-functionalities.md) for SDK usage patterns
- Check [Practical Examples](05-practical-examples.md) for React/TypeScript implementation examples

---

## Notes

- **Built on Boilerplate**: This app extends the [Contentstack Marketplace App Boilerplate](https://github.com/contentstack/marketplace-app-boilerplate) with custom image editing functionality
- **API Integration**: The app uses Contentstack's API proxy (`appSdk.api()`) with Advanced Settings for secure API key management and URL rewrites
- **Variable Substitution**: API keys are managed through Contentstack's Advanced Settings using `{{var.API_KEY}}` placeholders, keeping credentials secure
- Asset operations use Contentstack's SDK methods for seamless integration
- The app handles both camelCase and snake_case API response formats for compatibility
- Error messages are user-friendly and automatically hide after 5 seconds
