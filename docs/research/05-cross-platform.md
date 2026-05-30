# Cross-platform build best practices

Purpose: guidance for a generic framework that can generate apps for web only or for web + Android + iOS from one codebase, with agents aware of platform pitfalls and validation gates.

## Recommended default

Use **Expo + React Native + Expo Router** as the default cross-platform stack for TypeScript-first product apps that need web, iOS, and Android from one repo.

Why:
- Expo has first-class web support; `npx expo start --web` for development and `npx expo export --platform web` for production web export ([Expo web](https://docs.expo.dev/workflow/web/)).
- Expo Router provides file-based routing for Android, iOS, and web with universal deep links and platform-optimized navigation ([Expo Router](https://docs.expo.dev/router/introduction/)).
- React Native platform-specific files (`.ios`, `.android`, `.native`) and `Platform` module are official escape hatches ([React Native](https://reactnative.dev/docs/platform-specific-code)).
- Expo development builds support production-grade native modules beyond Expo Go ([development builds](https://docs.expo.dev/develop/development-builds/introduction/)).

## Stack comparison

| Stack | Best for | Strengths | Weaknesses / cautions |
|---|---|---|---|
| Expo / React Native | Product apps needing native mobile + web with shared TS/React | Native UI primitives, Expo SDK, OTA/development builds, web via React Native Web, file routing | Must avoid DOM-only APIs in shared code; native module compatibility; web SEO needs care. |
| Flutter | Highly custom UI across mobile/web/desktop | Single Dart UI toolkit; web is “another device target”; WebAssembly support; strong visual consistency ([Flutter web](https://docs.flutter.dev/platform-integration/web)) | Web is best for app-like SPAs; text-heavy SEO/document sites less ideal per Flutter docs. |
| Capacitor | Web-first teams wrapping an existing web app for mobile | Modern native container; “if it works in browser, probably works in mobile”; native plugins ([Capacitor](https://capacitorjs.com/docs)) | UI remains webview-based; native feel/performance depends on web app; mobile-specific UX must be added. |
| Tauri | Tiny secure desktop/mobile apps with web UI + Rust/native backend | Small binaries via system webview; any frontend; Rust safety; mobile support ([Tauri](https://tauri.app/start/)) | More desktop-oriented mental model; mobile ecosystem less mainstream than Expo/Flutter. |
| Kotlin Multiplatform | Shared domain/data logic with native UIs | Strong for sharing business logic across Android/iOS; native UI per platform | Not a one-codebase UI solution unless using Compose Multiplatform; web story differs. |
| Web-only React/Next | Web SaaS, SEO/content-heavy apps | Best web ecosystem, SSR/SEO, fastest hiring | Not native mobile; mobile apps require wrapper or separate native work. |

## Expo / React Native rules for generated code

### 1. Shared UI uses RN primitives

- Use `View`, `Text`, `Pressable`, `ScrollView`, `FlatList`, `Image`/`expo-image`.
- Expo notes React Native for Web wraps primitives like `View` and `Text`; raw DOM components like `<p>` can be web-only and will not render on native ([Expo web](https://docs.expo.dev/workflow/web/)).
- Avoid `<div>`, `<span>`, `<button>`, `<a>` in shared components.
- If DOM is unavoidable, isolate it in `.web.tsx` or Expo DOM components; Expo DOM components are a WebView bridge and have async/serializable prop constraints ([Expo DOM components](https://docs.expo.dev/guides/dom-components/)).

### 2. No direct browser globals in shared code

Avoid direct `window`, `document`, `localStorage`, `navigator`, `location`, `File`, raw DOM events, or CSSOM in shared modules.

Use platform-safe abstractions:
- Storage: AsyncStorage for normal data; SecureStore/keychain-style APIs for secrets.
- Linking/navigation: Expo Router and Expo Linking, not browser anchors.
- Files/images: Expo DocumentPicker/ImagePicker/FileSystem where supported.
- Dimensions: `useWindowDimensions()` or responsive hooks; React Native docs prefer `useWindowDimensions` because dimensions can change on rotation/foldables ([Dimensions](https://reactnative.dev/docs/dimensions)).

### 3. Platform splits are thin

- Use `Platform.OS` / `Platform.select` for tiny differences.
- Use `.web.tsx`, `.native.tsx`, `.ios.tsx`, `.android.tsx` for larger differences ([React Native platform-specific code](https://reactnative.dev/docs/platform-specific-code)).
- In Expo Router `app/`, platform-specific route files require a non-platform base route so routes remain universal for deep linking ([Expo platform-specific modules](https://docs.expo.dev/router/advanced/platform-specific-modules/)).
- Keep split files as wrappers around shared logic; do not fork whole features unless necessary.

### 4. Responsive layout by default

- Start mobile-first; support 320px phone through desktop widths.
- Use flexbox, wrapping, gap, maxWidth, percentage widths.
- Use `useWindowDimensions()` for adaptive layouts; do not cache dimensions globally.
- Avoid fixed content widths; use `maxWidth` with `width: '100%'`.
- Avoid absolute positioning for layout; reserve for overlays.
- Test landscape, tablets, desktop browser resize, and foldable/window changes when possible.

### 5. Touch and input parity

- Touch targets should be at least ~44x44 px.
- Mouse-only hover/right-click interactions need tap/keyboard alternatives.
- React Native touch handling centers on `Button`, `Pressable`/Touchables, scroll views, and gesture responder patterns ([touch docs](https://reactnative.dev/docs/handling-touches)).
- Forms need keyboard avoidance and visible focus/error states.
- Validate on mobile keyboard, hardware keyboard, screen reader labels, and browser tab navigation.

### 6. Navigation and routing

- Prefer Expo Router for universal file-based navigation; every route can be deep-linkable/shareable and works across platforms ([Expo Router](https://docs.expo.dev/router/introduction/)).
- Avoid direct `react-navigation` setup in starter templates unless exposed behind Expo Router.
- Avoid raw `<a href>` in shared app UI; use router `Link`/imperative navigation abstractions.
- On web, decide early whether app needs static rendering/SEO or client-only behavior.

### 7. Images/assets/fonts

- Use Expo-compatible image handling; always provide accessibility labels for meaningful images.
- Set `contentFit`/resize behavior explicitly.
- Load custom fonts through Expo font tooling; avoid platform-only font family assumptions.
- Do not rely on public web asset paths in native code; Expo DOM docs warn public assets in DOM components have limitations with EAS Update ([DOM components](https://docs.expo.dev/guides/dom-components/)).

### 8. Native dependencies

- Expo Go is limited to native libraries bundled into Expo Go; production-grade apps with extra native libraries need development builds ([development builds](https://docs.expo.dev/develop/development-builds/introduction/)).
- After adding native libraries or config plugins, rebuild native app (`npx expo run:android` / `npx expo run:ios`); JS-only changes can use `npx expo start` after first build ([local app development](https://docs.expo.dev/guides/local-app-development/)).
- Agents should detect native dependency additions and update validation gates accordingly.

## Testing matrix

| Scope | Web | Android | iOS |
|---|---|---|---|
| Static | `tsc`, ESLint, Jest | Same | Same |
| Component | React Native Testing Library with `jest-expo` | Same mocked native layer | Same mocked native layer |
| Web E2E | Playwright against `npx expo start --web` | N/A | N/A |
| Native smoke | N/A | `npx expo run:android` or dev build + emulator; Maestro/Detox optional | `npx expo run:ios` on macOS or EAS/dev build |
| Manual/agent exploratory | Browser + responsive sizes | Emulator/device if available | Simulator/device if available |

Expo recommends `jest-expo` for Jest configuration and `@testing-library/react-native` for components ([Expo unit testing](https://docs.expo.dev/develop/unit-testing/)).

## Agent validation checklist for cross-platform PRs

- [ ] No raw DOM elements/globals in shared code unless isolated.
- [ ] Platform-specific files have shared interfaces and minimal diffs.
- [ ] Layout tested/inspected at narrow phone, tablet, desktop widths.
- [ ] Touch targets and keyboard behavior considered.
- [ ] Navigation uses Expo Router abstractions.
- [ ] New native dependency triggers development build instructions/gates.
- [ ] Web E2E smoke exists for new important flow.
- [ ] Component/unit tests cover core logic and UI states.
- [ ] README documents platforms validated and known gaps.

## When not to choose Expo default

- Choose **web-only React/Next** for SEO-heavy sites, content publishing, landing pages, dashboards with no native mobile plan.
- Choose **Capacitor** when an existing production web app should be shipped quickly as mobile and native UX demands are modest.
- Choose **Flutter** when pixel-perfect custom UI and consistent rendering matter more than React/web ecosystem reuse.
- Choose **Kotlin Multiplatform** when native platform teams want shared business/data layers but keep native UI.
- Choose **Tauri** for desktop-first apps needing small binaries and native OS integration with web UI.

## Lessons for our framework

1. Make platform target explicit in PRD: `web-only`, `universal-web-mobile`, `native-mobile-first`, or `desktop`.
2. Default starter template should be **Expo + React Native + Expo Router + TypeScript + jest-expo + Playwright web smoke**.
3. Add a cross-platform linter/review checklist that blocks DOM globals and raw DOM tags in shared code.
4. Generate platform adapters for storage, linking, files, images, and environment access instead of letting features import platform APIs directly.
5. Require platform-specific files to be thin and documented; large divergence should be a plan-level decision.
6. Include dev commands in templates: `npx expo start --web`, `npx expo start`, `npx expo run:android`, `npx expo run:ios`, `npm test`, `npm run e2e`.
7. Use Playwright for web verification because it is deterministic and CI-friendly; use native smoke gates opportunistically based on available runners.
8. Document native dependency implications: Expo Go vs development build, rebuild triggers, and Android/iOS toolchain requirements.
9. For generic publishability, expose alternatives as templates, but keep one blessed default to avoid agent indecision.
10. Every generated feature README should state: supported platforms, validation performed, unsupported platform caveats, and responsive/accessibility notes.
