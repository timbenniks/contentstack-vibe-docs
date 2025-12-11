# Contentstack OAuth with Auth.js v5 (NextAuth)

A clean, production-ready integration pattern for Contentstack OAuth with Auth.js v5 in Next.js applications.

---

## Quick Reference (for AI Agents)

**Files to create:**

1. `lib/auth.ts` — Auth.js configuration with Contentstack provider
2. `app/api/auth/[...nextauth]/route.ts` — Route handler (2 lines)

**Required packages:**

```bash
npm install next-auth@5 @timbenniks/contentstack-endpoints
```

**Required environment variables:**

- `AUTH_SECRET` — Random string for JWT encryption
- `CONTENTSTACK_REGION` — Region code (na, eu, azure-na, azure-eu, gcp-na, gcp-eu)
- `CONTENTSTACK_APP_ID` — App UID from Developer Hub
- `CONTENTSTACK_CLIENT_ID` — OAuth client ID
- `CONTENTSTACK_CLIENT_SECRET` — OAuth client secret
- `NEXTAUTH_URL` — App URL for callbacks

**Contentstack setup:**

- Redirect URL: `{NEXTAUTH_URL}/api/auth/callback/contentstack`
- Required scope: `user:read`

**Critical implementation details:**

- Authorization URL: `{appUrl}/apps/{APP_ID}/authorize` — NOT `#!/apps/...`
- Token URL: `{appUrl}/apps-api/token`
- Userinfo URL: `{apiUrl}/v3/user` — response is nested under `user` key
- Use `checks: ["state"]` (or `["pkce", "state"]` for PKCE)
- Always use `session: { strategy: "jwt" }`

---

## Prerequisites

- Next.js 14+ with App Router
- Auth.js v5 (`next-auth@5`)
- A Contentstack app with OAuth enabled in Developer Hub

## Environment Variables

```env
# Auth.js secret (generate with: openssl rand -base64 32)
AUTH_SECRET="your-secret"

# Contentstack OAuth (from Developer Hub > Your App > OAuth)
CONTENTSTACK_REGION="eu"          # na, eu, azure-na, azure-eu, gcp-na, gcp-eu
CONTENTSTACK_APP_ID="your-app-uid"
CONTENTSTACK_CLIENT_ID="your-client-id"
CONTENTSTACK_CLIENT_SECRET="your-client-secret"

# App URL (for OAuth callbacks)
NEXTAUTH_URL="http://localhost:3000"
```

## Contentstack App Configuration

In Contentstack Developer Hub → Your App → OAuth:

1. **Redirect URL**: `{NEXTAUTH_URL}/api/auth/callback/contentstack`
2. **User Token Scopes**: `user:read`

## Installation

```bash
npm install next-auth@5 @timbenniks/contentstack-endpoints
```

---

## Option 1: Cookie-Based Sessions (Recommended)

The simplest approach — no database required. User sessions are stored as encrypted JWTs in HTTP-only cookies.

### Auth Configuration

**File: `lib/auth.ts`**

```typescript
import NextAuth from "next-auth";
import { getContentstackEndpoints } from "@timbenniks/contentstack-endpoints";

const region = process.env.CONTENTSTACK_REGION ?? "na";
const endpoints = getContentstackEndpoints(region);
const appUrl = endpoints.application!;
const apiUrl = endpoints.contentManagement!;

export const { handlers, auth, signIn, signOut } = NextAuth({
  session: { strategy: "jwt" },
  pages: { signIn: "/login", error: "/login" },

  providers: [
    {
      id: "contentstack",
      name: "Contentstack",
      type: "oauth",
      checks: ["state"],
      authorization: {
        url: `${appUrl}/apps/${process.env.CONTENTSTACK_APP_ID}/authorize`,
        params: { response_type: "code", scope: "user:read" },
      },
      token: `${appUrl}/apps-api/token`,
      userinfo: {
        url: `${apiUrl}/v3/user`,
        async request({ tokens }: { tokens: { access_token: string } }) {
          const res = await fetch(`${apiUrl}/v3/user`, {
            headers: { Authorization: `Bearer ${tokens.access_token}` },
          });
          const { user } = await res.json();
          return user;
        },
      },
      profile: (p) => ({
        id: p.uid,
        name:
          [p.first_name, p.last_name].filter(Boolean).join(" ") ||
          p.username ||
          p.email,
        email: p.email,
        image: p.profile_image ?? null,
      }),
      clientId: process.env.CONTENTSTACK_CLIENT_ID,
      clientSecret: process.env.CONTENTSTACK_CLIENT_SECRET,
    },
  ],

  callbacks: {
    session: ({ session, token }) => {
      if (token.sub) session.user.id = token.sub;
      return session;
    },
    jwt: ({ token, user }) => {
      if (user) token.sub = user.id;
      return token;
    },
  },
});

export const getSession = () => auth();
```

