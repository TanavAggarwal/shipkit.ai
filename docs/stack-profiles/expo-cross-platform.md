# Stack profile: expo-cross-platform

Strict profile for applications that must run on **Web + iOS + Android from one shared codebase** using Expo and React Native patterns.

Use this as a checklist. Commands shown here are examples; the authoritative commands live in [`../../PROJECT.md`](../../PROJECT.md).

## Forbidden APIs / patterns

Every line of shared frontend code must work on **Web + iOS + Android**. Before merging anything, an agent must mentally validate against all three.

1. **No web-only DOM APIs in shared code.** No `window`, `document`, `localStorage`, `navigator` directly. Use Expo equivalents:
   - Storage → `expo-secure-store` (sensitive) or `@react-native-async-storage/async-storage`
   - Linking → `expo-linking`
   - File picking → `expo-document-picker` / `expo-image-picker`
   - Camera → `expo-camera`
2. **No native-only APIs in shared code without a fallback.** When using a native module, check `Platform.OS` and provide a web fallback or a `.web.tsx` / `.native.tsx` file split.
3. **Use RN primitives.** `View`, `Text`, `Pressable`, `ScrollView`, `FlatList` — never raw `<div>` / `<span>` / `<button>`. Never import from `react-native-web` directly.
4. **Responsive by default.** Layouts must adapt from a 320 px phone to a 1920 px desktop. Use:
   - Flexbox (default), `gap`, percentage widths, `maxWidth` containers.
   - `useWindowDimensions()` for true responsive logic — never hard-coded breakpoints.
   - Defined breakpoints in `theme/breakpoints.ts`: `sm: 640, md: 768, lg: 1024, xl: 1280`.
   - **No fixed pixel widths** for content containers. No `position: absolute` for layout (only for overlays).
5. **Touch targets ≥ 44 × 44 px.** Mouse-only patterns (hover-only menus, right-click) must have a tap-friendly equivalent.
6. **Safe areas.** Wrap top-level screens in `SafeAreaView` from `react-native-safe-area-context`.
7. **Keyboard handling.** Forms use `KeyboardAvoidingView` with platform-aware behavior.
8. **Images.** Use `expo-image` (handles web + native). Always set `contentFit` and an `accessibilityLabel`.
9. **Fonts.** Loaded via `expo-font`. Never use platform-only font families.
10. **Navigation.** Always via `expo-router`. Never `react-navigation` directly. Never browser `<a href>`.
11. **Platform-specific files.** Use the Metro `.web.ts` / `.ios.ts` / `.android.ts` / `.native.ts` extension split when divergence is unavoidable. Keep these files thin wrappers around a shared interface.
12. **Test on all three.** New screens must include a validation note stating which platforms were validated and how.

Additional strict rules:

- Do not rely on public web asset paths from native code.
- Do not add native modules without documenting development-build implications and adding appropriate validation gates.
- Do not fork whole features by platform unless the plan explicitly approves it.

## Required test types

- **Static/type/lint:** shared TypeScript and lint gates from `PROJECT.md`.
- **Unit tests:** pure logic, adapters, hooks, validators, and platform abstractions.
- **Component tests:** React Native Testing Library for changed components/screens and visible UI states.
- **Integration tests:** API, persistence, auth, file, media, notification, and platform-service boundaries.
- **Web E2E smoke:** Playwright against the web build via [`app-testing`](../../.github/skills/app-testing/SKILL.md) for new or materially changed user flows.
- **Native smoke:** Android and iOS simulator/device checks when available, especially after native dependencies, navigation, storage, camera, file, or deep-link changes.

## Required E2E behavior

For frontend changes at standard or complex scope:

1. Author the Playwright smoke required by the plan and active project harness.
2. Boot the web build with the command declared in `PROJECT.md`.
3. Exercise one happy-path user flow per affected screen or route.
4. Verify responsive behavior at narrow phone width and at least one desktop width when layout changed.
5. Use user-visible text, labels, roles, or accessibility names where possible.
6. Mock or seed network state deterministically.
7. Record native validation status: run Android/iOS gates when available, or document why unavailable and what static checks covered.

## Platform/accessibility checklist

- [ ] No raw DOM elements/globals in shared code unless isolated in a platform-specific file.
- [ ] Shared UI uses RN primitives and Expo-compatible modules.
- [ ] Platform-specific files expose the same interface and stay thin.
- [ ] Layout works at 320 px, tablet widths, desktop widths, and orientation/window changes.
- [ ] Touch targets are at least 44 × 44 px.
- [ ] Keyboard avoidance, focus order, labels, and error states are handled.
- [ ] Safe areas are respected on top-level screens.
- [ ] Images use `expo-image`, set `contentFit`, and include accessibility labels where meaningful.
- [ ] Navigation uses `expo-router` and supports deep linking/shareable web routes.
- [ ] Sensitive storage uses `expo-secure-store`; non-sensitive persisted state uses approved async storage.
- [ ] New native dependencies include rebuild/development-build notes and validation gates.
- [ ] Feature docs state supported platforms, validation performed, known caveats, and responsive/accessibility notes.

## Default verification gates (examples)

Put real commands in `PROJECT.md`. Example gates for an Expo frontend with an optional backend service:

| Gate | Example command | Required when |
|---|---|---|
| install | `npm ci` | Dependencies are absent or changed. |
| typecheck | `npm run typecheck` | Always for code changes. |
| lint | `npm run lint` | Always for code changes. |
| unit/component | `npm test -- --ci` | Logic, hooks, adapters, components, screens. |
| web export/build | `npx expo export --platform web` | Routing, env, bundling, static web behavior. |
| web E2E | `python .github/skills/app-testing/scripts/with_server.py --server "npx expo start --web" --port 8081 -- python frontend/__tests__/e2e/smoke.py` | Changed user-facing flows. |
| Android smoke | `npx expo run:android` | Native dependencies or Android-specific behavior, when runner exists. |
| iOS smoke | `npx expo run:ios` | Native dependencies or iOS-specific behavior, when runner exists. |
| backend example | `<backend package test command from PROJECT.md>` | API/service changes supporting the app. |

## When this profile is N/A

Use a different profile when:

- The product is web-only and has no native mobile target.
- The mobile apps are separate native codebases rather than one shared codebase.
- The repository is service-only with no UI.
- A project-specific cross-platform profile supersedes these rules.
