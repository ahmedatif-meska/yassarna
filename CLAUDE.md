# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Yassarna (يَسَّرْنَا)

A mobile app that teaches the meanings of selected Quranic words inside their
ayat, in short Duolingo-style lessons, for Arabic-speaking learners. The name
comes from Al-Qamar 54:17. Arabic-first (RTL) with an English interface option.
It will ship on the Apple App Store and Google Play.

## How we work

- Step by step: do only the step asked, report, then wait for the next one.
- Run `npx tsc --noEmit` and `npx expo lint` before calling any step done.
- Don't commit unless asked.

## Stack and commands

Expo SDK 58 (React Native 0.88, React 19, iOS 16.4+, Android target SDK 36),
TypeScript strict, Expo Router. Expo changes every SDK release, so don't trust
memory: check https://docs.expo.dev/versions/v58.0.0/ (index with LLM
corrections: https://docs.expo.dev/llms.txt) before using any Expo or React
Native API.

```bash
npx expo install <package>   # always, instead of npm install: picks SDK-compatible versions
npx expo start               # dev server; scan the QR with Expo Go on the iPhone
npx tsc --noEmit             # typecheck
npx expo lint                # lint (eslint-config-expo, flat config)
npx expo-doctor              # dependency and config check
npx eas-cli@latest build     # store builds (EAS); `submit` uploads them
```

No test runner is set up yet (Expo's guide: https://docs.expo.dev/develop/unit-testing/).

## Architecture

The code is still the `create-expo-app` starter; it will be replaced screen by
screen. What holds across files:

- **Routing.** `src/app/` is file-based routing: every file is a screen,
  `_layout.tsx` defines navigators. The root layout renders `AppTabs`. Keep
  components, hooks and logic outside `src/app/`. Aliases: `@/*` → `src/*`,
  `@/assets/*` → `assets/*`. Typed routes are on (`experiments.typedRoutes`),
  generated into `.expo/types/`; delete `.expo/` if stale routes break
  typecheck.
- **Platform files.** `name.web.tsx` overrides `name.tsx` on web. Tabs use
  native tabs (`expo-router/native-tabs`) on iOS/Android and `expo-router/ui`
  headless tabs on web. `src/global.css` only affects web.
- **React Compiler is on** (`experiments.reactCompiler`), and the lint rules
  enforce it: don't read `ref.current` during render (create Animated values
  and PanResponders with a lazy `useState(() => …)`), don't call `setState`
  directly in an effect, keep render pure (no `Math.random` in render).
- **Native code.** `ios/` and `android/` are generated (Continuous Native
  Generation): never create or edit them; configure native behaviour in
  `app.json` and config plugins. Expo Go only has its bundled native modules;
  anything else needs a development build (`npx expo run:ios|android` or
  `eas build --profile development`). Prefer Expo modules over third-party ones.
- `npm run reset-project` moves the starter code to `example/` and leaves a
  blank `src/app`.

## Screen spec: what we build

**Yassarna — Sign-up to First Lesson**: https://claude.ai/artifact/WUNUWTidQHhEk5HYPBpZRh

A design canvas of 33 phone screens (390×844, light and dark) built on the
design system. It is the spec for every screen, flow and piece of copy. Read
`project/canvas.json` for the screen list and flow, then the screen's
`project/<Name>.dc.html` (and `project/screens.css`) before building it. Its
meanings and explanations are sample copy: real content must be reviewed and
sourced.

- 01–06 Start and account: opening animation, language, welcome (start
  without an account / create account / I have an account), email, 6-digit
  code, account ready.
- 07–20 First lesson with an account (Al-Kawthar): preview, flashcard, true or
  false, multiple choice, tap the word, complete the meaning, matching,
  optional speed round, repair (a missed word returns), result, Home.
- 21–28 First lesson without an account (Al-Ikhlas), same activity types.
- 29–33 Guest ending: result, save progress by email + code, saved, or
  continue on this device to a guest Home.

