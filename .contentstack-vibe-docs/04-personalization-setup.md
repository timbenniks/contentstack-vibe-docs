# Contentstack Personalization Setup for AI Agents

This guide covers Contentstack's Personalization features, including how to set up and implement personalized content delivery for AI coding assistants.

## Overview

Contentstack Personalization allows you to deliver personalized content to users based on their profile, behavior, location, and other attributes. It enables A/B testing, targeted content delivery, and dynamic content variations.

## Key Concepts

### Personas

**Personas** are user profiles that define characteristics and attributes:

- **Built-in Attributes**: Geographic location, device type, referrer, etc.
- **Custom Attributes**: User-defined properties (e.g., subscription tier, interests)
- **Segments**: Groups of users sharing similar characteristics

### Variants

**Variants** are different versions of content:

- **Default Variant**: Base content shown to all users
- **Personalized Variants**: Content tailored to specific personas/segments
- **A/B Test Variants**: Different versions for testing

### Rules

**Rules** define when variants should be shown:

- **Condition-based**: Show variant when user matches conditions
- **Percentage-based**: Show variant to a percentage of users
- **Time-based**: Show variant during specific time periods

## Setup Process

### 1. Enable Personalization in Stack

1. Navigate to **Settings → Personalization** in Contentstack
2. Enable **Personalization** feature
3. Configure default settings

### 2. Create Personas

1. Go to **Personalization → Personas**
2. Create personas based on user characteristics:
   - Geographic location
   - Device type
   - Browser
   - Custom attributes (subscription level, interests, etc.)

**Example Persona:**

- **Name**: Premium Subscriber
- **Attributes**:
  - Subscription tier: Premium
  - Geographic region: North America
  - Device: Desktop

### 3. Create Content Variants

1. Edit any entry in Contentstack
2. Click **Personalize** button
3. Create variants for different personas
4. Assign variants to personas or segments

### 4. Configure Delivery Token

Ensure your Delivery Token has Personalization enabled:

1. Go to **Settings → Tokens → Delivery Tokens**
2. Edit your delivery token
3. Enable **Personalization** scope
4. Save token

## SDK Implementation

### Basic Setup

```typescript
import contentstack from "@contentstack/delivery-sdk";

const stack = contentstack.stack({
  apiKey: "YOUR_API_KEY",
  deliveryToken: "YOUR_DELIVERY_TOKEN",
  environment: "production",
  region: "us",
});
```

### Fetching Personalized Content

Personalization works automatically when user attributes are provided. The SDK includes user context in API requests.

#### Method 1: Using Headers

```typescript
const entry = await stack
  .contentType("page")
  .entry("entry_uid")
  .addParam(
    "personalization",
    JSON.stringify({
      user: {
        uid: "user_123",
        location: {
          country: "US",
          region: "CA",
          city: "San Francisco",
        },
        device: {
          type: "desktop",
          browser: "Chrome",
        },
        custom: {
          subscription_tier: "premium",
          interests: ["technology", "design"],
        },
      },
    })
  )
  .fetch();
```

#### Method 2: Using Personalization Object

```typescript
const entry = await stack
  .contentType("page")
  .entry("entry_uid")
  .personalization({
    user: {
      uid: "user_123",
      location: {
        country: "US",
        region: "CA",
        city: "San Francisco",
      },
      device: {
        type: "desktop",
        browser: "Chrome",
      },
      custom: {
        subscription_tier: "premium",
        interests: ["technology", "design"],
      },
    },
  })
  .fetch();
```

### User Context Object Structure

```typescript
interface UserContext {
  uid?: string; // Unique user identifier
  location?: {
    country?: string; // ISO country code (e.g., "US")
    region?: string; // State/Province code
    city?: string; // City name
    latitude?: number; // Latitude coordinate
    longitude?: number; // Longitude coordinate
  };
  device?: {
    type?: string; // "desktop", "mobile", "tablet"
    browser?: string; // Browser name
    os?: string; // Operating system
  };
  referrer?: string; // Referrer URL
  custom?: Record<string, any>; // Custom attributes
}
```

## Implementation Patterns

### Pattern 1: Server-Side Personalization (SSR)

