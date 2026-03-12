# Mobile Safari Reference

iOS/iPadOS-specific Safari behavior, quirks, and constraints.

---

## The iOS Browser Monopoly

**Critical fact**: Every browser on iOS — Chrome, Firefox, Edge, Brave, DuckDuckGo — uses WebKit under the hood. Apple requires this by App Store policy. This means:

- "Works in Chrome on desktop, broken on iOS" = WebKit bug
- "Works in Chrome on Android, broken on iOS" = Also WebKit bug (Android Chrome uses Blink)
- There is no way to test non-WebKit behavior on iOS

---

## Table of Contents

1. [Viewport Behavior](#1-viewport-behavior)
2. [Keyboard & Input](#2-keyboard--input)
3. [Scrolling Behavior](#3-scrolling-behavior)
4. [Touch Behavior](#4-touch-behavior)
5. [PWA Behavior](#5-pwa-behavior)
6. [Permissions & APIs](#6-permissions--apis)
7. [Media Playback](#7-media-playback)
8. [Storage & Cookies](#8-storage--cookies)
9. [iOS-Specific CSS](#9-ios-specific-css)
10. [Debugging Mobile Safari](#10-debugging-mobile-safari)

---

## 1. Viewport Behavior

### The Safari Toolbar Problem

Mobile Safari has a dynamic top and bottom toolbar that shows/hides during scrolling. This means the visual viewport height changes, which breaks naive `100vh` usage.

```
┌─────────────────┐ ← Status bar (always visible)
│ ◀ Address bar   │ ← URL bar (hides when scrolling down)
├─────────────────┤
│                 │
│   Content       │ ← Visual viewport (changes size!)
│                 │
├─────────────────┤
│ [Tab] [Share]   │ ← Bottom toolbar (hides when scrolling)
└─────────────────┘ ← Home indicator (always ~34px on Face ID phones)
```

**Viewport units guide**:

| Unit | Meaning | Use For |
|---|---|---|
| `100vh` | CSS viewport (never changes) | Legacy fallback |
| `100svh` | Small viewport height (toolbars visible) | Fixed overlays that must fit when toolbars show |
| `100lvh` | Large viewport height (toolbars hidden) | Layouts that should use full screen when scrolling |
| `100dvh` | Dynamic viewport height (changes with toolbars) | Usually the right choice for full-screen layouts |

```css
/* Full-screen page section */
.hero {
  height: 100vh;     /* fallback */
  height: 100svh;    /* safe: always fits even with toolbars */
}

/* Modal that should use all available space */
.modal {
  max-height: 100dvh; /* adapts as toolbars show/hide */
}
```

### `viewport-fit=cover`

Required for content to extend behind the notch/home-bar:

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
```

Without this, `env(safe-area-inset-*)` returns 0.

---

## 2. Keyboard & Input

### Soft Keyboard Changes Layout

When the soft keyboard opens, Mobile Safari does NOT resize the `window.innerHeight`. Instead, the `VisualViewport` shrinks. This breaks fixed-position bottom elements (chat input, bottom nav).

```typescript
// ✅ Use Visual Viewport API for keyboard-aware layouts
function useKeyboardHeight() {
  const [keyboardHeight, setKeyboardHeight] = useState(0);
  
  useEffect(() => {
    if (!window.visualViewport) return;
    
    const handler = () => {
      const keyboardH = window.innerHeight - (window.visualViewport?.height ?? window.innerHeight);
      setKeyboardHeight(Math.max(0, keyboardH));
    };
    
    window.visualViewport.addEventListener('resize', handler);
    return () => window.visualViewport?.removeEventListener('resize', handler);
  }, []);
  
  return keyboardHeight;
}

// Usage in a chat component:
function ChatInput() {
  const keyboardHeight = useKeyboardHeight();
  
  return (
    <div
      style={{
        position: 'fixed',
        bottom: `max(${keyboardHeight}px, env(safe-area-inset-bottom))`,
        left: 0,
        right: 0,
      }}
    >
      <input type="text" />
    </div>
  );
}
```

### Input Zoom Prevention

iOS Safari zooms in when focusing an input with `font-size < 16px`.

```css
/* ✅ Never use font-size below 16px on inputs */
input,
select,
textarea {
  font-size: 16px;
  font-size: max(16px, 1rem);
}

/* If you must have small text, scale with transform after focus: */
input {
  font-size: 16px;
  transform: scale(0.875);
  transform-origin: left center;
}
```

### Input `type=date` on iOS

```typescript
// iOS shows a native date picker (scroll-wheel style)
// You CANNOT style it with CSS beyond basic properties
// Consider a JS date picker for consistent cross-platform UX:
// - react-datepicker
// - @radix-ui/react-popover + a calendar component
// - shadcn/ui Calendar
```

---

## 3. Scrolling Behavior

### Overscroll / Rubber Band Effect

iOS has a rubber band scroll effect. Control it with `overscroll-behavior`:

```css
/* Prevent overscroll on a specific element */
.modal {
  overscroll-behavior: contain; /* overscroll stops at element boundary */
}

/* Prevent body rubber-band (use carefully — kills native feel) */
body {
  overscroll-behavior: none; /* disables pull-to-refresh too */
}
```

### `position: fixed` During Scroll

Elements with `position: fixed` on iOS jitter during momentum scroll. Fix:

```css
.sticky-header {
  position: fixed;
  top: 0;
  /* Force GPU layer */
  -webkit-transform: translateZ(0);
  transform: translateZ(0);
  will-change: transform;
}
```

### Scroll Event Firing

On iOS, `scroll` events only fire after scrolling stops (not continuously):

```typescript
// ❌ Won't fire continuously on iOS — only at end of scroll
window.addEventListener('scroll', updateScrollPosition);

// ✅ Use scrollend event (Safari 17.4+) or touchmove for continuous:
window.addEventListener('touchmove', updateScrollPosition, { passive: true });

// Or accept iOS scroll behavior and use IntersectionObserver instead of scroll:
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    // fires when element enters/exits viewport
  });
}, { threshold: [0, 0.5, 1] });
observer.observe(targetElement);
```

---

## 4. Touch Behavior

### Disabling Touch Highlights

```css
/* Remove the grey tap highlight on iOS */
* {
  -webkit-tap-highlight-color: transparent;
}

/* But keep focus visible for accessibility: */
:focus-visible {
  outline: 2px solid var(--color-primary);
}
```

### Preventing Default Touch Behavior

```typescript
// ✅ Passive event listeners for better scroll performance
element.addEventListener('touchstart', handler, { passive: true });
element.addEventListener('touchmove', handler, { passive: true });

// ❌ Non-passive blocks scroll thread — avoid unless necessary
element.addEventListener('touchmove', (e) => {
  e.preventDefault(); // Requires passive: false, slows scrolling
}, { passive: false });
```

### Touch vs Click Delay

```css
/* Modern fix — sets touch-action to remove 300ms delay */
html {
  touch-action: manipulation;
}
```

---

## 5. PWA Behavior

### Adding to Home Screen

```html
<!-- In <head> for proper PWA support on iOS -->
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
<!-- Options: default | black | black-translucent -->

<meta name="apple-mobile-web-app-title" content="App Name">

<!-- Icons for iOS home screen -->
<link rel="apple-touch-icon" href="/icon-180.png"> <!-- 180x180 -->

<!-- Splash screens (optional but improves launch experience) -->
<link
  rel="apple-touch-startup-image"
  media="(device-width: 390px) and (device-height: 844px) and (-webkit-device-pixel-ratio: 3)"
  href="/splash-390x844.png"
/>
```

### PWA Limitations on iOS

| Feature | Status | Notes |
|---|---|---|
| Standalone mode | ✅ | Works well |
| Service Worker | ✅ | Safari 11.1+ |
| Push Notifications | ✅ | iOS 16.4+ — home screen only |
| Background Sync | ❌ | Not supported |
| Web Share | ✅ | Safari 14+ |
| Badging API | ✅ | iOS 16.4+ — home screen only |
| `navigator.standalone` | ✅ | Detect if running as PWA |
| Persistent storage | ⚠️ | ITP may clear after 7 days |

### Detecting PWA Mode

```typescript
// Check if running as an installed PWA
const isStandalone = window.navigator.standalone === true
  || window.matchMedia('(display-mode: standalone)').matches;

// React hook:
function useIsInstalledPWA() {
  return useMemo(() => 
    typeof window !== 'undefined' && (
      (window.navigator as any).standalone === true ||
      window.matchMedia('(display-mode: standalone)').matches
    ),
  []);
}
```

---

## 6. Permissions & APIs

### Requesting Permissions

```typescript
// Camera & Microphone (requires HTTPS + user gesture)
async function requestCamera() {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ video: true });
    return stream;
  } catch (err) {
    if (err.name === 'NotAllowedError') {
      // User denied — show instructions to enable in Settings
    }
  }
}

// Geolocation
function requestLocation() {
  if (!navigator.geolocation) return;
  
  navigator.geolocation.getCurrentPosition(
    (position) => { /* success */ },
    (error) => { /* denied or error */ },
    { enableHighAccuracy: true, timeout: 5000 }
  );
}

// Notifications (iOS 16.4+ — PWA only)
async function requestNotifications() {
  if (!('Notification' in window)) return;
  
  // Must be called from a user gesture
  const permission = await Notification.requestPermission();
  // 'granted' | 'denied' | 'default'
}
```

### Web Share API

```typescript
// ✅ Prefer Web Share on mobile for native share sheet
async function share(data: { title?: string; text?: string; url?: string }) {
  if (navigator.canShare?.(data)) {
    try {
      await navigator.share(data);
    } catch (err) {
      if (err.name !== 'AbortError') {
        // User dismissed — normal, not an error
        console.error('Share failed:', err);
      }
    }
  } else {
    // Fallback: copy to clipboard
    await navigator.clipboard.writeText(data.url || data.text || '');
  }
}
```

---

## 7. Media Playback

### Video Autoplay on iOS

**Rule**: iOS requires `muted` + `playsinline` for autoplay. No exceptions.

```typescript
// ✅ React component for auto-playing background video
export function BackgroundVideo({ src }: { src: string }) {
  return (
    <video
      src={src}
      autoPlay
      muted        // REQUIRED for autoplay
      playsInline  // REQUIRED — prevents fullscreen on iOS
      loop
      style={{ width: '100%', height: '100%', objectFit: 'cover' }}
    />
  );
}

// ✅ For user-initiated video (with sound):
function VideoPlayer({ src }: { src: string }) {
  const videoRef = useRef<HTMLVideoElement>(null);
  
  const handlePlay = async () => {
    if (!videoRef.current) return;
    try {
      await videoRef.current.play();
    } catch (err) {
      console.error('Play failed:', err);
    }
  };
  
  return (
    <>
      <video ref={videoRef} src={src} playsInline controls />
      <button onClick={handlePlay}>Play</button>
    </>
  );
}
```

---

## 8. Storage & Cookies

### ITP (Intelligent Tracking Prevention)

Safari's ITP aggressively blocks cross-site tracking:

- Third-party cookies blocked by default
- `localStorage` / `sessionStorage` partitioned by site
- After 7 days of no interaction: data may be purged

**Impact on apps**:
- OAuth flows with third-party domains may fail
- Analytics cookies from CDN subdomains may be blocked
- Auth tokens in third-party iframes may be cleared

**Mitigations**:
```typescript
// 1. Use same-site, httpOnly cookies for auth
// 2. Use Storage Access API if you need cross-site storage:
async function requestStorageAccess() {
  try {
    await document.requestStorageAccess();
    // Now can access first-party cookies in third-party iframe
  } catch {
    // Denied — show UI to guide user
  }
}

// 3. Move auth to first-party domain
// Instead of auth.example.com (third-party)
// Use app.example.com/auth (first-party)
```

### Private Browsing

```typescript
// Safari Private Mode:
// - localStorage throws SecurityError
// - sessionStorage works but limited
// - IndexedDB blocked
// - Cookies blocked

// ✅ Detect private mode (not 100% reliable):
async function isPrivateBrowsing(): Promise<boolean> {
  try {
    localStorage.setItem('__test', '1');
    localStorage.removeItem('__test');
    return false;
  } catch {
    return true;
  }
}
```

---

## 9. iOS-Specific CSS

```css
/* Disable callout (long-press popup) on links/images */
-webkit-touch-callout: none;

/* Disable text selection on UI elements */
-webkit-user-select: none;
user-select: none;

/* Enable momentum scrolling on custom scroll containers */
-webkit-overflow-scrolling: touch;

/* Prevent text size adjustment when rotating device */
-webkit-text-size-adjust: 100%;
text-size-adjust: 100%;

/* Safe area utilities (requires viewport-fit=cover) */
padding-top: env(safe-area-inset-top);
padding-bottom: env(safe-area-inset-bottom);
padding-left: env(safe-area-inset-left);
padding-right: env(safe-area-inset-right);

/* With fallbacks: */
padding-bottom: 16px;
padding-bottom: max(16px, env(safe-area-inset-bottom));
```

---

## 10. Debugging Mobile Safari

### Remote Debugging Setup

1. **iPhone/iPad**: Settings → Safari → Advanced → Web Inspector: ON
2. **Mac**: Safari → Settings → Advanced → Show features for web developers
3. Connect iPhone to Mac with USB cable
4. Mac Safari: Develop menu → [Your Device] → [Your Page]

**Full remote devtools opens** — console, elements, network, sources all work.

### Simulator (Xcode required)

1. Open Xcode → Xcode menu → Open Developer Tool → Simulator
2. Run your site in Simulator's Safari
3. Mac Safari: Develop → Simulator → [Your Page]

### Remote Inspection Without a Mac

If you only have Windows/Linux:

1. **BrowserStack** or **LambdaTest** — cloud Safari/iOS testing
2. **Appetize.io** — iOS Simulator in browser
3. **Inspect.dev** — Windows/Linux remote debugging for iOS Safari

### Common Console Errors to Watch For

```
SecurityError: The operation is insecure.
→ localStorage in private mode, or mixed content

TypeError: undefined is not an object (evaluating 'x.y')
→ Optional chaining needed, or API not supported

AbortError: Fetch is aborted
→ Usually AbortController / navigation, not a bug

NotAllowedError: The request is not allowed
→ Autoplay, clipboard, or permission denied

InvalidStateError: The object is in an invalid state.
→ AudioContext not started from user gesture
```
