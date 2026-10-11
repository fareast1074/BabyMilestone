# Little Days — Baby Tracker PWA

A gentle, mobile-first baby growth tracker, milestone checklist, and memory journal. The app source and UI are contained in `index.html`; the manifest, service worker, and icons are the small support files required for an installable Progressive Web App. It is ready for static hosting on GitHub Pages and supports iOS Safari and Android Chrome.

## Included

- Live baby age counter, profile picture, and five pastel themes
- Growth log with weight/height chart and historical measurement deletion
- Developmental milestones across Motor Skills, Language / Communication, Social / Emotional, and Cognitive, plus custom milestones
- Journal memories with text, compressed photos, and voice recordings with a live waveform and playback preview
- Firebase Realtime Database synchronization without sign-in; data changes sync automatically and remote changes are applied in real time
- Local-device storage for offline use, plus JSON backup export/import and demo data
- PWA manifest, service worker shell caching, safe-area support, mobile bottom navigation, and install guidance for iOS/Android

## Firebase setup (one time)

The app uses the Firebase project configured in `index.html` and writes to `/littleDaysBabyTracker/state`. No Firebase Authentication is used. In Firebase Console, open **Realtime Database → Rules** and add the `littleDaysBabyTracker` child inside your existing rules object. Do not remove other rules if the project is shared with another app. Merge this block into the existing `rules` object, then publish:

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

The example above shows the complete shape for a dedicated database rules file; if other apps use the same database, retain their existing rule entries and add the `littleDaysBabyTracker` block alongside them.

**Privacy warning:** public read/write rules mean anyone who knows the Firebase project/database endpoint may read, overwrite, or delete this path. Baby profile details, growth records, milestone history, journal text, photos, and audio are included. This is intentionally configured for no sign-in, but it is not private or access-controlled. Public access can also be abused and consume Firebase quota.

## Automatic conflict handling

- There are no manual cloud save/load buttons in the app. Local changes are saved to browser storage immediately and automatically queued for Firebase when connected.
- At startup, the app compares local and cloud update timestamps. The newer record wins; if timestamps match, the writer ID provides a deterministic tie-break. This is last-write-wins, not a field-by-field merge.
- While the app is open on multiple devices, realtime database events automatically update a device when another device has a newer record. Avoid editing the same record on multiple devices at exactly the same time; a later full-record update can replace earlier edits.
- Each cloud save is limited to 8 MB. Images are resized/compressed before storage, but large voice clips can still exceed the limit. In that case the app keeps the local copy and shows a small status indicator. Export a JSON backup periodically.
- Data is stored in one shared record for all installations. Anyone using the URL/project and permissive rules sees the same baby record.

## Deploy to GitHub Pages

1. Extract this ZIP. Keep `index.html`, `manifest.webmanifest`, `sw.js`, and the `icons/` folder together.
2. Upload/commit all files to the root of a GitHub repository.
3. In GitHub, open **Settings → Pages**, select the branch and `/ (root)` folder, and save.
4. Open the published HTTPS URL. To install on iPhone/iPad, open the site in Safari, tap **Share → Add to Home Screen**. On Android, open in Chrome and choose **Install app** or **Add to Home screen**.

For local testing, serve this directory over localhost rather than opening `index.html` through `file://`:

```bash
python -m http.server 8000
```

Open `http://localhost:8000`. Service workers and installability require HTTPS or localhost. The Chart.js library and Firebase SDK are loaded from their official CDN URLs, so charting and cloud sync require internet access; basic state editing uses local storage when offline.

The top header is intentionally hidden for a cleaner mobile-first view. Demo data, backup, and import are grouped under the collapsed **More tools** section; the install button appears when supported.
