# Local Development Setup — Thapar Bites

> How to get the project running locally. This is the target setup once scaffolding exists (see [ROADMAP.md](ROADMAP.md)); some steps become concrete as the first tickets land.

---

## 1. Prerequisites

| Tool | Purpose | Notes |
|---|---|---|
| **JDK 17+** | Kotlin Multiplatform build | Temurin recommended |
| **Android Studio** (latest) | Android app, KMP tooling | Install the Kotlin Multiplatform plugin |
| **Xcode** (on macOS) | iOS app | Required only to build/run iOS |
| **Node.js 20+** | Web app + Cloud Functions | |
| **Firebase CLI** | Emulators, deploy | `npm i -g firebase-tools` |
| **CocoaPods** (on macOS) | iOS Firebase deps | `sudo gem install cocoapods` |

> iOS builds require macOS + Xcode. Android and web can be developed on Windows/Linux/macOS.

---

## 2. Firebase project setup

1. Create a Firebase project (e.g. `thapar-bites-dev`) in the Firebase console.
2. Enable: **Authentication** (Google provider), **Firestore**, **Cloud Functions**, **Cloud Messaging**, **Storage**, **Hosting**.
3. Configure **Google Sign-In** and restrict to the `thapar.edu` hosted domain *(pending PRD Open Decision D6 — confirm Thapar uses Google Workspace)*.
4. Register apps and download configs:
   - Android → `google-services.json` → `androidApp/`
   - iOS → `GoogleService-Info.plist` → `iosApp/`
   - Web → copy the web config into the web app's env.
5. **Never commit** these config files or any secrets — see §6.

---

## 3. Repository layout

```
thapar-bites/
├── shared/            # KMP shared module
├── androidApp/
├── iosApp/
├── web/
├── functions/         # Cloud Functions
├── firebase/          # firestore.rules, firestore.indexes.json, firebase.json
└── docs/
```

---

## 4. Running each piece

### Firebase emulators (do this first)
```bash
firebase emulators:start
```
Runs Auth, Firestore, Functions, and Storage locally so you never touch prod data in dev. Point clients at the emulator host in debug builds.

### Shared module + Android
Open the project in Android Studio, let Gradle sync, then run the `androidApp` configuration on an emulator or device.

### iOS (macOS)
```bash
cd iosApp && pod install
```
Open the `.xcworkspace` in Xcode and run.

### Web app
```bash
cd web
npm install
npm run dev
```

### Cloud Functions
```bash
cd functions
npm install
npm run build      # if TypeScript
firebase emulators:start --only functions
```

---

## 5. Seeding dev data

Once scaffolding exists, a seed script (`firebase/seed.*`) will populate the emulator with:
- a couple of demo courts with menus, hours, and payment settings,
- a demo vendor and admin account,
- a demo student.

Run it against the emulator, never prod.

---

## 6. Secrets & config hygiene

- Razorpay **test** keys for dev; store via Firebase secrets/config, not in code.
- `.gitignore` must exclude: `google-services.json`, `GoogleService-Info.plist`, web `.env*`, any `*serviceAccount*.json`, Firebase secret files.
- Use the **Razorpay test mode** and Firebase **emulators** for all local work — no real money, no prod writes.

---

## 7. Common tasks

| Task | Command |
|---|---|
| Start everything locally | `firebase emulators:start` |
| Deploy rules only | `firebase deploy --only firestore:rules` |
| Deploy functions | `firebase deploy --only functions` |
| Deploy web | `firebase deploy --only hosting` |
| Run functions tests | `cd functions && npm test` |

> Deploy targets and CI will be finalized in the scaffolding ticket. Until then, treat this as the intended shape.