### Route Handler

**File: `app/api/auth/[...nextauth]/route.ts`**

```typescript
import { handlers } from "@/lib/auth";
export const { GET, POST } = handlers;
```

That's it! No database setup required.

---

## Option 2: Database-Persisted Sessions

Use this approach if you need to:

- Store additional user data
- Track user activity
- Link users to other models in your database
- Revoke sessions server-side

### Additional Installation

```bash
npm install @auth/prisma-adapter @prisma/client prisma
```

### Prisma Schema

**File: `prisma/schema.prisma`**

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"  // or "postgresql", "mysql"
  url      = env("DATABASE_URL")
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?
  user              User    @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model User {
  id            String    @id @default(cuid())
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  accounts      Account[]
  sessions      Session[]
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}
```

### Database Client

**File: `lib/db.ts`**

```typescript
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const db = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = db;
```

### Auth Configuration with Prisma

**File: `lib/auth.ts`** (updated with Prisma adapter)

```typescript
import NextAuth from "next-auth";
import { PrismaAdapter } from "@auth/prisma-adapter";
import { getContentstackEndpoints } from "@timbenniks/contentstack-endpoints";
import { db } from "./db";

const region = process.env.CONTENTSTACK_REGION ?? "na";
const endpoints = getContentstackEndpoints(region);
const appUrl = endpoints.application!;
const apiUrl = endpoints.contentManagement!;

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(db), // Add this line
  session: { strategy: "jwt" },
  pages: { signIn: "/login", error: "/login" },

  providers: [
    {
      id: "contentstack",
      name: "Contentstack",
      type: "oauth",
      checks: ["state"],
      authorization: {
        url: `${appUrl}/apps/${process.env.CONTENTSTACK_APP_ID}/authorize`,
        params: { response_type: "code", scope: "user:read" },
      },
      token: `${appUrl}/apps-api/token`,
      userinfo: {
        url: `${apiUrl}/v3/user`,
        async request({ tokens }: { tokens: { access_token: string } }) {
          const res = await fetch(`${apiUrl}/v3/user`, {
            headers: { Authorization: `Bearer ${tokens.access_token}` },
          });
          const { user } = await res.json();
          return user;
        },
      },
      profile: (p) => ({
        id: p.uid,
        name:
          [p.first_name, p.last_name].filter(Boolean).join(" ") ||
          p.username ||
          p.email,
        email: p.email,
        image: p.profile_image ?? null,
      }),
      clientId: process.env.CONTENTSTACK_CLIENT_ID,
      clientSecret: process.env.CONTENTSTACK_CLIENT_SECRET,
    },
  ],

  callbacks: {
    session: ({ session, token }) => {
      if (token.sub) session.user.id = token.sub;
      return session;
    },
    jwt: ({ token, user }) => {
      if (user) token.sub = user.id;
      return token;
    },
  },
});

export const getSession = () => auth();
```

### Initialize Database

```bash
npx prisma db push
```

---

## Usage

### Server Component

```typescript
import { auth } from "@/lib/auth";
import { redirect } from "next/navigation";

