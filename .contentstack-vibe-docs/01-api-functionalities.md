# Contentstack API Functionalities for AI Agents

This guide covers Contentstack's API capabilities including REST API and GraphQL API, their usage patterns, and best practices for AI coding assistants.

## Overview

Contentstack provides two main APIs for content delivery:

1. **REST API**: Traditional RESTful endpoints with JSON responses
2. **GraphQL API**: Flexible GraphQL queries for precise data fetching

Both APIs support:

- Published content delivery (via Delivery Token)
- Unpublished content preview (via Preview Token)
- Multiple regions and environments
- Locale filtering
- Content filtering, sorting, and pagination

## REST API

### Base URL Structure

```
https://{region}-cdn.contentstack.io/v3/{resource}
```

**Preview Base URL** (when using Preview Token):

```
https://{region}-rest-preview.contentstack.com/v3/{resource}
```

### Authentication

All REST API requests require authentication headers:

```http
api_key: YOUR_API_KEY
access_token: YOUR_DELIVERY_TOKEN
```

**For Preview** (unpublished content):

```http
api_key: YOUR_API_KEY
access_token: YOUR_PREVIEW_TOKEN
live_preview: PREVIEW_HASH (when in Live Preview mode)
```

### Common Endpoints

#### Fetch Single Entry

```http
GET /v3/content_types/{content_type_uid}/entries/{entry_uid}
```

**Query Parameters:**

- `environment`: Environment name (required)
- `locale`: Locale code (optional, defaults to default locale)

**Example Request:**

```bash
curl -X GET \
  'https://cdn.contentstack.io/v3/content_types/blog_post/entries/entry_uid?environment=production&locale=en-us' \
  -H 'api_key: YOUR_API_KEY' \
  -H 'access_token: YOUR_DELIVERY_TOKEN'
```

#### Fetch Multiple Entries

```http
GET /v3/content_types/{content_type_uid}/entries
```

**Query Parameters:**

- `environment`: Environment name (required)
- `locale`: Locale code (optional)
- `query`: JSON query for filtering (optional)
- `skip`: Number of entries to skip (pagination)
- `limit`: Number of entries to return (pagination, max 100)
- `include_count`: Include total count in response
- `asc`: Field to sort ascending
- `desc`: Field to sort descending

**Example Query:**

```json
{
  "query": {
    "title": { "$regex": "tutorial", "$options": "i" },
    "published_date": { "$lte": "2024-01-01" }
  },
  "skip": 0,
  "limit": 10,
  "asc": "published_date"
}
```

#### Query Operators

Contentstack REST API supports MongoDB-style query operators:

- `$eq`: Equal to
- `$ne`: Not equal to
- `$gt`: Greater than
- `$gte`: Greater than or equal to
- `$lt`: Less than
- `$lte`: Less than or equal to
- `$in`: Matches any value in array
- `$nin`: Matches none of the values in array
- `$exists`: Field exists
- `$regex`: Regular expression match
- `$and`: Logical AND
- `$or`: Logical OR

**Example:**

```json
{
  "query": {
    "$or": [
      { "category": { "$in": ["tech", "design"] } },
      { "tags": { "$regex": "javascript" } }
    ],
    "published_date": { "$gte": "2024-01-01" }
  }
}
```

#### Fetch Assets

```http
GET /v3/assets/{asset_uid}
```

**Query Parameters:**

- `environment`: Environment name (required)
- `version`: Asset version (optional)

#### Search Assets

```http
GET /v3/assets
```

**Query Parameters:**

- `environment`: Environment name (required)
- `query`: JSON query for filtering
- `skip`, `limit`: Pagination

### Including References

To include referenced entries in the response:

```http
GET /v3/content_types/{content_type_uid}/entries/{entry_uid}?include[]=reference_field_uid
```

**Multiple References:**

```http
GET /v3/content_types/{content_type_uid}/entries/{entry_uid}?include[]=author&include[]=category&include[]=tags
```

**Deep Includes** (include references within references):

```http
GET /v3/content_types/{content_type_uid}/entries/{entry_uid}?include[]=author.published_date
```

### Including Entry Count

```http
GET /v3/content_types/{content_type_uid}/entries?include_count=true
```

Response includes:

```json
{
  "entries": [...],
  "count": 42
}
```

### Response Format

**Single Entry:**

