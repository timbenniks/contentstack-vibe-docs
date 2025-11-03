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

### 1. Use Appropriate Tokens

- **Delivery Token**: For production applications
- **Preview Token**: For Live Preview and development
- **Management Token**: Only for backend operations

### 2. Include Only What You Need

**REST:**
```http
GET /v3/content_types/blog_post/entries?only[BASE][]=title&only[BASE][]=url
```

**GraphQL:**
```graphql
query {
  allContentstackBlogPost {
    nodes {
      title
      url
    }
  }
}
```

### 3. Implement Caching

- Cache responses when appropriate
- Use CDN caching for delivery API
- Implement client-side caching for better performance

### 4. Handle Pagination

Always implement pagination for large datasets:

```graphql
query($skip: Int!, $limit: Int!) {
  allContentstackBlogPost(skip: $skip, limit: $limit) {
    nodes { title }
    total
  }
}
```

### 5. Error Handling

Always handle API errors gracefully:

```javascript
try {
  const response = await fetch(apiUrl, options);
  if (!response.ok) {
    const error = await response.json();
    // Handle error
  }
  const data = await response.json();
} catch (error) {
  // Handle network error
}
```

### 6. Use SDK When Possible

Prefer using the official SDK (`@contentstack/delivery-sdk`) over direct API calls:

- Built-in error handling
- Type safety (TypeScript)
- Easier authentication
- Query builder helpers
- Live Preview support

## Regional Considerations

Always specify the correct region in your API calls:

- Use region-specific endpoints
- Ensure tokens are valid for the region
- Consider latency when choosing region

## Next Steps

- Learn about [SDK Functionalities](02-sdk-functionalities.md)
- Learn about [Live Preview Setup](03-live-preview-base-concepts.md)
- Learn about [Personalization Setup](04-personalization-setup.md)