## Design system: the single source for all UI

**Yassarna Design System**: https://claude.ai/artifact/V2sThMkg5W4Nu6nNqRhfGw

Before any UI work, read its `project/README.md` and `project/tokens.json`
(Artifact tool, `read` with `paths`), and the component's
`project/components/<Name>/README.md` when building one. Where older code
disagrees, the design system wins.

- **Tokens, never raw values.** Map its colour tokens to a `{light, dark}` theme
  (`bg`, `surface`, `surface-ayah`, `ink`, `ink-muted`, `primary`, `accent`,
  `success`, `danger`, ...), and use its `space-*`, `radius-*`, `size-*` and
  `shadow-*` values. No hex codes or magic numbers in components.
- **Fonts.** Readex Pro (Arabic + Latin files) for all interface text, Amiri
  Quran for Quran text only. Load the font files from the design system's
  `project/fonts/` with `expo-font`.
- **Components.** Build screens from its components: ActionBar, AnswerOption,
  AyahCard, Button, CodeInput, Explanation, FeedbackSheet, Flashcard, Hoopoe,
  HoopoeBubble, InfoList, LessonTopBar, MatchTile, Notice, QuranWord,
  RecitationButton, RewardChip, StatTile, TextField, TrueFalse. Props are in
  `project/components/index.d.ts`.
- **Feel.** Pressable controls are raised on a darker edge (`*-edge` tokens)
  and press down with a light haptic; reading surfaces are flat. No gradients
  or glows. Lively around the learner, calm around the Quran.

## Quran text: non-negotiable

- Use the approved Uthmani Hafs text verbatim: never retype, normalise, strip
  marks or generate it. (This is also App Store 1.1.5: no inaccurate or
  misleading quotations of religious texts.)
- Only in the `quran-*` styles (Amiri Quran, line height 2x). Never bold,
  italicise, letter-space, truncate, animate or recolour it, even on a wrong
  answer.
- Quran text sits only on `surface-ayah`; no mascot, points or confetti on it.
- Always show the reference (سورة الكوثر · الآية ١). Explanations are never in
  the Quran face and always name their source.
- A taught word is highlighted by its occurrence (surah, ayah, position), never
  matched by text alone.

## Language, RTL and copy

- Arabic is the default and right-to-left; English is an interface option.
  Ayat, taught words and meanings stay Arabic in both. Use start/end, never
  left/right.
- Arabic UI uses Arabic-Indic digits (٠١٢٣); wrap signed numbers and times in
  an LTR isolate. Use the Arabic comma (،) and question mark (؟).
- Warm Modern Standard Arabic, no dialect. The hoopoe speaks as "we"
  (لنراجعها معًا). Buttons start with a verb (ابدأ الدرس، تحقّق، تابع).
- Feedback teaches, never punishes: correct is أحسنت; wrong is ليس تمامًا plus
  the right answer and why. A wrong answer never loses points.
- Points are نقاط in Arabic (not "XP"). No emoji. No reward language attached
  to worship.

## Accessibility

Text 4.5:1 and controls 3:1 in both themes; touch targets at least
`size-touch`; every gesture (swipe, flip, drag) has a tap alternative; timed
rounds can be played untimed; support large text and reduced motion; result
states never rely on colour alone (icon + word). Only claim an Apple
Accessibility Nutrition Label feature (VoiceOver, Larger Text, Sufficient
Contrast, Differentiate Without Color, Reduced Motion) if every common task
works with it.

## App Store and Google Play rules

Checked 9 Nov 2026 against Apple's App Review Guidelines (updated 16 Oct 2026)
and Google Play's Developer Program Policy. Numbers like `5.1.1(v)` are Apple
guideline sections. Re-check the sources before each submission; deadlines move.

### Privacy and data

- A privacy policy is required on both stores, linked in the store console AND
  reachable inside the app. It must list what data is collected, how, every
  use, third parties, retention and deletion, how to withdraw consent, and a
  privacy contact. Google: public HTML page titled "Privacy Policy", not a PDF,
  not geofenced. A working support URL and in-app contact are also required.
  (5.1.1(i), 1.5)