```json
{
  "entry": {
    "uid": "entry_uid",
    "title": "Entry Title",
    "url": "/entry-url",
    "publish_details": {
      "environment": "production",
      "locale": "en-us",
      "time": "2024-01-01T00:00:00.000Z"
    },
    "created_at": "2024-01-01T00:00:00.000Z",
    "updated_at": "2024-01-01T00:00:00.000Z",
    "created_by": "user_uid",
    "updated_by": "user_uid",
    "ACL": {},
    "_version": 1,
    "field_name": "field_value"
  }
}
```

**Multiple Entries:**

```json
{
  "entries": [
    { "uid": "entry_1", ... },
    { "uid": "entry_2", ... }
  ],
  "count": 2
}
```

## GraphQL API

### Base URL

```
https://{region}-graphql.contentstack.com/graphql
```

**Preview Base URL**:

```
https://{region}-graphql-preview.contentstack.com/graphql
```

### Authentication

GraphQL requests use headers:

```http
api_key: YOUR_API_KEY
access_token: YOUR_DELIVERY_TOKEN
```

**For Preview**:

```http
api_key: YOUR_API_KEY
access_token: YOUR_PREVIEW_TOKEN
live_preview: PREVIEW_HASH
preview_token: YOUR_PREVIEW_TOKEN
include_applied_variants: "true"
```

### Basic Query Structure

```graphql
query {
  allContentstackPage(where: { url: "/" }) {
    nodes {
      uid
      title
      url
    }
  }
}
```

### Fetching Single Entry

```graphql
query GetEntry($uid: String!) {
  contentstackPage(uid: $uid) {
    uid
    title
    url
    description
    image {
      url
      title
    }
  }
}
```

**Variables:**

```json
{
  "uid": "entry_uid"
}
```

### Fetching Multiple Entries

```graphql
query GetEntries {
  allContentstackPage(
    where: { title: { regex: "tutorial", options: "i" } }
    skip: 0
    limit: 10
    sort: [{ published_date: ASC }]
  ) {
    nodes {
      uid
      title
      url
      published_date
    }
    total
  }
}
```

### Query Filters

GraphQL supports filtering via `where` clause:

```graphql
query {
  allContentstackBlogPost(
    where: {
      and: [
        { category: { in: ["tech", "design"] } }
        { published_date: { gte: "2024-01-01" } }
      ]
    }
  ) {
    nodes {
      title
      category
    }
  }
}
```

**Filter Operators:**

- `eq`: Equal to
- `ne`: Not equal to
- `gt`: Greater than
- `gte`: Greater than or equal to
- `lt`: Less than
- `lte`: Less than or equal to
- `in`: Matches any value in array
- `nin`: Matches none of the values
- `regex`: Regular expression match
- `exists`: Field exists
- `and`: Logical AND
- `or`: Logical OR

### Including References

References are automatically included when queried:

```graphql
query {
  contentstackBlogPost(uid: "entry_uid") {
    title
    author {
      uid
      title
      bio
    }
    category {
      uid
      title
    }
  }
}
```

### Fragments

Use fragments for reusable query parts:

```graphql
fragment BlogPostFields on ContentstackBlogPost {
  uid
  title
  url
  published_date
  excerpt
}

query {
  allContentstackBlogPost {
    nodes {
      ...BlogPostFields
      author {
        title
      }
    }
  }
}
```

### Assets

```graphql
query {
  contentstackAsset(uid: "asset_uid") {
    url
    title
    filename
    content_type
    file_size
    dimension {
      width
      height
    }
  }
}
```

### Search Assets

```graphql
query {
  allContentstackAsset(
    where: { title: { regex: "logo", options: "i" } }
    limit: 20
  ) {
    nodes {
      uid
      url
      title
    }
    total
  }
}
```

## Error Handling

### REST API Errors

**Status Codes:**

