# Sourcing Desk

Working case for a proposed Proxima post-engagement subscription offering, built toward the VP
promotion board in March 2027.

- `index.html` — internal working case: thesis, market map, tier design, live economics model,
  evidence plan, objection prep, open questions, timeline.
- `concept.html` — client-facing pilot pitch.

Live at https://brianchernauskas.github.io/sourcing-desk/

## One setup step before checklist sync works

The evidence checklist can sync ticks across devices through the existing `bourbonffldraft`
Firestore project. That project's current rules only cover `/entries` and `/config/settings`, so
**one rule has to be added** or every sync write is denied:

```
match /sourcing-desk/{doc} {
  allow read, write: if true;
}
```

Add it alongside the existing matches in Firebase console → Firestore → Rules → Publish. The full
ruleset becomes:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /entries/{entry}       { allow read, write: if true; }
    match /config/settings       { allow read, write: if true; }
    match /sourcing-desk/{doc}   { allow read, write: if true; }
  }
}
```

Until that rule exists the page still works — it falls back to saving on the device it is open on,
and the sync indicator reads "Could not connect".

## How the sync is protected

This page is public, so the checklist must not sit at a guessable address.

Click **Sync across devices**, enter a phrase, and the browser hashes `sourcing-desk:<phrase>` with
SHA-256. That hash *is* the Firestore document id. The phrase itself is never transmitted and never
stored — only the resulting id is kept, in `localStorage`, so the device reconnects on its own next
visit.

Enter the same phrase on a second device and it lands on the same document. Anyone who does not know
the phrase cannot locate the document among 2^256 possible ids. Anyone who does know it has full
access — this is a capability, not a login, and the `allow read, write: if true` rule above is what
makes that trade.

Notes:

- **No recovery.** A mistyped phrase is not an error, it is a different and empty checklist. Retype
  the right one and the real list comes back.
- **First connect unions, it does not overwrite.** Ticks already on the device are merged up into
  whatever is already stored, so pairing a second device never wipes the first one's progress.
- **Stop syncing here** unpairs the current device only. Ticks stay on it and the stored document is
  left untouched.
- Changes propagate live via `onSnapshot`, so a tick on a laptop appears on a phone that has the page
  open.

The economics model inputs on the same page are still `localStorage` only — per-device, not synced.

## Local preview

Configured in `.claude/launch.json` as `sourcing-desk` on port 3012. Serving over http (rather than
opening the file directly) matters, because `localStorage` and the Firebase module import both need a
real origin.