```typescript
// Next.js, Nuxt, Express, etc.
export async function getPersonalizedPage(
  entryUid: string,
  userContext: UserContext
) {
  const entry = await stack
    .contentType("page")
    .entry(entryUid)
    .personalization({ user: userContext })
    .fetch();

  return entry;
}

// Usage in route handler
app.get("/page/:uid", async (req, res) => {
  const userContext = {
    uid: req.user?.id,
    location: {
      country: req.headers["cf-ipcountry"] || "US",
      city: req.headers["cf-ipcity"],
    },
    device: {
      type: req.headers["user-agent"]?.includes("Mobile")
        ? "mobile"
        : "desktop",
    },
    custom: {
      subscription_tier: req.user?.subscriptionTier || "free",
    },
  };

  const page = await getPersonalizedPage(req.params.uid, userContext);
  res.render("page", { page });
});
```

### Pattern 2: Client-Side Personalization (CSR)

```typescript
// React, Vue, etc.
async function fetchPersonalizedContent(entryUid: string) {
  // Get user context from client-side
  const userContext = {
    uid: getUserId(),
    location: await getUserLocation(),
    device: {
      type: getDeviceType(),
      browser: getBrowser(),
    },
    custom: {
      subscription_tier: getUserSubscriptionTier(),
      interests: getUserInterests(),
    },
  };

  const entry = await stack
    .contentType("page")
    .entry(entryUid)
    .personalization({ user: userContext })
    .fetch();

  return entry;
}
```

### Pattern 3: Middleware Pattern

```typescript
// Express middleware example
function personalizationMiddleware(req, res, next) {
  req.userContext = {
    uid: req.user?.id,
    location: {
      country: req.headers["cf-ipcountry"] || "US",
      region: req.headers["cf-ipregion"],
      city: req.headers["cf-ipcity"],
    },
    device: {
      type: req.headers["user-agent"]?.includes("Mobile")
        ? "mobile"
        : "desktop",
    },
    custom: {
      subscription_tier: req.user?.subscriptionTier || "free",
    },
  };
  next();
}

// Use middleware
app.use(personalizationMiddleware);

// Then in routes
app.get("/page/:uid", async (req, res) => {
  const entry = await stack
    .contentType("page")
    .entry(req.params.uid)
    .personalization({ user: req.userContext })
    .fetch();

  res.render("page", { page: entry });
});
```

## Helper Functions

### Get User Location

```typescript
async function getUserLocation(): Promise<UserContext["location"]> {
  try {
    // Option 1: Use geolocation API
    const position = await new Promise<GeolocationPosition>(
      (resolve, reject) => {
        navigator.geolocation.getCurrentPosition(resolve, reject);
      }
    );

    return {
      latitude: position.coords.latitude,
      longitude: position.coords.longitude,
    };
  } catch (error) {
    // Option 2: Use IP-based geolocation
    const response = await fetch("https://ipapi.co/json/");
    const data = await response.json();

    return {
      country: data.country_code,
      region: data.region_code,
      city: data.city,
    };
  }
}
```

### Get Device Type

```typescript
function getDeviceType(): string {
  const ua = navigator.userAgent;
  if (/tablet|ipad|playbook|silk/i.test(ua)) {
    return "tablet";
  }
  if (
    /mobile|iphone|ipod|android|blackberry|opera|mini|windows\sce|palm|smartphone|iemobile/i.test(
      ua
    )
  ) {
    return "mobile";
  }
  return "desktop";
}
```

### Get Browser

```typescript
function getBrowser(): string {
  const ua = navigator.userAgent;
  if (ua.includes("Chrome")) return "Chrome";
  if (ua.includes("Firefox")) return "Firefox";
  if (ua.includes("Safari")) return "Safari";
  if (ua.includes("Edge")) return "Edge";
  return "Unknown";
}
```

## Integration with Live Preview

Personalization works alongside Live Preview:

