# Safari/WebKit Feature Compatibility Tables

Quick-reference tables for checking feature support across Safari versions.
All data reflects Safari on macOS. iOS Safari matches the macOS version shipped with the same OS release.

---

## CSS Features

| Feature | Safari 14 | Safari 15 | Safari 16 | Safari 17 | Safari 18 | Notes |
|---|---|---|---|---|---|---|
| `gap` in Flexbox | ⚠️ 14.1 | ✅ | ✅ | ✅ | ✅ | Use @supports gap |
| `aspect-ratio` | ⚠️ Bug | ✅ | ✅ | ✅ | ✅ | Need explicit width in 14 |
| `backdrop-filter` | ✅ (prefixed) | ✅ | ✅ | ✅ | ✅ | Always use -webkit- prefix |
| CSS Grid subgrid | ❌ | ❌ | ✅ | ✅ | ✅ | Use @supports |
| `container` queries | ❌ | ❌ | ✅ | ✅ | ✅ | Safari 16+ |
| `:has()` selector | ❌ | ❌ | ✅ | ✅ | ✅ | Safari 15.4+ actually |
| `100dvh` / `100svh` | ❌ | ❌ | ✅ 16.4 | ✅ | ✅ | Dynamic/Small viewport |
| CSS `@layer` | ❌ | ✅ 15.4 | ✅ | ✅ | ✅ | |
| CSS `@container` | ❌ | ❌ | ✅ | ✅ | ✅ | |
| `color-mix()` | ❌ | ❌ | ✅ | ✅ | ✅ | |
| `@starting-style` | ❌ | ❌ | ❌ | ✅ | ✅ | Entry animations |
| CSS View Transitions | ❌ | ❌ | ❌ | ❌ | ✅ 18 | |
| Style Queries | ❌ | ❌ | ❌ | ❌ | ✅ 18 | |
| `text-wrap: balance` | ❌ | ❌ | ❌ | ✅ | ✅ | |
| `:focus-visible` | ❌ | ✅ 15.4 | ✅ | ✅ | ✅ | |
| `overscroll-behavior` | ✅ | ✅ | ✅ | ✅ | ✅ | |
| `scroll-behavior: smooth` | ⚠️ | ✅ 15.4 | ✅ | ✅ | ✅ | |
| Scroll snap | ✅ | ✅ | ✅ | ✅ | ✅ | |
| `contain` | ✅ | ✅ | ✅ | ✅ | ✅ | |
| Custom Properties (vars) | ✅ | ✅ | ✅ | ✅ | ✅ | |
| `clamp()` | ✅ | ✅ | ✅ | ✅ | ✅ | |
| `:is()` / `:where()` | ✅ 14 | ✅ | ✅ | ✅ | ✅ | |
| `align-content` in block | ❌ | ❌ | ❌ | ✅ 17.4 | ✅ | |

---

## JavaScript / Web APIs

| Feature | Safari 14 | Safari 15 | Safari 16 | Safari 17 | Safari 18 | Notes |
|---|---|---|---|---|---|---|
| `structuredClone` | ❌ | ❌ | ✅ 15.4 | ✅ | ✅ | Use JSON fallback |
| `Array.at()` | ❌ | ❌ | ✅ 15.4 | ✅ | ✅ | |
| `Object.hasOwn()` | ❌ | ❌ | ✅ 15.4 | ✅ | ✅ | |
| Top-level `await` | ❌ | ✅ | ✅ | ✅ | ✅ | |
| `queueMicrotask` | ✅ | ✅ | ✅ | ✅ | ✅ | |
| `ResizeObserver` | ✅ 13.1 | ✅ | ✅ | ✅ | ✅ | |
| `IntersectionObserver` | ✅ 12.1 | ✅ | ✅ | ✅ | ✅ | v2 options limited |
| `WeakRef` | ✅ 14.5 | ✅ | ✅ | ✅ | ✅ | |
| `globalThis` | ✅ | ✅ | ✅ | ✅ | ✅ | |
| `Promise.allSettled` | ✅ | ✅ | ✅ | ✅ | ✅ | |
| `Promise.any` | ✅ | ✅ | ✅ | ✅ | ✅ | |
| `AbortController` | ✅ | ✅ | ✅ | ✅ | ✅ | |
| `URLPattern` | ❌ | ❌ | ❌ | ❌ | ❌ | Not supported |
| Web Workers | ✅ | ✅ | ✅ | ✅ | ✅ | |
| WebAssembly | ✅ | ✅ | ✅ | ✅ | ✅ | |
| `navigator.clipboard` | ✅ | ✅ | ✅ | ✅ | ✅ | HTTPS + gesture required |
| Web Share API | ✅ | ✅ | ✅ | ✅ | ✅ | |
| Web Crypto | ✅ | ✅ | ✅ | ✅ | ✅ | |
| File System Access | ❌ | ❌ | ❌ | ❌ | ❌ | Not planned |
| Web Bluetooth | ❌ | ❌ | ❌ | ❌ | ❌ | Not supported |
| Web USB | ❌ | ❌ | ❌ | ❌ | ❌ | Not supported |
| WebXR | ❌ | ❌ | ❌ | ❌ | ✅ 18 | visionOS focus |
| `Error.cause` | ❌ | ✅ 15 | ✅ | ✅ | ✅ | |
| `at()` on String | ❌ | ❌ | ✅ 15.4 | ✅ | ✅ | |
| Temporal API | ❌ | ❌ | ❌ | ❌ | ❌ | Proposal stage |

