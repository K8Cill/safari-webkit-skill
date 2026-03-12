# Safari/WebKit Bug Patterns Reference

A curated library of confirmed Safari/WebKit bugs with minimal reproducers and production-ready fixes.

---

## Table of Contents

1. [CSS Layout Bugs](#1-css-layout-bugs)
2. [Viewport & Safe Area](#2-viewport--safe-area)
3. [Scrolling Bugs](#3-scrolling-bugs)
4. [Animation & Transition Bugs](#4-animation--transition-bugs)
5. [JavaScript / API Bugs](#5-javascript--api-bugs)
6. [Date Parsing Bug](#6-date-parsing-bug)
7. [Form & Input Bugs](#7-form--input-bugs)
8. [Media Bugs](#8-media-bugs)
9. [Touch & Pointer Bugs](#9-touch--pointer-bugs)
10. [Fetch & Network Bugs](#10-fetch--network-bugs)
11. [Next.js Specific Bugs](#11-nextjs-specific-bugs)

---

## 1. CSS Layout Bugs

### 1.1 Flexbox `gap` not supported (Safari < 14.1)

**Symptom**: Gap between flex items is missing on older iOS.

```css
/* ❌ Broken in Safari < 14.1 */
.container {
  display: flex;
  gap: 16px;
}

/* ✅ Fixed — use margin fallback with feature query */
.container {
  display: flex;
}
.container > * + * {
  margin-left: 16px; /* fallback */
}
@supports (gap: 1px) {
  .container > * + * { margin-left: 0; }
  .container { gap: 16px; }
}
```

### 1.2 `position: sticky` inside `overflow: hidden` parent

**Symptom**: Sticky element doesn't stick — silently fails.

```css
/* ❌ Parent kills sticky */
.parent {
  overflow: hidden; /* or overflow: auto, scroll */
}
.child {
  position: sticky;
  top: 0;
}

/* ✅ Fixed — remove overflow from ancestor OR use clip instead of hidden */
.parent {
  overflow: clip; /* clip doesn't create a scroll container, sticky works */
}
```

### 1.3 `min-height: 100vh` on iOS

**Symptom**: Page taller than viewport, content behind Safari toolbar.

```css
/* ❌ Doesn't account for Safari's dynamic toolbar */
.hero {
  min-height: 100vh;
}

/* ✅ Fixed — use dvh with vh fallback */
.hero {
  min-height: 100vh; /* fallback for older browsers */
  min-height: 100dvh; /* dynamic viewport height — Safari 15.4+ */
}
```

### 1.4 CSS Grid subgrid (Safari 16+, limited)

**Symptom**: Subgrid alignment broken or ignored.

```css
/* Check support before using */
@supports (grid-template-rows: subgrid) {
  .child {
    display: grid;
    grid-row: span 3;
    grid-template-rows: subgrid;
  }
}
```

### 1.5 `aspect-ratio` without explicit width (Safari 14 bug)

**Symptom**: Element collapses to zero height.

```css
/* ❌ Broken in Safari 14 */
.box {
  aspect-ratio: 16 / 9;
}

/* ✅ Add explicit width or use padding-bottom hack as fallback */
.box {
  width: 100%;
  aspect-ratio: 16 / 9;
}
/* Or for maximum compatibility: */
.box-wrapper {
  position: relative;
  padding-bottom: 56.25%; /* 9/16 */
}
.box {
  position: absolute;
  inset: 0;
}
```

### 1.6 `backdrop-filter` requires `-webkit-` prefix

**Symptom**: Blur effect missing in Safari.

```css
/* ❌ Missing prefix */
.glass {
  backdrop-filter: blur(12px);
}

/* ✅ Fixed */
.glass {
  -webkit-backdrop-filter: blur(12px);
  backdrop-filter: blur(12px);
}
```

---

## 2. Viewport & Safe Area

### 2.1 Safe area insets for notch/home-bar

**Symptom**: Content behind iPhone notch or home indicator.

```html
<!-- Required in <head> -->
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
```

```css
/* Use env() for safe areas */
.header {
  padding-top: env(safe-area-inset-top);
}
.footer {
  padding-bottom: env(safe-area-inset-bottom);
}
.sidebar {
  padding-left: env(safe-area-inset-left);
  padding-right: env(safe-area-inset-right);
}

/* With fallback */
.footer {
  padding-bottom: 16px;
  padding-bottom: max(16px, env(safe-area-inset-bottom));
}
```

### 2.2 Keyboard pushes viewport up (Mobile Safari)

**Symptom**: When soft keyboard opens, `100vh` elements resize, causing layout jumps.

```css
/* Use svh (small viewport height) for elements that must stay fixed-height */
.full-screen-modal {
  height: 100svh; /* smallest possible viewport — excludes browser UI */
}

/* For chat inputs / bottom-anchored elements: */
.chat-input {
  position: fixed;
  bottom: 0;
  bottom: env(safe-area-inset-bottom);
}
```

```javascript
// JavaScript approach — listen for visual viewport changes
window.visualViewport?.addEventListener('resize', () => {
  const keyboardHeight = window.innerHeight - window.visualViewport.height;
  document.documentElement.style.setProperty('--keyboard-height', `${keyboardHeight}px`);
});
```

### 2.3 `100vw` includes scrollbar width on macOS Safari

```css
/* ❌ Causes horizontal scroll when scrollbar appears */
.full-width {
  width: 100vw;
}

/* ✅ Use 100% for block-level elements */
.full-width {
  width: 100%;
}
```

---

## 3. Scrolling Bugs

### 3.1 Momentum scroll (inertial scrolling) disabled

**Symptom**: Scrollable container on iOS feels sluggish/non-native.

```css
/* ✅ Enable iOS momentum scrolling */
.scrollable {
  overflow-y: scroll;
  -webkit-overflow-scrolling: touch; /* legacy — still helps on older iOS */
  overscroll-behavior: contain; /* prevent scroll chaining */
}
```

### 3.2 `position: fixed` element moves during scroll

**Symptom**: Fixed header/footer jumps/flickers while scrolling on iOS.

```css
/* ✅ Force GPU compositing */
.fixed-header {
  position: fixed;
  top: 0;
  transform: translateZ(0); /* promotes to compositor layer */
  -webkit-transform: translateZ(0);
  will-change: transform;
}
```

### 3.3 Scroll events not firing on body

**Symptom**: `window.addEventListener('scroll')` not triggering on iOS Safari.

```javascript
// ❌ Sometimes doesn't work on iOS
window.addEventListener('scroll', handler);

// ✅ Use document instead, or add {passive: true}
document.addEventListener('scroll', handler, { passive: true });

// Or for a custom scrollable element:
element.addEventListener('scroll', handler, { passive: true });
```

### 3.4 ScrollIntoView behavior on iOS

```javascript
// ❌ smooth behavior not supported in old Safari
element.scrollIntoView({ behavior: 'smooth' });

// ✅ Use feature detection or a polyfill
if ('scrollBehavior' in document.documentElement.style) {
  element.scrollIntoView({ behavior: 'smooth' });
} else {
  element.scrollIntoView(); // instant fallback
}
```

---

## 4. Animation & Transition Bugs

### 4.1 CSS transform flicker (promote to layer)

**Symptom**: CSS animations flicker or show artifacts in Safari.

```css
/* ✅ Force GPU compositing before animation starts */
.animated-element {
  transform: translateZ(0);
  -webkit-transform: translateZ(0);
  backface-visibility: hidden;
  -webkit-backface-visibility: hidden;
}

/* Only animate transform + opacity — these are compositor properties */
@keyframes slideIn {
  from { transform: translateX(-100%); opacity: 0; }
  to   { transform: translateX(0);     opacity: 1; }
}
```

### 4.2 `transition` on auto dimensions

**Symptom**: `height: auto` transitions don't work anywhere, but developers often think it's a Safari bug.

```css
/* ✅ Use max-height trick or JS-calculated height */
.accordion-content {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.3s ease;
}
.accordion-content.open {
  max-height: 500px; /* larger than content will ever be */
}

/* Or use JS to set explicit height: */
/* element.style.height = element.scrollHeight + 'px'; */
```

### 4.3 CSS `@keyframes` with `transform` origin

**Symptom**: Rotation or scale animation appears to rotate from wrong point.

```css
/* Explicitly set transform-origin — Safari sometimes defaults differently */
.spinner {
  transform-origin: center center;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to   { transform: rotate(360deg); }
}
```

---

## 5. JavaScript / API Bugs

### 5.1 `localStorage` throws in private browsing

**Symptom**: App crashes with `SecurityError` in Safari private mode.

```javascript
// ✅ Always wrap localStorage in try/catch
function safeLocalStorage(key, value) {
  try {
    if (value !== undefined) {
      localStorage.setItem(key, value);
    } else {
      return localStorage.getItem(key);
    }
  } catch (e) {
    console.warn('localStorage unavailable:', e);
    return null;
  }
}
```

### 5.2 `navigator.clipboard` requires HTTPS + user gesture

```javascript
// ✅ Always feature-detect and handle permission
async function copyToClipboard(text) {
  if (!navigator.clipboard) {
    // Fallback for older Safari
    const el = document.createElement('textarea');
    el.value = text;
    document.body.appendChild(el);
    el.select();
    document.execCommand('copy');
    document.body.removeChild(el);
    return;
  }
  try {
    await navigator.clipboard.writeText(text);
  } catch (err) {
    console.error('Clipboard write failed:', err);
  }
}
```

### 5.3 `structuredClone` not available (Safari < 15.4)

```javascript
// ✅ Polyfill
const deepClone = typeof structuredClone === 'function'
  ? structuredClone
  : (obj) => JSON.parse(JSON.stringify(obj));
```

### 5.4 `Array.at()` not available (Safari < 15.4)

```javascript
// ✅ Polyfill
if (!Array.prototype.at) {
  Array.prototype.at = function(n) {
    n = Math.trunc(n) || 0;
    if (n < 0) n += this.length;
    if (n < 0 || n >= this.length) return undefined;
    return this[n];
  };
}
```

---

## 6. Date Parsing Bug

### 6.1 Date string with spaces rejected (CRITICAL)

**Symptom**: `new Date(dateString)` returns `Invalid Date` or `NaN` in Safari.

**This is one of the most common Safari bugs in production apps.**

```javascript
// ❌ BROKEN IN SAFARI — ISO 8601 with space separator
new Date('2024-01-15 10:00:00') // → Invalid Date in Safari!
new Date('2024-01-15T10:00:00') // → Valid in all browsers

// ❌ Also broken
new Date('2024/01/15')          // → Invalid Date in Safari (sometimes)

// ✅ Always use ISO 8601 with T separator
new Date('2024-01-15T10:00:00Z')     // UTC
new Date('2024-01-15T10:00:00+01:00') // with offset

// ✅ If you receive a date from a server with a space, normalize it:
function parseDate(dateStr) {
  if (!dateStr) return null;
  // Replace space separator with T for Safari compatibility
  const normalized = dateStr.replace(' ', 'T');
  const date = new Date(normalized);
  return isNaN(date.getTime()) ? null : date;
}

// ✅ In Next.js with API routes, always serialize dates explicitly:
// JSON.stringify does NOT guarantee ISO format — use .toISOString()
res.json({ date: myDate.toISOString() }); // "2024-01-15T10:00:00.000Z"
```

---

## 7. Form & Input Bugs

### 7.1 `input[type=date]` styling

**Symptom**: Date inputs look different, can't be fully customized in Safari.

```css
/* Safari shows a native date picker — limited CSS control */
/* Best approach: use a JS date picker library */

/* If you must use native, at least normalize: */
input[type=date] {
  -webkit-appearance: none;
  appearance: none;
  /* Then apply your styles */
}
```

### 7.2 `input` zoom on focus (Mobile Safari)

**Symptom**: Page zooms in when user taps an input field on iOS.

```html
<!-- ✅ Prevent zoom by setting font-size to 16px minimum -->
<!-- The zoom triggers when font-size < 16px -->
```

```css
/* ✅ Ensure all inputs have at least 16px font-size */
input, select, textarea {
  font-size: max(16px, 1rem); /* never less than 16px */
}

/* OR: disable zoom entirely (not recommended for accessibility) */
/* <meta name="viewport" content="..., user-scalable=no"> */
```

### 7.3 `select` element custom styling

```css
/* Safari requires -webkit-appearance: none to allow custom styling */
select {
  -webkit-appearance: none;
  appearance: none;
  /* Now you can add custom arrow etc. */
}
```

---

## 8. Media Bugs

### 8.1 Video autoplay policy on iOS

**Symptom**: Video doesn't autoplay on iOS even with `autoplay` attribute.

```html
<!-- ✅ iOS requires: muted + playsinline + autoplay -->
<video autoplay muted playsinline loop>
  <source src="video.mp4" type="video/mp4">
</video>
```

```javascript
// For programmatic play:
const video = document.querySelector('video');
video.muted = true; // MUST be muted for autoplay
video.play().catch(err => {
  // Handle: user gesture required
  console.log('Autoplay blocked:', err);
});
```

### 8.2 Audio context requires user gesture

```javascript
// ❌ Will be blocked on iOS
const ctx = new AudioContext();
ctx.resume(); // fails silently

// ✅ Initialize inside a user interaction handler
button.addEventListener('click', async () => {
  const ctx = new AudioContext();
  if (ctx.state === 'suspended') {
    await ctx.resume();
  }
  // Now safe to use audio
});
```

### 8.3 HLS video support

```html
<!-- Safari natively supports HLS (.m3u8) — no JS library needed! -->
<video src="stream.m3u8" controls></video>

<!-- For other browsers, use hls.js with fallback: -->
```

```javascript
const video = document.querySelector('video');
if (video.canPlayType('application/vnd.apple.mpegurl')) {
  // Safari — native HLS
  video.src = 'stream.m3u8';
} else {
  // Other browsers — use hls.js
  import Hls from 'hls.js';
  const hls = new Hls();
  hls.loadSource('stream.m3u8');
  hls.attachMedia(video);
}
```

---

## 9. Touch & Pointer Bugs

### 9.1 300ms tap delay (legacy iOS)

**Symptom**: Tap/click feels delayed on older iOS.

```html
<!-- ✅ Modern fix: set touch-action in CSS -->
```

```css
/* Disable double-tap zoom → removes 300ms delay */
html {
  touch-action: manipulation;
}
/* Or per-element: */
button, a, [role=button] {
  touch-action: manipulation;
}
```

### 9.2 `:hover` persists after tap on iOS

**Symptom**: Hover styles "stick" after user taps a button on iOS.

```css
/* ✅ Only apply hover styles on devices that support hover */
@media (hover: hover) {
  .button:hover {
    background: var(--hover-color);
  }
}
```

### 9.3 Touch events vs Pointer events

```javascript
// ✅ Prefer Pointer Events API (supported Safari 13+)
element.addEventListener('pointerdown', handler);
element.addEventListener('pointermove', handler);
element.addEventListener('pointerup', handler);

// Add pointer capture for drag operations:
element.addEventListener('pointerdown', (e) => {
  element.setPointerCapture(e.pointerId);
});
```

---

## 10. Fetch & Network Bugs

### 10.1 Safari fetch cache behavior

**Symptom**: Safari caches POST requests or serves stale data.

```javascript
// ✅ Be explicit about caching
const response = await fetch('/api/data', {
  method: 'GET',
  cache: 'no-cache', // or 'no-store' for never-cache
  headers: {
    'Cache-Control': 'no-cache',
  },
});
```

### 10.2 CORS preflight differs in Safari

Safari sends `Origin` header differently in some edge cases. Always configure CORS on the server to respond to both preflight OPTIONS and actual requests with proper headers.

```
Access-Control-Allow-Origin: https://your-domain.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
```

### 10.3 Service Worker scope on iOS (Safari 16.4+)

```javascript
// ✅ Register with explicit scope
if ('serviceWorker' in navigator) {
  try {
    const registration = await navigator.serviceWorker.register('/sw.js', {
      scope: '/',
    });
    console.log('SW registered:', registration.scope);
  } catch (err) {
    console.error('SW registration failed:', err);
  }
}
```

---

## 11. Next.js Specific Bugs

### 11.1 `useLayoutEffect` SSR warning → hydration mismatch

**Symptom**: Console warning + possible hydration mismatch in Safari (and all browsers).

```typescript
// ❌ useLayoutEffect in a component that SSR renders
import { useLayoutEffect } from 'react';

// ✅ Use isomorphic version
import { useEffect, useLayoutEffect } from 'react';

const useIsomorphicLayoutEffect = 
  typeof window !== 'undefined' ? useLayoutEffect : useEffect;

export { useIsomorphicLayoutEffect };
```

### 11.2 Date serialization from server components

```typescript
// ❌ Dates are not serializable — causes hydration mismatch
export default async function Page() {
  const data = await db.query('SELECT created_at FROM posts');
  return <PostList posts={data} />; // data.created_at is a Date object
}

// ✅ Serialize dates to ISO strings before passing to client
export default async function Page() {
  const data = await db.query('SELECT created_at FROM posts');
  const serialized = data.map(post => ({
    ...post,
    created_at: post.created_at.toISOString(), // string, safe to pass
  }));
  return <PostList posts={serialized} />;
}
```

### 11.3 Next.js Image component and WebP on older Safari

```typescript
// Next.js Image automatically serves WebP — but add AVIF fallback config
// next.config.ts
const nextConfig = {
  images: {
    formats: ['image/avif', 'image/webp'],
    // Safari 16+ supports AVIF; Safari 14+ supports WebP
    // Next.js handles format negotiation via Accept header
  },
};
```

### 11.4 App Router — `cookies()` and Safari ITP

Safari's Intelligent Tracking Prevention (ITP) restricts third-party cookies. In Next.js App Router with auth:

```typescript
// ✅ Always use same-site cookies
// In middleware.ts or API routes:
response.cookies.set('session', token, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'lax', // 'strict' or 'lax' — not 'none' (Safari blocks without user gesture)
  path: '/',
});
```

### 11.5 Tailwind CSS — Safari-specific utilities

```typescript
// next.config.ts — ensure PostCSS processes vendor prefixes
// Install: npm install -D autoprefixer
// postcss.config.js:
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {}, // ← adds -webkit- prefixes automatically
  },
};
```

```css
/* Tailwind doesn't add -webkit-backdrop-filter — add manually or use @layer */
@layer utilities {
  .backdrop-blur-safari {
    -webkit-backdrop-filter: blur(12px);
    backdrop-filter: blur(12px);
  }
}
```
