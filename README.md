# Training Log

**You can't get better - or stay consistent - at something you can't track.**

A self-built weekly training tracker I use daily to log strength, VR cardio, and walking sessions, pulled from a Polar H10 chest strap. Built because I wanted zone-and-calorie data for diet decisions, without the all-day surveillance anxiety of a wrist-worn band.

**[Try it live →](https://training-log-ks.web.app/)** &nbsp;•&nbsp; Guest login: `guest` / `guest` - read-only demo data, no signup

---

## Why this exists

I train using VR cardio (Meta Quest 3) alongside strength work on a home pull-up bar. I wanted real heart-rate-zone and calorie data to plan my diet around - not a 24/7 wearable telling me my "readiness score" is low before I've even gotten out of bed. A Polar H10 chest strap solved the accuracy problem (ECG beats optical wrist sensors during fast VR movement). This app solved the *"now what do I do with that data"* problem.

<p align="center">
  <img src="https://github.com/user-attachments/assets/0bd16165-c43e-4dac-9667-34c2a2676650" width="220" alt="Polar H10 device">
</p>

Sessions happen in a fixed order - warmup, strength, VR cardio to exhaustion - across a rotation of games:

<p align="center">
  <img src="https://github.com/user-attachments/assets/7a9cc57a-65b6-4cda-8c24-e19b77071da0" width="270" alt="XR Workout">
  <img src="https://github.com/user-attachments/assets/72b61413-b967-41c0-a233-e72728fa0406" width="270" alt="Beat Saber">
  <img src="https://github.com/user-attachments/assets/aafa2d42-72f2-4c8e-922e-881c9d584b7b" width="270" alt="Thrill of the Fight 2">
</p>

I still use Polar Flow to review the raw session data - HR curve, zone breakdown, energy source split - before transcribing the numbers that actually matter into this tracker.

<p align="center">
  <img src="https://github.com/user-attachments/assets/d725265e-5652-487d-89b8-bc48efb6c853" width="160" alt="Polar Flow session summary">
  <img src="https://github.com/user-attachments/assets/b01056d9-de53-4c78-8d41-b7b01645ff4b" width="160" alt="Polar Flow training zones">
  <img src="https://github.com/user-attachments/assets/641ab05e-64a2-4eab-a46b-79b4fbd76f2f" width="160" alt="Polar Flow energy sources">
  <img src="https://github.com/user-attachments/assets/ea82e9ac-cee6-4936-a422-23b93339227e" width="270" alt="Polar Flow kcal/min graph">
</p>

## What it does

- **Daily session logging** - strength (warmup, pull-ups, bands), VR cardio (duration, app, HR, calories, zone), and walking, logged separately since they have different intensity profiles worth reviewing on their own.
- **Sunday-only body measurements** - weight, waist, stomach, and VO2max (retested every 3–4 weeks, not weekly - real change takes longer than a week to show up, so anything more frequent is just noise).
- **Weekly export** - auto-calculated totals (calories, VR minutes, walking minutes, avg HR) generated on demand, formatted for a quick weekly review instead of manual math.
- **Real login + guest mode** - username-or-email login backed by Firebase Auth, plus a fully sandboxed guest account that never touches real data, so this repo/demo can be shared publicly without exposing anything personal.
- **Settings** - change username, download a full JSON backup, clear a week or wipe everything (typed confirmation required for the destructive one).

<p align="center">
  <img src="https://github.com/user-attachments/assets/ed06cdb4-dea7-47a0-93d9-5ba2234028b2" width="260" alt="Login page">
  <img src="https://github.com/user-attachments/assets/c5b973d3-4572-4617-a42c-598a14b829b6" width="260" alt="Daily tracker view">
  <img src="https://github.com/user-attachments/assets/c1e4564f-fcef-481f-bfae-7d5f48a8d6a0" width="260" alt="Sunday measurements view">
</p>
<p align="center">
  <img src="https://github.com/user-attachments/assets/141778b7-f2ef-41cb-9da1-8dc39bdc3b71" width="260" alt="Settings page">
  <img src="https://github.com/user-attachments/assets/f60d5f73-498e-4f5a-a984-981980a45b48" width="260" alt="Weekly export output">
</p>

## Architecture

Vanilla HTML/CSS/JS - no framework, no build step. Deliberate choice: this is a single-user daily tool, and a build pipeline would have added complexity without adding anything I needed.

<p align="center">
  <img src="https://github.com/user-attachments/assets/8c405fa0-fe21-47ae-bad2-414f1f7d6eed" width="600" alt="Architecture flow diagram">
</p>

**Stack:**
- **Frontend** - plain JS, tab-based day selector, form fields synced to a per-week JSON object in memory
- **Auth** - Firebase Authentication (email/password), with a username→email lookup layer on top so login doesn't require typing an email
- **Database** - Cloud Firestore, one document per ISO week (`week_YYYY-MM-DD`), containing all seven days
- **Hosting** - Firebase Hosting, deployed via the Firebase CLI

**Data model:**
```
/training-log/{weekKey}   → { Mon: {...}, Tue: {...}, ... }
/usernames/{username}     → { email: "..." }
/profile/{uid}            → { username: "..." }
```

## Design decisions worth noting

The storage layer went through three iterations before landing on Firebase, and the reasoning behind each pivot is the more interesting part of this build than the final result:

1. **Browser's File System Access API, synced through a Google Drive folder.** Worked on desktop. On Android, Chrome's directory picker only exposes local device storage - cloud providers aren't selectable, by design, likely because the API's write model doesn't play well with cloud latency.
2. **Same approach, tried in Brave** (daily browser on both devices). Brave blocks the File System Access API outright, with no override flag - a deliberate privacy stance from their team, not a bug.
3. **Firebase Firestore.** Plain network requests under the hood, so it's identical across every browser and platform - no picker permissions, no platform-specific capability checks. Given two failed attempts at making a browser API do cross-device sync, betting on a purpose-built sync backend instead of continuing to patch around browser quirks was the right call.

**Security model:** Firestore rules gate all reads/writes behind `request.auth != null`, with the `usernames` collection allowing single-document lookups (not full listing) so the login page can resolve a username to an email before authentication completes. Guest mode is entirely client-side - it never calls Firebase at all, so there's structurally no path from the public demo credentials to real data, rather than relying on a rule to block it after the fact.

## What tracking actually changed

Reviewing weekly instead of daily stopped the tracker from becoming another thing to obsess over - the whole reason I skipped wrist-worn continuous tracking in the first place. Separating VR, strength, and walking into distinct blocks made it obvious which sessions were doing what: a two-hour VR session at Zone 1 burns real calories from fat but doesn't move VO2max, while a 30-minute high-intensity session does the opposite. Without logging both side by side, that distinction would've stayed invisible.

## Setup (if forking this)

1. Create a Firebase project → enable Firestore + Email/Password Auth.
2. Add the Firestore rules from this repo.
3. Paste your config into the placeholder block in `index.html`.
4. `firebase init hosting` → `firebase deploy --only hosting`.

## License

This project is open source under the MIT License.
You are free to use, modify, and distribute this project for personal or commercial purposes. Attribution appreciated but not required.

## Author

**Kewal Shah**

B.E. in Information Technology, self-taught builder, and someone who finds it very difficult to stop once something interesting enough is in front of them.

This tracker is not a single-skill project. It required designing a real authentication system, writing Firestore security rules that actually hold up, modeling data around a weekly review habit instead of a database schema, and debugging across three different storage approaches - including chasing down why a browser API worked on one device and silently failed on another. I did not have most of these skills when I started logging my first workout by hand in a spreadsheet. Wanting a tool that actually fit how I train is how I acquired them.

I work across embedded systems, cybersecurity, networking, and hardware design. Not by design - by following the next problem wherever it leads. Cybersecurity is where I am going deep right now, holding CompTIA Network+ (814/900) and preparing for Security+. This project is a smaller reflection of that direction - a personal tool, but built with the same instinct: real login over a fake one, security rules that are actually enforced, and a guest mode that is sandboxed by design rather than by promise, not because a fitness tracker demanded it, but because that's how I default to building things now.

It sits alongside [CIPHER](#) - a custom handheld built from scratch with a hidden Kali Linux layer - as one of the smaller, faster builds I make between the longer ones. If CIPHER is the deep dive, this is the daily habit that keeps the discipline behind it intact.

| | |
|---|---|
| GitHub | [@Work-KewalShah](https://github.com/Work-KewalShah) |
| LinkedIn | [Kewal Shah](https://www.linkedin.com/in/kewal-shah-work/) |

---

*You can't get better - or stay consistent - at something you can't track. This is how I track it.*

If this build helped you or gave you ideas for tracking your own training, consider leaving a ⭐ on the repository.
