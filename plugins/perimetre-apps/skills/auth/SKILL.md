---
name: auth
description: >
  Périmètre authentication patterns for Next.js/Node.js apps. Default to Logto
  for new projects — it is the recommended provider. Fall back to other OIDC
  providers, Auth.js, or Clerk only when there is a clear reason. Use
  username+password or direct Google/Microsoft auth only for specific needs.
  Covers session management, middleware route protection, tRPC auth context, and
  service layer integration. Triggers on: authentication, auth, login, logout,
  sign in, sign out, session, Logto, @logto/next, OIDC, OAuth, JWT, access
  token, refresh token, authedUserProcedure, getUser, isAuthenticated,
  withLogto, LogtoClient, LogtoNextClient, Auth.js, NextAuth, Clerk, social
  login, SSO, MFA, protected route, middleware auth, useUser, getLogtoContext,
  handleSignIn, handleSignOut, handleCallback, user context, user session.
---
# Authentication Guide

Périmètre apps default to **Logto** for authentication. It is open-source,
OIDC-compliant, self-hostable, ships with MFA and social connectors out of the
box, and has a first-class Next.js SDK. Only deviate from it when there is a
clear, project-specific reason.

## Provider Decision Table

| Situation | Recommended provider |
|-----------|----------------------|
| New project, no special constraint | **Logto** (default) |
| Team already on Auth.js / NextAuth | Auth.js — acceptable, no migration needed |
| Product needs hosted auth + team management UI | Clerk |
| Regulatory requirement for internal username+password | Custom credentials (Auth.js CredentialsProvider) |
| B2B SaaS with enterprise SSO requirement only | Direct SAML/OIDC connector in Logto |
| Prototype / quick demo | Auth.js with a single social provider |

> **Note:** Direct Google/Microsoft OAuth without an auth layer (i.e. rolling
> your own token exchange) is strongly discouraged. Use Logto's social
> connectors or Auth.js instead — they handle token refresh, PKCE, and session
> security correctly.

---

## Logto (Default)

### Setup

```bash
pnpm add @logto/next
```

```typescript
// src/server/lib/logto.ts
import LogtoClient from '@logto/next';

export const logtoClient = new LogtoClient({
  endpoint: process.env.LOGTO_ENDPOINT!,       // e.g. https://auth.yourapp.com
  appId: process.env.LOGTO_APP_ID!,
  appSecret: process.env.LOGTO_APP_SECRET!,
  baseUrl: process.env.NEXT_PUBLIC_BASE_URL!,  // e.g. https://yourapp.com
  cookieSecret: process.env.LOGTO_COOKIE_SECRET!,
  cookieSecure: process.env.NODE_ENV === 'production',
  // Optional: request extra scopes
  scopes: ['openid', 'profile', 'email', 'offline_access'],
  // Optional: resource indicators for API access tokens
  resources: [process.env.LOGTO_API_RESOURCE!],
});
```

Required env vars:

```bash
LOGTO_ENDPOINT=https://auth.yourapp.com
LOGTO_APP_ID=your-app-id
LOGTO_APP_SECRET=your-app-secret
LOGTO_COOKIE_SECRET=random-32-char-string   # openssl rand -base64 32
NEXT_PUBLIC_BASE_URL=https://yourapp.com
LOGTO_API_RESOURCE=https://yourapp.com/api  # if using API access tokens
```

### Auth Route Handlers

```typescript
// src/app/api/logto/[action]/route.ts
import { logtoClient } from '@/server/lib/logto';
import { NextRequest } from 'next/server';

export async function GET(
  req: NextRequest,
  { params }: { params: { action: string } }
) {
  const { action } = await params;

  switch (action) {
    case 'sign-in':
      return logtoClient.handleSignIn()(req);

    case 'sign-out':
      return logtoClient.handleSignOut()(req);

    case 'callback':
      return logtoClient.handleSignInCallback()(req);

    default:
      return new Response('Not found', { status: 404 });
  }
}
```

### Reading the User in Server Components

```typescript
// src/app/dashboard/page.tsx
import { logtoClient } from '@/server/lib/logto';
import { redirect } from 'next/navigation';

export default async function DashboardPage() {
  const { isAuthenticated, claims } = await logtoClient.getLogtoContext();

  if (!isAuthenticated) redirect('/api/logto/sign-in');

  return <div>Welcome, {claims?.name}</div>;
}
```

### Middleware — Protecting Routes

```typescript
// src/middleware.ts
import { logtoClient } from '@/server/lib/logto';
import { NextRequest } from 'next/server';

export async function middleware(req: NextRequest) {
  const { isAuthenticated } = await logtoClient.getLogtoContext({ req });

  if (!isAuthenticated) {
    const signInUrl = new URL('/api/logto/sign-in', req.url);
    signInUrl.searchParams.set('redirect_uri', req.url);
    return Response.redirect(signInUrl);
  }
}

export const config = {
  matcher: ['/dashboard/:path*', '/account/:path*'],
};
```

### tRPC Auth Middleware

Drop-in replacement for the `authedUserProcedure` shown in the tRPC skill:

```typescript
// src/server/routers/middlewares/auth.ts
import { UnauthorizedError } from '@/shared/exceptions';
import { middleware } from '..';
import { logtoClient } from '@/server/lib/logto';
import { headers } from 'next/headers';

export const authedUserProcedure = middleware(async ({ next, ctx }) => {
  // Verify the session via Logto
  const { isAuthenticated, claims } = await logtoClient.getLogtoContext();

  if (!isAuthenticated || !claims?.sub) {
    throw new UnauthorizedError('User is not authenticated');
  }

  return next({
    ctx: {
      ...ctx,
      user: {
        id: claims.sub,
        email: claims.email,
        name: claims.name,
      },
    },
  });
});
```

