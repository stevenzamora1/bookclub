# Communist China Book Club

A one-page site for tracking what we read in 2026 — book of the month, our top books,
scores out of 5, and the quotes worth keeping.

**Live site:** https://stevenzamora1.github.io/bookclub/

---

## What it does

- **Month grid.** Every month is a card with the cover, the pick, and both scores.
  Click one to open it.
- **Current month gets a ring.** Driven by your device clock, so it moves on its own.
  Past months you never scored get a muted "Not scored" stamp.
- **Both of us can edit from the link.** Tap your name in the top bar, hit
  "Add my scores," type. It saves by itself about a second after you stop.
- **Covers look themselves up.** Any book without one gets fetched on page load.
  "Browse covers" shows up to 12 editions if the automatic pick is wrong.
- **Colors are configurable.** Five presets or pick your own. Whatever you choose,
  the other person sees too.

---

## How it's built

One file. `index.html` contains the markup, styles, and script — no build step,
no npm, no framework. Push it and it's live.

Data lives in a single Firestore document (`bookclub/2026`) holding the whole year
as JSON. The Firebase SDK loads from Google's CDN at runtime.

```
index.html      the entire site
README.md       this
```

### Why Firestore and not something simpler

GitHub Pages is static — there's no server to receive an edit. Firestore was picked
over Supabase specifically because Supabase pauses free projects after 7 days of
inactivity, and a book club that meets monthly would find the site dead every single
time. Firestore's free tier has no inactivity pause.

---

## Data shape

```jsonc
{
  "club":    { "name": "...", "year": "2026", "zodiac": "Year of the Horse" },
  "members": [ { "id": "steven", "name": "Steven" }, ... ],
  "meta":    { "updatedBy": "mia", "updatedAt": "2026-08-04T..." },
  "theme":   { "--bg": "#16100e", ... },
  "months": [
    {
      "id": "august",              // must be a lowercase month name
      "name": "August",
      "bom": {                     // book of the month, or null
        "title": "...", "author": "...", "goodreads": 3.94, "cover": "",
        "scores": { "steven": 4, "mia": 5 },
        "quotes": { "steven": "...", "mia": "..." }
      },
      "top": [                     // each person's standout read
        { "member": "mia", "title": "...", "author": "...",
          "score": 5, "cover": "", "quote": "..." }
      ]
    }
  ]
}
```

`month.id` has to be the lowercase English month name — that's how the site works out
which card is the current one.

---

## Backups

**Download backup** saves the whole thing as JSON. **Restore backup** loads one back in
and pushes it to Firestore.

Worth doing occasionally. The security rules are wide open, so anyone who ends up with
the link can overwrite the data, and Firestore's free tier keeps no snapshots.

---

## Rebuilding the Firebase side

If the project is ever deleted or you start a fresh year:

1. [console.firebase.google.com](https://console.firebase.google.com) → Add project,
   skip Analytics
2. Databases & Storage → Firestore Database → Create database
   - Standard edition
   - **Database ID must stay `(default)`** — the SDK won't find it otherwise
   - Location `nam5`, and note this is permanent
   - Production mode
3. Rules tab → paste the block below → Publish

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /bookclub/{docId} {
         allow read, write: if true;
       }
     }
   }
   ```

4. Project settings → Your apps → `</>` → register, skip Hosting
5. Copy the six values into the `window.FIREBASE` block near the top of `index.html`

First load writes the seed data into an empty document, so a brand new project
comes up populated rather than blank.

### Security, honestly

Those rules let anyone read and write. The Firebase config is public by design — it
ships in the JavaScript of every Firebase web app and isn't a secret — but the rules
are what actually control access, and right now they control nothing. Fine for two
people and a shared link. Not fine for anything you'd be upset to lose or have read.

To lock it down later, the usual move is Firebase Anonymous Auth plus
`allow read, write: if request.auth != null`, which stops drive-by bots without
making either of us log in.

---

## Starting 2027

Change `window.DOC_PATH` to `["bookclub", "2027"]` and update `club.year`. The 2026
document stays where it is, untouched.
