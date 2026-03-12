# 🧭 Safari/WebKit Skill for Claude

A professional-grade Claude skill that turns Claude into a Safari/WebKit compatibility expert. Built for developers using **Next.js**, **React**, **TypeScript**, and modern web tooling who are tired of "works in Chrome, broken in Safari."

---

## What It Does

Once installed, Claude automatically activates this skill when it detects Safari/WebKit-related work. It can:

- **Diagnose** Safari-specific bugs from symptoms alone
- **Review code** for WebKit compatibility issues before they ship
- **Fix** CSS, JS, and React/Next.js bugs with minimal, targeted patches
- **Explain** why something behaves differently in WebKit vs Chromium
- **Optimize** performance for JavaScriptCore and WebKit's compositor
- **Guide** Mobile Safari quirks: viewport, keyboard, touch, safe areas, PWA

---

## Skill Contents

```
safari-webkit-skill/
├── SKILL.md                        # Core skill — workflows, checklists, behavior rules
└── references/
    ├── bug-patterns.md             # 30+ confirmed bugs with fixes (CSS, JS, Next.js, media...)
    ├── compat-tables.md            # Feature support across Safari 14–18 + WebKit vs Blink table
    ├── nextjs-safari.md            # Next.js-specific: hydration, App Router, Tailwind, auth, fonts
    └── mobile-safari.md           # iOS Safari: viewport, keyboard, touch, PWA, ITP, debugging
```

---

## Installation

### Claude.ai (Web / Desktop / Mobile)

> Requires a **Pro, Max, Team, or Enterprise** plan with **Code Execution** enabled.

1. Go to **Settings → Capabilities** and enable **Code execution and file creation**
2. Go to **Customize → Skills**
3. Click **Upload skill** and select `safari-webkit-skill.skill`  
   *(or zip the folder yourself — see below)*
4. Toggle the skill **on**

That's it. Claude will automatically load it when relevant.

### Claude Code (CLI)

```bash
# Personal install — available across all your projects
cp -r safari-webkit-skill ~/.claude/skills/

# Or project-local install
cp -r safari-webkit-skill .claude/skills/
```

Claude Code discovers and invokes the skill automatically based on context.

### Manual ZIP upload

If you cloned this repo and need a `.zip`:

```bash
zip -r safari-webkit-skill.zip safari-webkit-skill/
```

Then upload the zip in **Customize → Skills**.

---

## Usage

The skill activates automatically. Just describe your problem naturally:

```
"My video component works in Chrome but won't autoplay on iOS"

"Safari is rendering my flexbox layout differently than Chrome"

"I'm getting a hydration mismatch in Next.js but only in Safari"

"Review this component for Safari compatibility"

"My position: fixed header flickers on iPhone while scrolling"
```

You can also be explicit:

```
"Use the safari-webkit skill to audit this CSS"
```

---

## Example Fixes Included

- `100vh` + Mobile Safari toolbar → `dvh` fallback pattern
- `position: sticky` broken inside `overflow: hidden` → `overflow: clip`
- `new Date('2024-01-15 10:00:00')` → `Invalid Date` in Safari (the silent killer)
- Flexbox `gap` not working on iOS 14 → `@supports` pattern
- Video autoplay blocked on iOS → `muted` + `playsinline` requirement
- `useLayoutEffect` SSR warning → isomorphic hook pattern
- `backdrop-filter` missing → `-webkit-backdrop-filter` prefix
- Input zoom on focus → `font-size: 16px` minimum
- Soft keyboard pushing layout → `VisualViewport` API pattern
- Safari ITP blocking auth cookies → `sameSite: lax` config

---

## Compatibility Tables Included

Quick-reference tables for:

- CSS features across Safari 14 → 18
- Web APIs across Safari 14 → 18  
- PWA/Service Worker features on iOS 14 → 18
- Media format support
- WebKit vs Chromium/Blink engine differences

---

## Built With

Created using the [skill-creator](https://github.com/anthropics/skills) skill and the [Agent Skills open standard](https://agentskills.io).

---

## Contributing

Found a Safari bug that's not covered? PRs welcome.

Add it to `references/bug-patterns.md` following the existing format:

```markdown
### X.Y Short descriptive title

**Symptom**: What the developer sees

**Before (broken in Safari):**
[minimal code that reproduces]

**After (fixed):**
[corrected code]
```

---

## License

MIT
