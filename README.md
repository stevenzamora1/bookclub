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
- **Multiple quotes per book.** Add as many as you want, per person, per book.
- **Any year.** Arrows beside the year in the header. A year that doesn't exist yet
  gets created with twelve blank months and the right Chinese zodiac animal.
- **Wrapped.** Year-end dashboard: pages read, averages, biggest score disagreement,
  what genres you actually read, and whether you're harsher than Goodreads.
- **Next pick roulette.** Dump candidates into a shared pile, spin, and assign the
  winner straight to a month. Settles stalemates.

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

Page counts and genres come from Google Books and Open Library, fetched on load for
any book missing them. That's what powers the Wrapped stats.

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
  "members": [ { "id": "steven", "name": "Steven", "emoji": "" },
               { "id": "mia", "name": "Mia", "emoji": "🌭" } ],
  "backlog": [ { "title": "...", "author": "...", "addedBy": "mia" } ],
  "meta":    { "updatedBy": "mia", "updatedAt": "2026-08-04T..." },
  "theme":   { "--bg": "#16100e", ... },
  "months": [
    {
      "id": "august",              // must be a lowercase month name
      "name": "August",
      "bom": {                     // book of the month, or null
        "title": "...", "author": "...", "goodreads": 3.94,
        "cover": "", "pages": 336, "genres": ["Romance", "Suspense"],
        "scores": { "steven": 4, "mia": 5 },
        "quotes": { "steven": ["...", "..."], "mia": ["..."] }
      },
      "top": [                     // each person's standout read
        { "member": "mia", "title": "...", "author": "...", "score": 5,
          "cover": "", "pages": 400, "genres": [], "quotes": ["...", "..."] }
      ]
    }
  ]
}
```

`month.id` has to be the lowercase English month name — that's how the site works out
which card is the current one.

Quotes are arrays. Old documents storing a single string are converted automatically
on load, so nothing needs migrating by hand.

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
3. Authentication → Get started → Sign-in method → enable **Anonymous**
4. Firestore → Rules tab → paste the block below → Publish

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /bookclub/{docId} {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```

5. Project settings → Your apps → `</>` → register, skip Hosting
6. Copy the six values into the `window.FIREBASE` block near the top of `index.html`

First load writes the seed data into an empty document, so a brand new project
comes up populated rather than blank.

### Security, honestly

The Firebase config in `index.html` is public by design — it ships in the JavaScript
of every Firebase web app and identifies the project rather than authorizing access.
GitHub's secret scanner flags it anyway. That's a false positive; rotating the key
accomplishes nothing and breaks the site.

What actually controls access is the rules. They require `request.auth != null`, and
the site signs in anonymously on load, so neither of us sees a login screen. This stops
the bots that scan public GitHub repos for Firebase configs with wide-open rules —
which is a real and automated thing, not a hypothetical.

It does not stop a determined person who reads the source. For two people tracking
paperbacks that's the right amount of effort. Download a backup now and then anyway;
the free tier keeps no snapshots.

Worth also doing: in the [Google Cloud console](https://console.cloud.google.com/apis/credentials),
restrict the browser API key to HTTP referrers matching your Pages domain. Takes a
minute and means the key is useless from anywhere else.

---

## Adding a year

Just click the `›` arrow next to the year. A document that doesn't exist yet is created
on the spot — twelve empty months, the correct zodiac animal, and your current colors
and members carried over. Previous years stay untouched and you can flip back anytime.

`window.START_YEAR` in `index.html` sets the earliest year the `‹` arrow will go back to.
