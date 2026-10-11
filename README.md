# Little Days PWA — Firebase Realtime Database (no sign-in)

A mobile-first baby growth tracker and memory journal, installable on iPhone/iPad and Android. The app uses the configured Firebase Realtime Database directly without Firebase Authentication. Every install points to the same cloud record.

## Included
- Baby profile, age display, pastel themes, growth measurements and chart
- Milestone checklist and custom milestones
- Journal entries with optional profile/journal photos and voice recordings
- Local browser storage for offline use, plus JSON export/import backup
- Automatic cloud save after edits once the app has connected and a cloud/local copy choice has been made
- iOS/Android PWA manifest, icons, service worker and mobile layout

## Important: this database path is public
You asked for no sign-in and no database security. To make unauthenticated writes work, the Realtime Database rules below grant public read/write access to the `littleDaysBabyTracker` path. **Anyone who knows the project/database URL may read, overwrite, or delete the Little Days cloud record.** This includes baby profile details, measurements, milestone history, journal text, and any media embedded in the record. Do not use this setup for private information. Public rules can also be abused and may consume your Firebase quota.

## 1. Set Firebase Realtime Database rules
Open Firebase Console → **Realtime Database → Rules**. Add the `littleDaysBabyTracker` child to your existing `rules` object. Do not blindly replace unrelated rules if this Firebase project is already used by another app. Merge the block below into the existing rules JSON, keeping other app paths unchanged. If the path already exists, merge/adjust that path rather than duplicating it. Publish the rules.

```json
{
  "rules": {
    "littleDaysBabyTracker": {
      ".read": true,
      ".write": true
    }
  }
}
```

The app writes to `/littleDaysBabyTracker/state`. The rule makes that path and its children readable and writable without authentication. Avoid setting public read/write at the database root if other applications share this Firebase project. Existing parent-level rules may already expose other data; review the complete rules before publishing.

## 2. Run or deploy
For PWA installation, serve the files over `localhost` or HTTPS; opening `index.html` with `file://` will not enable service workers.

```bash
python -m http.server 8000
```

Open `http://localhost:8000` for local testing. For use on a phone, deploy the **entire folder** to an HTTPS static host and keep `index.html`, `manifest.webmanifest`, `sw.js`, and `icons/` together.

## 3. How cloud saving works
- When a valid cloud record exists and this device has no meaningful local data, the app restores the cloud record automatically.
- If both cloud and local records contain data, the app asks you to choose **Load cloud data** (replace this device's local data) or **Save this device to cloud** (replace the shared cloud record).
- After choosing, app edits automatically save to the shared record when online. While offline, the local copy continues to work and edits stay on the device until a later save succeeds.
- All devices share one record; this is not multi-user collaboration or a merge system. Avoid editing on multiple devices at the same time.
- The app stops a single cloud save if serialized data exceeds 8 MB. Remove large media clips or photos and try again. Local data is not removed when a cloud save fails.
- Use **Backup** to export a JSON file before replacing cloud or local data.

## Notes
- No Firebase Authentication or Firebase Storage is used; profile images and journal audio/photos are stored in the app state record, so larger media can grow the record quickly.
- Chart.js is loaded from a CDN, so chart rendering may need an internet connection even when the app shell is installed.
- If rules deny access, check that the `littleDaysBabyTracker` rule is published and that you used the right Firebase project.