export default async function ProtectedPage() {
  const session = await auth();
  if (!session) redirect("/login");

  return <div>Welcome, {session.user.name}</div>;
}
```

### Client Component (Sign In)

```typescript
"use client";
import { signIn } from "next-auth/react";

export function SignInButton() {
  return (
    <button
      onClick={() => signIn("contentstack", { callbackUrl: "/dashboard" })}
    >
      Sign in with Contentstack
    </button>
  );
}
```

### Server Action (Sign In)

```typescript
import { signIn } from "@/lib/auth";

export default function LoginPage() {
  return (
    <form
      action={async () => {
        "use server";
        await signIn("contentstack", { redirectTo: "/dashboard" });
      }}
    >
      <button type="submit">Sign in with Contentstack</button>
    </form>
  );
}
```

### Middleware (Route Protection)

**File: `middleware.ts`** (project root)

```typescript
export { auth as middleware } from "@/lib/auth";

export const config = {
  matcher: ["/dashboard/:path*", "/app/:path*"],
};
```

---

## URL Construction (Important!)

The authorization URL format is **critical** and easy to get wrong.

### Correct Format

```
Authorization: {appUrl}/apps/{APP_ID}/authorize
Token:         {appUrl}/apps-api/token
Userinfo:      {apiUrl}/v3/user
```

### Common Mistakes

```
❌ {appUrl}/#!/apps/{APP_ID}/authorize     # Hash routing doesn't work
❌ {appUrl}/apps/authorize                  # Missing APP_ID
❌ {appUrl}/apps/{CLIENT_ID}/authorize      # Wrong ID (use APP_ID, not CLIENT_ID)
```

### URL Resolution with @timbenniks/contentstack-endpoints

```typescript
import { getContentstackEndpoints } from "@timbenniks/contentstack-endpoints";

const endpoints = getContentstackEndpoints("eu");

// endpoints.application  → "https://eu-app.contentstack.com"
// endpoints.contentManagement → "https://eu-api.contentstack.com"
```

### Full URL Examples (EU Region)

| Endpoint      | URL                                                                       |
| ------------- | ------------------------------------------------------------------------- |
| Authorization | `https://eu-app.contentstack.com/apps/693970588f70cef8ad0a4578/authorize` |
| Token         | `https://eu-app.contentstack.com/apps-api/token`                          |
| Userinfo      | `https://eu-api.contentstack.com/v3/user`                                 |
| Callback      | `http://localhost:3000/api/auth/callback/contentstack`                    |

---

## Key Points

| Aspect              | Detail                                                         |
| ------------------- | -------------------------------------------------------------- |
| **PKCE support**    | Contentstack supports PKCE — see note below                    |
| **Custom userinfo** | Contentstack returns data nested under `user` key              |
| **Region URLs**     | Use `@timbenniks/contentstack-endpoints` for correct endpoints |
| **JWT required**    | Always use `session: { strategy: "jwt" }`                      |

### PKCE (Proof Key for Code Exchange)

Contentstack supports [PKCE](https://www.contentstack.com/docs/developers/developer-hub/pkce-for-contentstack-oauth) for enhanced security. This example uses `checks: ["state"]` for simplicity, but you can enable PKCE:

```typescript
{
  id: "contentstack",
  type: "oauth",
  checks: ["pkce", "state"],  // Enable PKCE
  // ... rest of config
}
```

PKCE is recommended for:

- Public clients (SPAs, mobile apps)
- Enhanced protection against authorization code interception attacks

For server-side Next.js apps with a client secret, `checks: ["state"]` is sufficient.

## Contentstack Profile Fields

The userinfo endpoint returns:

```typescript
interface ContentstackUser {
  uid: string;
  email: string;
  first_name?: string;
  last_name?: string;
  username?: string;
  profile_image?: string;
}
```

## References

- [Contentstack OAuth Documentation](https://www.contentstack.com/docs/developers/developer-hub/contentstack-oauth)
- [Auth.js Documentation](https://authjs.dev)
- [@timbenniks/contentstack-endpoints](https://www.npmjs.com/package/@timbenniks/contentstack-endpoints)