Usage in a router is identical to the tRPC skill pattern:

```typescript
create: procedure
  .use(authedUserProcedure)
  .input(z.object({ title: z.string() }))
  .mutation(async ({ input, ctx }) => {
    // ctx.user is typed and guaranteed to exist
    const result = await postsService({}).create({
      ...input,
      userId: ctx.user.id,
    });
    if (!('ok' in result)) throw result;
    return result.post;
  }),
```

### Client Component — Current User

```typescript
'use client';
import { useUser } from '@logto/next/client';

export function UserMenu() {
  const { user, isLoading } = useUser();

  if (isLoading) return <Spinner />;
  if (!user) return <a href="/api/logto/sign-in">Sign in</a>;

  return (
    <div>
      <span>{user.name}</span>
      <a href="/api/logto/sign-out">Sign out</a>
    </div>
  );
}
```

Wrap the app (or layout) with the Logto provider:

```typescript
// src/app/layout.tsx
import { LogtoProvider } from '@logto/next/client';
import { logtoConfig } from '@/server/lib/logto';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <LogtoProvider config={logtoConfig}>
          {children}
        </LogtoProvider>
      </body>
    </html>
  );
}
```

### API Access Tokens (for backend-to-backend or fetch calls)

```typescript
// When calling a protected API from a Server Component or Route Handler
const { accessToken } = await logtoClient.getLogtoContext();

const response = await fetch('https://api.yourapp.com/data', {
  headers: { Authorization: `Bearer ${accessToken}` },
});
```

Logto handles token refresh automatically via `offline_access` scope.

---

## Auth.js / NextAuth (Acceptable Alternative)

Use when the project already has Auth.js in place, or when a simpler setup
with a single social provider is sufficient.

```typescript
// src/app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import Google from 'next-auth/providers/google';

const handler = NextAuth({
  providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
});

export { handler as GET, handler as POST };
```

tRPC middleware with Auth.js:

```typescript
import { getServerSession } from 'next-auth';
import { authOptions } from '@/app/api/auth/[...nextauth]/route';

export const authedUserProcedure = middleware(async ({ next, ctx }) => {
  const session = await getServerSession(authOptions);

  if (!session?.user) throw new UnauthorizedError('User is not authenticated');

  return next({ ctx: { ...ctx, user: session.user } });
});
```

---

## Username + Password (Specific Needs Only)

Use **only** when there is a clear reason: internal admin tool, regulatory
requirement, or a project that cannot use an external identity provider.

With Logto: enable the "Username & Password" connector in the Logto dashboard —
no code changes needed.

With Auth.js CredentialsProvider (last resort):

```typescript
import Credentials from 'next-auth/providers/credentials';
import { verifyPassword } from '@/server/lib/auth';

Credentials({
  credentials: {
    email: { label: 'Email', type: 'email' },
    password: { label: 'Password', type: 'password' },
  },
  async authorize(credentials) {
    if (!credentials?.email || !credentials?.password) return null;
    const user = await db.users.findByEmail(credentials.email);
    if (!user) return null;
    const valid = await verifyPassword(credentials.password, user.passwordHash);
    return valid ? { id: user.id, email: user.email, name: user.name } : null;
  },
});
```

> **Security reminder:** Never store plain-text passwords. Use bcrypt or Argon2.
> Rate-limit the credentials endpoint. Enforce HTTPS.

---

## Security Checklist

- [ ] All auth cookies are `httpOnly`, `secure`, and `sameSite: 'lax'`
- [ ] Cookie secret is a random 32+ char string (not reused across envs)
- [ ] PKCE is enabled (Logto and Auth.js do this automatically)
- [ ] Token refresh is handled automatically (Logto with `offline_access`, Auth.js `jwt` strategy)
- [ ] Protected routes enforce auth in **middleware** — not only in page components
- [ ] `authedUserProcedure` middleware is applied to every tRPC mutation that writes data
- [ ] Logto endpoint / Auth.js callback URL is allowlisted (not wildcard)
- [ ] User ID from the auth context is used as the source of truth — never trust user-supplied IDs for ownership checks

---

## Quick Reference

```typescript
// Logto: read user in Server Component
const { isAuthenticated, claims } = await logtoClient.getLogtoContext();

// Logto: tRPC middleware → ctx.user
export const authedUserProcedure = middleware(async ({ next, ctx }) => {
  const { isAuthenticated, claims } = await logtoClient.getLogtoContext();
  if (!isAuthenticated) throw new UnauthorizedError();
  return next({ ctx: { ...ctx, user: { id: claims!.sub, email: claims!.email } } });
});

// Auth.js: tRPC middleware → ctx.user
export const authedUserProcedure = middleware(async ({ next, ctx }) => {
  const session = await getServerSession(authOptions);
  if (!session?.user) throw new UnauthorizedError();
  return next({ ctx: { ...ctx, user: session.user } });
});

// Protecting a route in middleware.ts
if (!isAuthenticated) return Response.redirect(new URL('/api/logto/sign-in', req.url));

// Sign-in / sign-out links
<a href="/api/logto/sign-in">Sign in</a>
<a href="/api/logto/sign-out">Sign out</a>
```
