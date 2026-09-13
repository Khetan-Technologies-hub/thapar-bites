## 📖 Story / Why
Establish the Kotlin Multiplatform monorepo so every later ticket has a foundation to build on. This ticket creates the empty-but-running skeleton for all targets — nothing user-facing yet, just the structure, build, and Firebase wiring described in `CLAUDE.md` and `docs/ARCHITECTURE.md`.

## 🧭 Context
Greenfield repo. Target layout (from `CLAUDE.md`): `shared/` (KMP), `androidApp/`, `iosApp/`, `web/`, `functions/`, `firebase/`, `docs/`. Stack is locked: Kotlin Multiplatform + Compose Multiplatform (Android/iOS), **Koin** DI, **MVVM** with shared ViewModels exposing `StateFlow`, Firebase backend via the **GitLive Firebase Kotlin SDK**, Cloud Functions in **TypeScript**, web in **React + TypeScript** on the Firebase JS SDK.

⚠️ **iOS builds require macOS + Xcode** — the iOS portion of this ticket must be done on a Mac. Android, web, and functions can be scaffolded on any OS.

## 🔑 Access & prerequisites
- A **Firebase project** (dev, e.g. `thapar-bites-dev`) created in the Firebase console with Auth, Firestore, Functions, Storage, Hosting enabled. Download configs and place them per `docs/SETUP.md` — **do not commit them** (`.gitignore` already excludes them).
- Tooling per `docs/SETUP.md`: JDK 17+, Android Studio (+ KMP plugin), Node 20+, Firebase CLI; Xcode + CocoaPods on the Mac.
- No secrets in the repo.

## ✅ Scope / What to build
- [ ] Gradle KMP project with modules: `shared`, `androidApp`, `iosApp`, `web`, `functions`, `firebase`.
- [ ] `shared/`: Koin DI setup, a base `StateFlow`-driven ViewModel abstraction, and one placeholder domain model — enough to prove the shared → UI pattern.
- [ ] `androidApp/` + `iosApp/`: Compose Multiplatform entry points rendering a placeholder screen driven by a shared ViewModel.
- [ ] `web/`: React + TypeScript skeleton (Vite) with Firebase JS SDK initialized, rendering a placeholder page.
- [ ] `functions/`: TypeScript Cloud Functions project initialized (strict mode), with one trivial callable to prove deploy/emulator.
- [ ] `firebase/`: `firebase.json`, locked-down default `firestore.rules` (deny-all baseline), empty `firestore.indexes.json`, emulator config.
- [ ] GitHub Actions CI skeleton: build + lint for shared/android, web, functions.

## 🎯 Acceptance Criteria
- [ ] Android app builds and runs on an emulator, showing the placeholder from a shared ViewModel.
- [ ] iOS app builds and runs on the simulator (on macOS), showing the same placeholder.
- [ ] `web` runs via `npm run dev` and renders its placeholder, with Firebase initialized (no console errors).
- [ ] `functions` builds; `firebase emulators:start` runs Auth + Firestore + Functions locally.
- [ ] CI passes on push.
- [ ] No config/secret files are committed.

## 🚫 Out of scope
- Any real features, auth, data models, or UI beyond placeholders.

## 🔗 Dependencies
- None — this is the first ticket.

## 📚 References
- `CLAUDE.md`, `docs/ARCHITECTURE.md` (§2 components, §4.1 stack), `docs/SETUP.md`.

## 🤖 Kickoff prompt (paste into Claude Code)
```
/start-ticket <this-issue-number>
```