- Guest learning must work without an account; never force login for features
  that don't need one. Ask only for data the feature needs; contact details are
  optional. (5.1.1(v), 5.1.1(x))
- Get consent before collecting analytics, even anonymous, with an easy way to
  withdraw it (an analytics on/off control). Google: an in-app disclosure right
  before collection, not only in the policy. (5.1.1(ii))
- No tracking: no ad SDKs, no linking app data with third-party data. Keep
  `NSPrivacyTracking` false and don't show the tracking prompt. Adding tracking
  later requires App Tracking Transparency. (5.1.2)
- Never reuse data for a new purpose, or send it to a third party (including
  AI services), without disclosure and fresh consent. (5.1.2)
- Ask for permissions only when needed, in context, with clear purpose strings;
  the app must keep working if the user says no. Launch needs no camera,
  microphone or location. (5.1.1(iii)-(iv))
- Keep store disclosures true and current: Apple App Privacy answers and
  Google Data safety form, including data collected by any SDK. Expected types:
  email, user ID / alias, product interaction, report text, crash data; linked
  to the user, not used for tracking. A wrong form can block updates.

### Account deletion

- If accounts exist, deletion must start inside the app (account settings),
  delete the account and its personal data (deactivation doesn't count), and
  include guest/anonymous accounts and anything shared publicly (aliases). No
  email or support call required; a confirmation code step is fine. Tell the
  user how long it takes and confirm when done. Same flow in every country.
  (5.1.1(v))
- Google also requires a public web page where users can request deletion
  without the app; its URL goes in the Play Console data deletion questions.
  Deletion must reach every backend provider that holds the data; disclose any data kept for
  legal reasons.
- Adding Sign in with Apple: revoke its tokens on deletion. Adding
  subscriptions: explain that Apple billing continues and offer immediate
  deletion.

### Content, rights and user-generated content

- Hold a licence for every third-party work (recitation audio, tafsir,
  translations) and keep proof; attach it in App Review notes. Never let users
  download third-party media without authorisation. (5.2.1-5.2.3)
- No inflammatory or mean-spirited religious commentary. (1.1.1, 1.1.5)
- Public aliases, leaderboards or any user text count as user-generated
  content: filter objectionable names, provide in-app report and block, act on
  reports promptly, and (Google) have users accept terms first. (1.2)
- Never claim to be an official app of an organisation. Don't copy another
  app's name, look or brand. (4.1)

### Review readiness

- Submit only finished builds: no crashes, placeholders or dead buttons, and
  the backend live during review. (2.1)
- Give reviewers a working login: email + one-time code can't be received by a
  reviewer, so provide a review account with a fixed code (Google: "app access"
  instructions in Play Console). Explain how to reach every feature in Review
  Notes. (2.1)
- The app must offer real lasting value, not a wrapped website. Ask before
  downloading large extra content (offline audio) and show its size. (4.2,
  4.2.3)
- Public APIs only; no downloaded code that changes features; background modes
  only for their purpose (`audio` for recitation playback). Must work on
  IPv6-only networks. (2.5.1, 2.5.2, 2.5.4, 2.5.5)
- Widgets and notifications only about the app's own content. Notifications
  are never required to use the app; promotional ones need explicit opt-in and
  an in-app opt-out. (2.5.16, 4.5.4) Android 13+: request `POST_NOTIFICATIONS`
  at a moment the user chose, not on first launch, and create the notification
  channel first.
- Use only the system review prompt (`expo-store-review`), never a custom one.
  (5.6.1)

### Login, payments and age

- Email + code is our own account system and needs nothing else. If Google
  sign-in is added, also offer Sign in with Apple (`ios.usesAppleSignIn`). (4.8)
- No ads or purchases at launch. If digital content or subscriptions are ever
  sold: Apple In-App Purchase with restore (subscriptions at least 7 days, what
  the user gets shown before buying) and Google Play Billing; never point users
  to other payment methods. (3.1.1, 3.1.2)
- General audience, not a kids app: never say "for kids" or market to under-13s
  (Google may treat a cartoon mascot in marketing as child-directed; keep the
  hoopoe in store art adult-friendly). Any target age group that includes
  children brings in Google's Families policy. (2.3.8, 5.1.4)
- Answer Apple's age rating questionnaire (4+/9+/13+/16+/18+) and Google's IARC
  questionnaire truthfully, and redo them when features change (leaderboards
  can count as contests; an open in-app web browser can raise Apple's rating to
  16+). Age-assurance laws (Utah, Louisiana, Texas) may require Apple's Declared
  Age Range API or Google's Age Signals API: get legal review before launch.

### Store listing

- Name at most 30 characters, no keyword stuffing, no emoji or ranking/price
  claims in the title. Screenshots show the real app in use, not just splash or
  login. Describe only features that exist; icon and screenshots suitable for
  4+. (2.3.1, 2.3.3, 2.3.7, 2.3.8)
- Apple: declare EU trader status (Digital Services Act) or the app is removed
  from EU storefronts.

### Build and config requirements

- Apple: built with Xcode 26+ and the iOS 26 SDK (since 28 Apr 2026); EAS with
  SDK 58 meets this.
- Google: target API 36 (required since 31 Aug 2026; SDK 58 default) and
  16 KB memory page support (updates blocked without it from 1 Feb 2027; check
  the AAB with `bundletool dump config`). Register the app for Android
  developer verification (from 30 Sep 2026).
- New personal Google Play account: run a closed test with at least 12 testers
  opted in for 14 consecutive days before applying for production.
- Privacy manifest: declare required-reason APIs in `app.json`
  `ios.privacyManifests` (`NSPrivacyAccessedAPITypes`, with reasons such as
  `CA92.1` UserDefaults, `C617.1` file timestamps, `35F9.1` boot time, `E174.1`
  disk space; `1C8F.1` if a widget shares UserDefaults), `NSPrivacyTracking`
  false and `NSPrivacyCollectedDataTypes`. Expo packages' own manifests aren't
  read reliably, so copy their reasons in (https://docs.expo.dev/guides/apple-privacy/).
  Upload to TestFlight to verify: Apple emails any missing reason.
- Third-party SDKs on Apple's commonly-used list must ship their own privacy
  manifest and signature.
- Export compliance: set `ios.config.usesNonExemptEncryption: false` (HTTPS only).
- Purpose strings and background modes go in `ios.infoPlist` (Expo doesn't
  validate them). Strip permissions libraries add but we don't use with
  `android.blockedPermissions`. Increase `android.versionCode` every release.

### Sources

- App Review Guidelines: https://developer.apple.com/app-store/review/guidelines/
- Apple account deletion: https://developer.apple.com/support/offering-account-deletion-in-your-app/
- Apple App Privacy details: https://developer.apple.com/app-store/app-privacy-details/
- Apple privacy manifests: https://developer.apple.com/documentation/bundleresources/privacy-manifest-files
- Apple upcoming requirements: https://developer.apple.com/news/upcoming-requirements/
- Google User Data policy: https://support.google.com/googleplay/android-developer/answer/10144311
- Google Data safety: https://support.google.com/googleplay/android-developer/answer/10787469
- Google account deletion: https://support.google.com/googleplay/android-developer/answer/13327111
- Google target API: https://support.google.com/googleplay/android-developer/answer/11926878
- Google testing for new accounts: https://support.google.com/googleplay/android-developer/answer/14151465
- Google target audience / Families: https://support.google.com/googleplay/android-developer/answer/9867159
- Android 16 KB pages: https://developer.android.com/guide/practices/page-sizes
- Expo app config: https://docs.expo.dev/versions/v58.0.0/config/app/
