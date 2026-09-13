## 📖 Story / Why
Only Thapar students may order, and vendors/admins need provisioned accounts. This ticket builds the identity + role foundation everything else depends on: campus-gated login and role-based access.

## 🧭 Context
Firebase Auth. Students log in with **Google Sign-In restricted to the `thapar.edu` domain**. Roles (`student`/`vendor`/`admin`) are carried as **Firebase custom claims**, never in a client-writable field. Rules and functions are the source of truth for authorization. See `docs/ARCHITECTURE.md` §3 and `docs/DATA_MODEL.md` §security.

⚠️ **Blocked on PRD Open Decision D6** — confirm Thapar uses Google Workspace for `thapar.edu` so `hd=thapar.edu` Google Sign-In works. If not, fall back to email-link verification (also captured in the PRD).

## 🔑 Access & prerequisites
- Firebase Auth **Google provider** enabled and restricted to `thapar.edu`.
- The `setUserRole` function needs Admin SDK privileges — runs as a Cloud Function (no service-account key in the repo; use the Functions runtime identity / Firebase secrets).
- An admin account to test granting `vendor`/`admin` claims.

## ✅ Scope / What to build
- [ ] Google Sign-In (thapar.edu-restricted) on mobile (shared + Compose) and web.
- [ ] `onUserCreate` (Auth trigger): verify email domain, set default `student` claim, create `users/{uid}`.
- [ ] `setUserRole` (admin-only callable): grant `vendor`/`admin` claims and link a court where relevant.
- [ ] Firestore **security rules v1** implementing the role matrix in `docs/DATA_MODEL.md` §5.
- [ ] Auth-gated navigation: signed-out → sign-in screen; signed-in → role-appropriate landing (student app vs vendor web).

## 🎯 Acceptance Criteria
- [ ] A `@thapar.edu` account signs in on mobile **and** web, receives a `student` custom claim, and gets a `users/{uid}` doc.
- [ ] A non-`thapar.edu` account is rejected (no access).
- [ ] An admin can grant a `vendor` claim to a user via `setUserRole`.
- [ ] Security rules (verified against the emulator) deny cross-role access — e.g. a student cannot write another user's doc or any court.
- [ ] Role is read from the auth token claim, not from the `users` doc.

## 🚫 Out of scope
- Saved delivery addresses (later), profile editing beyond basics, vendor dashboard features (ticket 3), any ordering.

## 🔗 Dependencies
- Ticket 1 (scaffolding). Blocked by PRD **D6** confirmation.

## 📚 References
- `docs/DATA_MODEL.md` §5 (security matrix), `docs/FUNCTIONS.md` (`onUserCreate`, `setUserRole`), `docs/ARCHITECTURE.md` §3, PRD §7 D6.

## 🤖 Kickoff prompt (paste into Claude Code)
```
/start-ticket <this-issue-number>
```