---

## PWA / Service Worker Features (iOS Safari)

| Feature | iOS 14 | iOS 15 | iOS 16 | iOS 16.4 | iOS 17 | iOS 18 |
|---|---|---|---|---|---|---|
| Service Workers | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Web App Manifest | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| `display: standalone` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Push Notifications | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| Background Sync | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Cache API | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| IndexedDB | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Home Screen badges | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| `navigator.standalone` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Web Share Target | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Payment Request API | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

**Critical Note on iOS Push (16.4+)**:
- App must be installed to home screen (standalone mode)
- User must grant permission after install
- Must be served over HTTPS
- Different permission prompt timing than native apps

---

## Form Input Types

| Input Type | Safari Support | Notes |
|---|---|---|
| `date` | ✅ | Native picker, limited CSS |
| `datetime-local` | ✅ | Safari 14.1+ |
| `time` | ✅ | |
| `month` | ❌ | Falls back to text |
| `week` | ❌ | Falls back to text |
| `color` | ✅ | |
| `range` | ✅ | |
| `search` | ✅ | |
| `number` | ✅ | |
| `email` | ✅ | |
| `url` | ✅ | |
| `tel` | ✅ | |

---

## Media Formats

| Format | Safari Support | Notes |
|---|---|---|
| MP4 (H.264) | ✅ | Best compatibility |
| WebM (VP9) | ✅ 14.1+ | |
| WebM (AV1) | ✅ 17+ | |
| HEVC | ✅ | Hardware decode |
| HLS (.m3u8) | ✅ | Native — no JS needed |
| DASH | ❌ | Not native, needs JS |
| WebP | ✅ 14+ | |
| AVIF | ✅ 16+ | |
| HEIC | ✅ 17+ | |
| MP3 | ✅ | |
| AAC | ✅ | |
| OGG | ❌ | Not supported |
| FLAC | ✅ | |
| Opus | ✅ 15.4+ | |

---

## WebKit vs Chromium: Key Engine Differences

| Behavior | WebKit (Safari) | Blink (Chrome) |
|---|---|---|
| JavaScript Engine | JavaScriptCore (JSC) | V8 |
| CSS prefix | -webkit- | -webkit- (legacy) |
| Rendering | WebKit compositor | Skia + CC |
| GPU process | Separate | Separate |
| Service Worker scope | Matches spec strictly | More permissive |
| ITP (tracking protection) | Yes (aggressive) | No |
| Third-party cookies | Blocked by default | Being deprecated |
| `<input type=date>` | Native OS picker | Custom Chrome UI |
| `100vh` behavior | Includes browser chrome | Excludes by default |
| CSS scroll snap | Slightly different momentum | Snaps on scroll end |
| Flexbox rounding | Sometimes different | Different from spec |
| Font rendering | Subpixel on macOS | Grey AA |
| `window.open` in async | Blocked as popup | Sometimes allowed |
| RegExp lookbehind | ✅ Safari 16.4+ | ✅ Long-supported |

---

## Useful Testing Targets

For production apps, prioritize testing against:

1. **Safari 16.4** — First iOS version with PWA push notifications; wide iOS 16 install base
2. **Safari 17** — Major CSS additions (@starting-style, popover)
3. **Safari 18** — View Transitions, WebXR
4. **Mobile Safari on iPhone SE** — Smallest screen, reveals layout issues
5. **iPad Safari in Split View** — Viewport math gets interesting

**iOS Version → Safari Version mapping**:
- iOS 15 → Safari 15
- iOS 16 → Safari 16
- iOS 17 → Safari 17
- iOS 18 → Safari 18
