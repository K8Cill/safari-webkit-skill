# Next.js + Safari: Known Issues & Solutions

This file covers issues specific to Next.js applications running in Safari/WebKit.

---

## Table of Contents

1. [Hydration Issues](#1-hydration-issues)
2. [App Router Specific](#2-app-router-specific)
3. [Pages Router Specific](#3-pages-router-specific)
4. [Fetch & Data Fetching](#4-fetch--data-fetching)
5. [CSS Modules & Tailwind](#5-css-modules--tailwind)
6. [Image Component](#6-image-component)
7. [Authentication & Cookies](#7-authentication--cookies)
8. [Font Optimization](#8-font-optimization)
9. [Deployment Considerations](#9-deployment-considerations)

---

## 1. Hydration Issues

### 1.1 Date rendering mismatch

**Root cause**: Server renders with UTC, Safari may parse the same date string differently.

```typescript
// ❌ Causes hydration mismatch
export default function PostDate({ date }: { date: string }) {
  return <time>{new Date(date).toLocaleDateString()}</time>;
  // Server: en-US locale formatting
  // Safari client: may use device locale, producing different string
}

// ✅ Suppress hydration mismatch for client-only formatting
'use client';
import { useState, useEffect } from 'react';

export function PostDate({ date }: { date: string }) {
  const [formatted, setFormatted] = useState('');
  
  useEffect(() => {
    setFormatted(new Date(date).toLocaleDateString());
  }, [date]);
  
  // During SSR/first render: empty (avoids mismatch)
  // After hydration: shows formatted date
  return <time dateTime={date}>{formatted}</time>;
}

// ✅ Or use suppressHydrationWarning for known safe mismatches
export function PostDate({ date }: { date: string }) {
  return (
    <time dateTime={date} suppressHydrationWarning>
      {new Date(date).toLocaleDateString()}
    </time>
  );
}
```

### 1.2 `useLayoutEffect` in shared components

```typescript
// ❌ Warning + potential mismatch
import { useLayoutEffect } from 'react';

function Tooltip({ children }) {
  useLayoutEffect(() => {
    // measure DOM
  }, []);
}

// ✅ Isomorphic hook — use everywhere
// lib/use-isomorphic-layout-effect.ts
import { useEffect, useLayoutEffect } from 'react';

export const useIsomorphicLayoutEffect =
  typeof window !== 'undefined' ? useLayoutEffect : useEffect;
```

### 1.3 Browser-only APIs accessed at render time

```typescript
// ❌ Breaks SSR, causes hydration mismatch
function Component() {
  const isMac = navigator.platform.includes('Mac'); // crashes on server
  return <div>{isMac ? 'macOS' : 'other'}</div>;
}

// ✅ Defer to client only
'use client';
function Component() {
  const [isMac, setIsMac] = useState(false);
  
  useEffect(() => {
    setIsMac(navigator.platform.includes('Mac'));
  }, []);
  
  return <div>{isMac ? 'macOS' : 'other'}</div>;
}

// ✅ Or check at usage point
function Component() {
  const isMac = typeof navigator !== 'undefined' 
    && navigator.platform.includes('Mac');
  return <div>{isMac ? 'macOS' : 'other'}</div>;
}
```

### 1.4 Conditional rendering based on `window`

```typescript
// ❌ Hydration mismatch — server doesn't have window
function Component() {
  if (typeof window === 'undefined') return null; // wrong place!
  return <ActualContent />;
}

// ✅ Use mounted state pattern
'use client';
function Component() {
  const [mounted, setMounted] = useState(false);
  
  useEffect(() => {
    setMounted(true);
  }, []);
  
  if (!mounted) return null; // or a skeleton
  return <ActualContent />;
}
```

---

## 2. App Router Specific

### 2.1 Server Components + Date serialization

```typescript
// app/posts/page.tsx
import { db } from '@/lib/db';

// ❌ Date objects can't cross server/client boundary
async function PostsPage() {
  const posts = await db.getPosts(); // returns { id, title, createdAt: Date }
  return <PostList posts={posts} />; // Error: Date is not serializable
}

// ✅ Serialize before passing to Client Components
async function PostsPage() {
  const posts = await db.getPosts();
  const serialized = posts.map(post => ({
    ...post,
    createdAt: post.createdAt.toISOString(), // string ✓
  }));
  return <PostList posts={serialized} />;
}
```

### 2.2 Streaming + Safari

```typescript
// Safari supports HTTP streaming (used by Next.js Suspense streaming)
// But Safari may buffer more aggressively than Chrome

// ✅ For streaming responses, ensure proper headers:
// next.config.ts
export default {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
        ],
      },
    ];
  },
};
```

### 2.3 `cookies()` and `headers()` in Server Components

```typescript
// app/page.tsx
import { cookies } from 'next/headers';

export default async function Page() {
  const cookieStore = await cookies();
  const theme = cookieStore.get('theme');
  
  // Safari ITP: cookies set on different domain won't be sent
  // Always verify cookies are same-site or first-party
}
```

### 2.4 Parallel Routes and Interception — Safari quirk

Safari's back/forward navigation can behave differently with Next.js parallel routes.
If users report issues with browser back button in parallel routes, add:

```typescript
// In your layout, listen for popstate
'use client';
import { useEffect } from 'react';
import { useRouter } from 'next/navigation';

export function SafariHistoryFix() {
  const router = useRouter();
  
  useEffect(() => {
    const handlePopState = () => {
      router.refresh(); // force re-render on Safari back navigation
    };
    
    window.addEventListener('popstate', handlePopState);
    return () => window.removeEventListener('popstate', handlePopState);
  }, [router]);
  
  return null;
}
```

---

## 3. Pages Router Specific

### 3.1 `getServerSideProps` date serialization

```typescript
// ❌ Date objects not serializable as props
export const getServerSideProps = async () => {
  const post = await getPost();
  return { props: { post } }; // Error if post.date is a Date object
};

// ✅ Serialize dates
export const getServerSideProps = async () => {
  const post = await getPost();
  return {
    props: {
      post: {
        ...post,
        date: post.date.toISOString(),
      },
    },
  };
};
```

### 3.2 `_document.tsx` and Safari meta tags

```typescript
// pages/_document.tsx
import { Html, Head, Main, NextScript } from 'next/document';

export default function Document() {
  return (
    <Html lang="en">
      <Head>
        {/* Safari-specific meta tags */}
        <meta name="apple-mobile-web-app-capable" content="yes" />
        <meta name="apple-mobile-web-app-status-bar-style" content="default" />
        <meta name="apple-touch-fullscreen" content="yes" />
        {/* PWA icons for Safari */}
        <link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png" />
      </Head>
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  );
}
```

---

## 4. Fetch & Data Fetching

### 4.1 Next.js fetch cache + Safari

```typescript
// Next.js extended fetch with caching — Safari respects HTTP cache headers
// When using next/cache, be explicit:

// Server Component — revalidate every hour
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 3600 },
});

// No cache (always fresh)
const data = await fetch('https://api.example.com/data', {
  cache: 'no-store',
});

// Safari will respect Cache-Control on the response — make sure your API sets it correctly
```

### 4.2 Route Handlers (App Router) — Safari CORS

```typescript
// app/api/data/route.ts
import { NextResponse } from 'next/server';

export async function OPTIONS(request: Request) {
  // Safari sometimes sends preflight for same-origin requests in strict mode
  return new NextResponse(null, {
    status: 204,
    headers: {
      'Access-Control-Allow-Origin': process.env.NEXT_PUBLIC_APP_URL || '*',
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    },
  });
}

export async function GET(request: Request) {
  const data = await getData();
  return NextResponse.json(data);
}
```

---

## 5. CSS Modules & Tailwind

### 5.1 Autoprefixer is critical

```javascript
// postcss.config.js — ensure autoprefixer is installed and configured
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {}, // adds -webkit-backdrop-filter, etc.
  },
};
```

### 5.2 Tailwind classes with no Safari support

```typescript
// These Tailwind classes need manual Safari handling:

// backdrop-blur — needs -webkit- prefix
// Use: backdrop-blur-md + a CSS override in globals.css
// globals.css:
/*
.backdrop-blur-md {
  -webkit-backdrop-filter: blur(12px);
  backdrop-filter: blur(12px);
}
*/

// safe area utilities — Tailwind doesn't include these by default
// Add to tailwind.config.ts:
const config = {
  theme: {
    extend: {
      padding: {
        'safe-top': 'env(safe-area-inset-top)',
        'safe-bottom': 'env(safe-area-inset-bottom)',
        'safe-left': 'env(safe-area-inset-left)',
        'safe-right': 'env(safe-area-inset-right)',
      },
    },
  },
};
```

### 5.3 `min-h-screen` and Mobile Safari

```typescript
// ❌ min-h-screen = 100vh = broken on Mobile Safari (includes browser toolbar)
<div className="min-h-screen">

// ✅ Override in globals.css
/*
@layer utilities {
  .min-h-screen {
    min-height: 100vh; /* fallback */
    min-height: 100dvh; /* Safari 15.4+ */
  }
}
*/

// Or add custom Tailwind utility
// tailwind.config.ts:
const config = {
  theme: {
    extend: {
      minHeight: {
        'screen': ['100vh', '100dvh'], // Array = multiple values, last wins in modern CSS
        'screen-dynamic': '100dvh',
      },
    },
  },
};
```

---

## 6. Image Component

```typescript
// Next.js Image component is generally Safari-safe, but:

// ✅ Always provide width and height to prevent layout shift (CLS)
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1200}
  height={630}
  priority // LCP images
/>

// ✅ For fill mode, ensure parent has position: relative
<div className="relative aspect-video"> {/* aspect-video = aspect-ratio: 16/9 */}
  <Image
    src="/hero.jpg"
    alt="Hero"
    fill
    className="object-cover"
  />
</div>

// Format config (next.config.ts):
const nextConfig = {
  images: {
    formats: ['image/avif', 'image/webp'],
    // Next.js serves AVIF to Safari 16+ and WebP to Safari 14+
    // Falls back to original format for older Safari
  },
};
```

---

## 7. Authentication & Cookies

### 7.1 Safari ITP and third-party auth

```typescript
// Safari's Intelligent Tracking Prevention (ITP) blocks third-party cookies
// If using OAuth with a redirect flow, prefer same-site cookie auth

// next-auth configuration that works with Safari ITP:
// auth.ts
import NextAuth from 'next-auth';

export const { handlers, auth, signIn, signOut } = NextAuth({
  cookies: {
    sessionToken: {
      options: {
        httpOnly: true,
        sameSite: 'lax', // 'lax' works with Safari ITP; 'none' gets blocked
        path: '/',
        secure: process.env.NODE_ENV === 'production',
      },
    },
  },
});
```

### 7.2 Safari storage limits in private mode

```typescript
// localStorage throws in Safari private browsing
// sessionStorage has 5MB limit (lower in private mode)
// IndexedDB is blocked in private mode

// ✅ Feature-detect and fallback
function createStorage() {
  try {
    localStorage.setItem('__test', '1');
    localStorage.removeItem('__test');
    return localStorage;
  } catch {
    // Private mode or restricted — use in-memory fallback
    const store: Record<string, string> = {};
    return {
      getItem: (key: string) => store[key] ?? null,
      setItem: (key: string, value: string) => { store[key] = value; },
      removeItem: (key: string) => { delete store[key]; },
    };
  }
}

export const storage = createStorage();
```

---

## 8. Font Optimization

```typescript
// next/font works well with Safari, but note:

// ✅ next/font automatically generates @font-face with correct formats
// Safari supports WOFF2 (preferred) and WOFF
import { Inter } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap', // prevents FOIT (Flash of Invisible Text)
  // 'swap' shows fallback immediately, swaps when loaded
  // Safari respects font-display: swap
});

// ✅ For system fonts — Safari's system font is different from Chrome
// .font-sans in Tailwind includes -apple-system, BlinkMacSystemFont
// This maps to SF Pro on macOS/iOS Safari
```

---

## 9. Deployment Considerations

### 9.1 Vercel + Safari

```typescript
// Vercel edge functions run before reaching Safari
// Headers can be set to fix Safari-specific issues:

// next.config.ts
const nextConfig = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          // Allow SharedArrayBuffer (needed for some WebAssembly)
          { key: 'Cross-Origin-Embedder-Policy', value: 'require-corp' },
          { key: 'Cross-Origin-Opener-Policy', value: 'same-origin' },
          // Prevent Safari caching issues with API routes
          {
            key: 'Vary',
            value: 'Accept-Encoding, Accept',
          },
        ],
      },
      {
        source: '/api/(.*)',
        headers: [
          { key: 'Cache-Control', value: 'no-store, must-revalidate' },
        ],
      },
    ];
  },
};
```

### 9.2 Content Security Policy and Safari

```typescript
// Safari's CSP implementation is largely spec-compliant
// But note: 'strict-dynamic' support came in Safari 15.4

// For broad compatibility:
const cspHeader = `
  default-src 'self';
  script-src 'self' 'unsafe-inline' 'unsafe-eval';
  style-src 'self' 'unsafe-inline';
  img-src 'self' blob: data:;
  font-src 'self';
  connect-src 'self';
  media-src 'self';
`;
// Note: 'unsafe-inline' is a fallback — use nonces with Safari 15.4+ for better security
```
