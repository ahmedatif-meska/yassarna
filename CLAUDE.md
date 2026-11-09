# Yassarna (يَسَّرْنَا)

A mobile app that teaches the meanings of selected Quranic words inside their
ayat, in short Duolingo-style lessons, for Arabic-speaking learners. The name
comes from Al-Qamar 54:17. Arabic-first (RTL) with an English
interface option.

## How we work

- Step by step: do only the step asked, report, then wait for the next one.
- Run `npx tsc --noEmit` and `npx expo lint` before calling any step done.
- Don't commit unless asked.

## Stack

Expo SDK 58, React Native, React 19, TypeScript (strict), Expo Router.
Expo changes every SDK release, so don't trust memory: read the `expo` major in
`package.json` and check the matching docs at
https://docs.expo.dev/versions/v58.0.0/ (index with LLM corrections:
https://docs.expo.dev/llms.txt) before using any Expo or React Native API.

```bash
npx expo install <package>   # always, instead of npm install: picks SDK-compatible versions
npx expo start               # dev server; open in Expo Go on the iPhone
npx tsc --noEmit             # typecheck
npx expo lint                # lint
npx expo-doctor              # dependency and config check
```

- Routes live in `src/app/` (every file is a screen, `_layout.tsx` defines
  navigators). Components, hooks and logic stay outside `src/app/`. Import
  `Link`, `router` and `useLocalSearchParams` from `expo-router`. The `@/*`
  alias points to `src/*`.
- `ios/` and `android/` are generated (Continuous Native Generation): never
  create or edit them; configure native behaviour in `app.json` and config
  plugins.
- Expo Go only has its bundled native modules. A library with other native code
  needs a development build (`npx expo run:ios|android` or
  `eas build --profile development`). Prefer Expo modules over third-party ones.
- Builds and releases use EAS: `npx eas-cli@latest <command>`.

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
  marks or generate it.
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
states never rely on colour alone (icon + word).