- `200`: Success
- `400`: Bad Request (invalid query)
- `401`: Unauthorized (invalid token)
- `404`: Not Found (entry/asset doesn't exist)
- `422`: Validation Error
- `500`: Server Error

**Error Response:**

```json
{
  "error_code": 141,
  "error_message": "Entry not found",
  "errors": {
    "entry_uid": ["Entry does not exist"]
  }
}
```

### GraphQL Errors

GraphQL returns `200` status with errors in response:

```json
{
  "data": null,
  "errors": [
    {
      "message": "Entry not found",
      "locations": [{ "line": 2, "column": 3 }],
      "path": ["contentstackPage"]
    }
  ]
}
```

## Rate Limiting

Both APIs have rate limits:

- **Delivery API**: Varies by plan (typically 1000-10000 requests/hour)
- **Preview API**: Lower limits for preview endpoints
- **Management API**: Different limits based on subscription

**Rate Limit Headers:**

```http
X-RateLimit-Limit: 10000
X-RateLimit-Remaining: 9950
X-RateLimit-Reset: 1640995200
```

## Best Practices

Based on [Contentstack's official API best practices](https://www.contentstack.com/docs/developers/apis/content-delivery-api), follow these guidelines for optimal API usage:

### 1. Use Appropriate Tokens

- **Delivery Token**: For production applications
- **Preview Token**: For Live Preview and development
- **Management Token**: Only for backend operations

**Security Note**: Never store API keys and tokens in your code or version control. Use environment variables and secure storage.

### 2. Limit Response Payload

**CRITICAL**: Keep response payloads under **5 MB** to avoid performance issues and infrastructure load.

**Best Practices:**

- Validate and filter data before returning to client
- Remove unnecessary fields from responses
- Use projection queries to limit fields returned

### 3. Optimize Includes and Reference Depth

**Limit Includes:**

- Keep total number of includes to **10 or fewer**
- Avoid deep nesting of references
- Only include references you actually need

**Example - Too Many Includes (Avoid):**

```http
GET /v3/content_types/blog_post/entries/entry_uid?include[]=author&include[]=category&include[]=tags&include[]=related_posts&include[]=comments&include[]=likes&include[]=shares&include[]=metadata&include[]=translations&include[]=variants
```

**Example - Optimized Includes:**

```http
GET /v3/content_types/blog_post/entries/entry_uid?include[]=author&include[]=category
```

**Split Large Includes:**
If you need many references, split them into multiple calls:

```typescript
// Instead of one call with 10 includes
const entry = await fetchEntryWithManyIncludes(uid);

// Split into multiple calls
const entry = await fetchEntry(uid);
const author = await fetchAuthor(entry.author_uid);
const category = await fetchCategory(entry.category_uid);
// ... fetch other references as needed
```

### 4. Use Projection Queries

Use `only` and `except` parameters to limit response size:

**REST - Only Specific Fields:**

```http
GET /v3/content_types/blog_post/entries?only[BASE][]=title&only[BASE][]=url&only[BASE][]=published_date
```

**REST - Exclude Fields:**

```http
GET /v3/content_types/blog_post/entries?except[BASE][]=internal_notes&except[BASE][]=draft_content
```

**GraphQL - Query Only Needed Fields:**

```graphql
query {
  allContentstackBlogPost {
    nodes {
      title
      url
      published_date
      # Only request fields you actually use
    }
  }
}
```

### 5. Implement Pagination

Always implement pagination for large datasets:

**REST:**

```http
GET /v3/content_types/blog_post/entries?skip=0&limit=10&include_count=true
```

**GraphQL:**

```graphql
query ($skip: Int!, $limit: Int!) {
  allContentstackBlogPost(skip: $skip, limit: $limit) {
    nodes {
      title
    }
    total
  }
}
```

**Best Practices:**

- Use reasonable page sizes (10-50 items)
- Always include total count for pagination UI
- Implement infinite scroll or page-based navigation

### 6. Avoid Deep Reference Nesting

**Avoid:**

```http
GET /v3/content_types/blog_post/entries/entry_uid?include[]=author.company.employees.department.manager
```

**Prefer:**

```http
GET /v3/content_types/blog_post/entries/entry_uid?include[]=author
```

Then fetch nested references separately if needed.

### 7. Use Modular Blocks Instead of Multiple References

When building pages with multiple content blocks, use **Modular Blocks** instead of multiple content type references:

**Why Modular Blocks:**

- Reduces number of includes needed
- All blocks fetched in single call
- More efficient than multiple references
- Better performance

**Example Structure:**

```typescript
// Content Type: Page
// Field: blocks (Modular Block)
// Blocks: HeroBlock, ContentBlock, CTABlock, GalleryBlock

// Single call fetches all blocks
const page = await stack.contentType("page").entry("page_uid").fetch();
// All blocks are included automatically, no includes needed
```

### 8. Implement Caching Strategy

**CDN Caching:**

- Contentstack automatically caches API responses via CDN
- Cache is invalidated when content is published
- Leverage CDN for better performance

**Application-Level Caching:**

```typescript
// Cache frequently-used data
const cache = new Map();

async function getCachedEntry(uid: string) {
  const cacheKey = `entry_${uid}`;

  if (cache.has(cacheKey)) {
    return cache.get(cacheKey);
  }

  const entry = await stack.contentType("blog_post").entry(uid).fetch();
  cache.set(cacheKey, entry);

  // Set TTL (time to live)
  setTimeout(() => cache.delete(cacheKey), 3600000); // 1 hour

  return entry;
}
```

**Cache Frequently-Used Data:**

- User groups, titles, metadata
- FAQ sections
- Navigation menus
- Content that doesn't change often

### 9. Implement Lazy Loading

Load content progressively instead of all at once:

**Example - Lazy Load Sections:**

```typescript
// Load critical content first
const hero = await fetchHero();
const mainContent = await fetchMainContent();

// Load secondary content later
setTimeout(async () => {
  const sidebar = await fetchSidebar();
  const footer = await fetchFooter();
}, 100);
```

**Framework-Specific:**

- **React**: Use `React.lazy()` and `Suspense`
- **Vue**: Use dynamic imports
- **Next.js**: Use dynamic imports with `ssr: false`

### 10. Optimize Your Code

**Eliminate Redundancies:**

- Remove duplicate API calls
- Check if data is already fetched before requesting
- Avoid making queries unique with random numbers/timestamps
- Cache responses in your application

**Example - Avoid Duplicate Calls:**

```typescript
// BAD: Fetches same data multiple times
function renderPage() {
  const entry = await fetchEntry(uid);
  const author = await fetchAuthor(entry.author_uid);
  const entry2 = await fetchEntry(uid); // Duplicate!
}

// GOOD: Fetch once, reuse
function renderPage() {
  const entry = await fetchEntry(uid);
  const author = await fetchAuthor(entry.author_uid);
  // Reuse entry object
}
```

### 11. Error Handling

Always handle API errors gracefully:

```javascript
try {
  const response = await fetch(apiUrl, options);
  if (!response.ok) {
    const error = await response.json();
    // Handle error appropriately
    console.error("API Error:", error.error_message);
    return null;
  }
  const data = await response.json();
  return data;
} catch (error) {
  // Handle network error
  console.error("Network Error:", error);
  return null;
}
```

### 12. Use SDK When Possible

Prefer using the official SDK (`@contentstack/delivery-sdk`) over direct API calls:

- Built-in error handling
- Type safety (TypeScript)
- Easier authentication
- Query builder helpers
- Live Preview support
- Automatic request optimization

### 13. Batch Operations When Needed

If you need multiple includes that exceed limits, batch them:

```typescript
// Split includes into batches
async function fetchEntryWithBatchedIncludes(uid: string) {
  // Batch 1: Core references
  const entry = await stack
    .contentType("blog_post")
    .entry(uid)
    .includeReference(["author", "category"])
    .only(["title", "url", "author", "category"])
    .fetch();

  // Batch 2: Additional references
  const tags = await stack
    .contentType("tag")
    .entry()
    .query()
    .where("entries", QueryOperation.IN, [uid])
    .find();

  // Merge results
  return { ...entry, tags: tags.entries };
}
```

### 14. Monitor API Usage

- Check rate limit headers in responses
- Implement retry logic with exponential backoff
- Monitor response times and payload sizes
- Set up alerts for API errors

### Summary Checklist

When making API calls, ensure:

- ✅ Response payload < 5 MB
- ✅ Total includes ≤ 10
- ✅ Using projection queries (`only`/`except`)
- ✅ Implementing pagination
- ✅ Avoiding deep reference nesting
- ✅ Caching frequently-used data
- ✅ Using lazy loading for non-critical content
- ✅ Eliminating duplicate calls
- ✅ Handling errors gracefully
- ✅ Using SDK when possible

## Regional Considerations

Always specify the correct region in your API calls:

- Use region-specific endpoints
- Ensure tokens are valid for the region
- Consider latency when choosing region

## Next Steps

- Learn about [SDK Functionalities](02-sdk-functionalities.md)
- Learn about [Live Preview Guide](03-live-preview-guide.md)
