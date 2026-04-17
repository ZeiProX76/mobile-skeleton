# SCOPE — buildthisniw mobile-skeleton

The goal of this skeleton: **from `/skeleton mobile` to your first shipped monetized screen in under an hour, with patterns that survive App Review and scale past 10k users.**

This is the roadmap. README.md covers the day-one setup and the testing pattern. This file covers what's in, what's next, and the order to fill the gaps.

---

## What this skeleton is

A forked + extended version of [Dusttoo/react-native-expo-supabase-starter](https://github.com/Dusttoo/react-native-expo-supabase-starter) — the baseline is solid: Expo + Supabase + RevenueCat + TanStack Query + Zustand + Jest with chainable Supabase mocks.

buildthisniw layers two things on top:

1. **Dual-rail monetization** — RevenueCat for the default IAP path, plus a Stripe deep-link arbitrage flow for web-first subscribers (the Apple-fee avoidance pattern that's legal under App Store Guideline 3.1.3 and the EU DMA).
2. **codekit command integration** — `/mobile`, `/payments`, `/deploy` all know the layout of this skeleton, so the orchestrator can route work to the right files without re-learning the stack every session.

---

## MVP — the critical path

Everything on this list is required before the buyer's first feature ships to TestFlight or the Play Console. If it's not on this list, it's roadmap.

| # | Capability | Status | Files |
|---|-----------|--------|-------|
| 1 | Supabase client + auth | ✅ shipped | `src/lib/supabase.ts`, `app/(auth)/*` |
| 2 | Tab + stack routing | ✅ shipped | `app/_layout.tsx`, `app/(tabs)/*` |
| 3 | Data layer (services + TanStack Query) | ✅ shipped | `src/services/`, `src/hooks/` |
| 4 | UI state (Zustand) | ✅ shipped | `src/store/` |
| 5 | RevenueCat paywall (IAP rail) | ✅ shipped | `app/paywall.tsx`, `docs/revenuecat-setup.md` |
| 6 | Jest with chainable Supabase mocks | ✅ shipped | `tests/helpers/supabase-chain-mock.ts` |
| 7 | Stripe deep-link arbitrage (web rail) | ⚠️ planned | see Task M1 below |
| 8 | Sentry + error reporting | ⚠️ planned | Task M2 |
| 9 | Expo OTA updates configured | ⚠️ planned | Task M3 |
| 10 | Push notifications (Expo Notifications) | ⚠️ planned | Task M4 |
| 11 | App Store / Play Store submission checklist | ⚠️ planned | Task M5 |

---

## Roadmap — task cards

Each card is self-contained. A buyer (or future session) should be able to pick one and ship it without rereading the rest of this doc.

### M1 · Stripe deep-link arbitrage flow

**Why:** Apple takes 30% on in-app purchases. For web-first subscribers (Dropbox / Netflix model), letting users pay on the web and deep-linking back into the app avoids that fee while staying compliant with Guideline 3.1.3 (reader app exception) or the EU DMA alternative payment entitlement. Covered by the `iap-compliance` skill.

**Acceptance:**
- Tap "Subscribe on web" on `paywall.tsx` → opens Stripe Checkout in system browser
- After payment → Stripe redirects to `buildthisniw-mobile://entitlement-active?token=...`
- App parses the deep link, writes entitlement to Supabase, unlocks gated screens
- Works in Expo dev client and in a production build

**Files:**
- `app.json` — add `"scheme": "buildthisniw-mobile"` (~3 lines)
- `src/lib/deep-link.ts` — NEW, Expo Linking handler (~40 lines)
- `src/services/checkout-service.ts` — NEW, calls edge function (~30 lines)
- `supabase/functions/create-checkout/index.ts` — NEW, Stripe Checkout Session (~60 lines)
- `supabase/functions/stripe-webhook/index.ts` — NEW, entitlement sync (~70 lines)
- `.env.example` — add Stripe keys + app scheme (~4 lines)

Estimate: ~200 lines across 6 files.

### M2 · Sentry error reporting

**Why:** Crash-free rate is an App Store metric. Silent errors in production are worse than visible ones.

**Acceptance:**
- Uncaught error → appears in Sentry within 30 seconds with source map
- Network errors from TanStack Query → breadcrumbs attached
- User identified by Supabase UID after auth

**Files:**
- `src/lib/sentry.ts` — NEW, init + scope helpers (~40 lines)
- `app/_layout.tsx` — wrap root with `Sentry.wrap()` (~5 lines)
- `src/hooks/use-query-error-handler.ts` — NEW, adds Sentry breadcrumbs (~20 lines)
- `.env.example` — add `EXPO_PUBLIC_SENTRY_DSN`

Estimate: ~70 lines.

### M3 · Expo OTA updates

**Why:** Fix a JS bug without a week of App Review.

**Acceptance:**
- `eas update --branch production` pushes a JS-only patch
- App checks for updates on foreground, applies on next launch
- Update channel gated by `EAS_CHANNEL` env

**Files:**
- `app.json` — `expo.updates` block (~10 lines)
- `src/lib/updates.ts` — NEW, check + apply helper (~30 lines)
- `app/_layout.tsx` — call update check on mount (~5 lines)
- `eas.json` — NEW, build + update channels (~30 lines)

Estimate: ~75 lines.

### M4 · Push notifications

**Why:** Re-engagement is how mobile apps survive. Also required for most paywall nag flows.

**Acceptance:**
- App requests permission after onboarding (not on first launch)
- Expo push token written to `profiles.push_token`
- Supabase edge function can send a push via the Expo Push API

**Files:**
- `src/lib/notifications.ts` — NEW, permission + token registration (~50 lines)
- `src/hooks/use-push-registration.ts` — NEW (~30 lines)
- `supabase/functions/send-push/index.ts` — NEW (~40 lines)
- `docs/push-setup.md` — NEW, APNs + FCM key upload steps (~60 lines)

Estimate: ~180 lines.

### M5 · Store submission checklist

**Why:** First submission is the longest. A checklist collapses it from two days to two hours.

**Acceptance:**
- `docs/submission.md` covers: bundle ID, privacy labels, export compliance, screenshots, review notes template, IAP metadata
- Each item has a doc link or a `TODO:` marker the buyer can grep for

Estimate: ~100 line doc, no code.

---

## Non-goals for this skeleton

- **Not a monorepo** — the buildthisniw kit is three repos (web / mobile / desktop) because the tooling diverges (EAS vs Vercel vs Tauri). Don't try to unify them.
- **No web build** — Expo's web output works for demos but the buyer ships web from `webapp-skeleton`. Keep React Native primitives, don't chase universal-web compatibility.
- **No bare workflow** — managed Expo is the velocity win. If the buyer needs a custom native module, they eject after product-market fit, not before.
- **No second UI library** — Zustand + RN primitives are the style. Don't add Tamagui, Gluestack, or NativeBase.

---

## How codekit commands compose on top

| Command | What it does here |
|---------|-------------------|
| `/mobile <feature>` | Authors a screen + service + hook + test using this skeleton's conventions |
| `/payments` | Adds a Stripe edge function or RevenueCat entitlement in the right folder |
| `/deploy mobile` | Runs `eas build` and `eas submit` with the configured channels |
| `/audit mobile` | Checks paywall language against `iap-compliance` rules before submission |

The skills that know this skeleton: `revenuecat`, `iap-compliance`, `expo-build-and-test`, `stripe-payments`.

---

## Version

This scope doc tracks the repo `main` branch. For the version wired into codekit, see `SKELETONS.mobile.commit` in `codekit/lib/constants.js`.