```typescript
import ContentstackLivePreview from "@contentstack/live-preview-utils";

const stack = contentstack.stack({
  apiKey: "YOUR_API_KEY",
  deliveryToken: "YOUR_DELIVERY_TOKEN",
  environment: "preview",
  region: "us",
  live_preview: {
    enable: true,
    preview_token: "YOUR_PREVIEW_TOKEN",
    host: "rest-preview.contentstack.com",
  },
});

// Initialize Live Preview
ContentstackLivePreview.init({
  ssr: false,
  enable: true,
  stackSdk: stack.config as IStackSdk,
  stackDetails: {
    apiKey: "YOUR_API_KEY",
    environment: "preview",
  },
});

// Fetch with both Live Preview and Personalization
const entry = await stack
  .contentType("page")
  .entry("entry_uid")
  .personalization({
    user: {
      uid: "user_123",
      location: { country: "US" },
      custom: { subscription_tier: "premium" },
    },
  })
  .fetch();
```

## A/B Testing

Contentstack supports A/B testing through variants:

```typescript
// Fetch entry with A/B test variant
const entry = await stack
  .contentType("page")
  .entry("entry_uid")
  .personalization({
    user: {
      uid: "user_123",
      // A/B test variant is automatically selected based on rules
    },
  })
  .fetch();

// Check which variant was returned
if (entry._variant) {
  console.log("Variant:", entry._variant.name);
  console.log("Variant UID:", entry._variant.uid);
}
```

## Best Practices

### 1. Always Provide User Context

```typescript
// Good: Provide comprehensive user context
const userContext = {
  uid: userId,
  location: { country, region, city },
  device: { type, browser },
  custom: { subscription_tier, interests },
};

// Bad: Missing user context
const entry = await stack.contentType("page").entry("uid").fetch();
```

### 2. Cache Personalized Content Appropriately

```typescript
// Cache key should include user context
function getCacheKey(entryUid: string, userContext: UserContext): string {
  const contextHash = hashObject(userContext);
  return `entry_${entryUid}_${contextHash}`;
}

async function getCachedPersonalizedEntry(
  entryUid: string,
  userContext: UserContext
) {
  const cacheKey = getCacheKey(entryUid, userContext);
  const cached = cache.get(cacheKey);

  if (cached) {
    return cached;
  }

  const entry = await stack
    .contentType("page")
    .entry(entryUid)
    .personalization({ user: userContext })
    .fetch();

  cache.set(cacheKey, entry, 300); // 5 minute TTL
  return entry;
}
```

### 3. Handle Missing User Data Gracefully

```typescript
function buildUserContext(req: Request): UserContext {
  return {
    uid: req.user?.id,
    location: {
      country: req.headers["cf-ipcountry"] || "US",
      // Fallback to defaults if unavailable
    },
    device: {
      type: detectDeviceType(req.headers["user-agent"]),
    },
    custom: {
      subscription_tier: req.user?.subscriptionTier || "free",
    },
  };
}
```

### 4. Test Variants

```typescript
// Test different variants in development
const testVariants = async (entryUid: string) => {
  const variants = ["variant_a", "variant_b", "variant_c"];

  for (const variant of variants) {
    const entry = await stack
      .contentType("page")
      .entry(entryUid)
      .personalization({
        user: {
          uid: `test_user_${variant}`,
          custom: { test_variant: variant },
        },
      })
      .fetch();

    console.log(`Variant ${variant}:`, entry.title);
  }
};
```

## Troubleshooting

### Variants Not Showing

1. **Check Personalization is Enabled**: Verify in Contentstack Settings
2. **Verify Token Scope**: Ensure Delivery Token has Personalization enabled
3. **Check User Context**: Ensure user context matches persona attributes
4. **Review Rules**: Check variant rules in Contentstack UI

### Performance Issues

1. **Implement Caching**: Cache personalized content appropriately
2. **Reduce Context Size**: Only include necessary user attributes
3. **Use CDN**: Leverage Contentstack's CDN for faster delivery

### Debugging

```typescript
// Enable debug logging
const entry = await stack
  .contentType("page")
  .entry("entry_uid")
  .personalization({
    user: userContext,
    debug: true, // Enable debug mode
  })
  .fetch();

// Check variant information
console.log("Variant:", entry._variant);
console.log("Persona:", entry._persona);
```

## Next Steps

- Review [Live Preview Setup](03-live-preview-base-concepts.md)
- Learn about [Live Preview Implementation](03a-live-preview-implementation.md)
- Explore [Base Concepts](00-contentstack-base-concepts.md)
