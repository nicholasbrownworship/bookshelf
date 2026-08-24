# The Shelf

A library catalog for your (and your wife's) books, ebooks, and audiobooks. One `index.html` file — no build step, no framework, no backend required.

## Display screen (new)

Loading the page with no URL parameters now shows the **display screen** by default — this is meant for a monitor or tablet you leave running in the library. It auto-rotates through random covers from your collection every few seconds, with a blurred-cover backdrop, and shows QR codes for any ebook/audiobook that has a store link saved (see below).

- Click/tap the screen to pause on the current book; click again to resume.
- The small gear icon in the bottom-right corner opens the full management interface (search, add, edit, sort, etc.) — click "Back to display" there to return to the kiosk view.
- Bookmark `index.html?manage=1` on your own devices to jump straight into management instead of the display.
- **Display settings** (in Manage) lets you set how many seconds each book stays up, and whether physical books are included in the rotation (they never get QR codes, just cover art).

### Store links & QR codes

Add a book's real Amazon (Kindle or Audible), Apple Books, or Libby product link when editing it, and the display screen will generate a QR code for each one. Scanning it opens that exact page — if you own the book on the account signed into that app, it opens straight to reading/listening; if not, it shows the buy page. That behavior comes from the store apps themselves, so getting the exact right link matters more than getting *a* link — a "Search Amazon ↗" / "Search Apple Books ↗" helper is included in the edit form to make finding it faster, but there's no reliable way to auto-generate the exact product URL from an ISBN, so these are pasted in by hand.

For Libby, there's no public search link — open the title in the Libby app, tap Share, and copy the link it gives you.

## Features

- **Add by ISBN** — paste an ISBN-10/13 and it pulls title, author, cover, page count, year, and genre from Google Books (falling back to Open Library). Edit anything before saving, or skip the lookup and enter a book by hand.
- **Bulk ISBN import** — paste a whole list of ISBNs (one per line) and it looks them all up and adds them, with a live log of what worked and what didn't.
- **Sort** by title, author, rating, series (grouped, in reading order), publish year, or date added.
- **Filter** by format (physical / ebook / audiobook), owner (mine / wife's / shared), tag, and a "lent out" toggle.
- **Search** across title, author, series, genre, and tags.
- **Two views** — a cover grid, and a "shelf" view that renders your books as spines on a shelf.
- **Tags / collections** — free-form tags like "to lend," "book club," "favorites" for grouping beyond genre.
- **Lending tracker** — mark a book as lent to someone (with a date), see it flagged on the shelf, mark it returned when it's back.
- **Reading goal** — set a target for the year; the header shows progress based on books marked "read" with a finish date in the current year.
- **Star ratings, reading status** (want to read / reading / read / DNF), series + book number, notes field.
- **Real-time sync (optional)** — connect a free Firebase project so your shelf updates live across your and your wife's devices. Without it, everything still works using the browser's local storage.
- **Export / Import** — grab a JSON backup any time, or merge/replace your library from one.

## Running it locally

Open `index.html` in a browser. That's it.

## Hosting on GitHub Pages

1. Push this folder to a repo.
2. **Settings → Pages** → Source: "Deploy from a branch" → pick your branch and the `/ (root)` folder (or wherever `index.html` lives).
3. GitHub gives you a URL like `https://yourusername.github.io/reponame/` within a minute or two.

## Setting up real-time sync (optional)

Without this, your library lives only in the current browser — export/import is how you'd move it between devices. If you want it to update live for both of you, here's the free way to do it with Firebase:

1. Go to **console.firebase.google.com** and create a free project.
2. **Build → Authentication** → "Get started" → enable the **Email/Password** sign-in method.
3. **Build → Firestore Database** → "Create database" → start in **test mode**.
4. In Firestore's **Rules** tab, replace the rules with:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /households/{householdId}/books/{bookId} {
         allow read, write: if request.auth != null;
       }
     }
   }
   ```
   This means: anyone signed in can read/write any household's books. For two people sharing one household ID, that's normal and fine — Firebase Auth is still required, so it's not open to the public internet. If you want it locked to just the two of you specifically, you can tighten this further to check `request.auth.token.email` against an allow-list; ask if you'd like help with that version.
5. **Project settings** (gear icon) → **General** → "Your apps" → add a **Web app** → copy the `firebaseConfig` object it gives you.
6. In the app, click **"Local only — set up sync"** in the header, paste that config in, pick a household ID (any shared word, like `brown-family`), and create an account.
7. On your wife's device, open the same app, click Sync, paste the *same* config and household ID, and use **"Sign in"** with a second account (or the same one, if you're both fine sharing a login).

Both devices will now see the same shelf update live. The free Firebase tier (Spark plan) comfortably covers a personal library — you won't hit its limits doing this.

## A few notes

- **Cover images** that fail to load fall back to a generated placeholder using the title and author, so the grid never shows broken image icons.
- **ISBN lookups need an internet connection** (public APIs, no key required). Everything else works offline once the page has loaded — sync excepted.
- Series numbers accept decimals (e.g. `2.5` for a novella between books 2 and 3).
- The reading goal is based on a book's **"Finished on"** date, which auto-fills to today when you mark a book "Read" — you can back-date it if you're logging older reads.
- **Never commit your Firebase config or paste API tokens into a public chat or repo file.** The Firebase web config is safe to expose client-side by design (it's not a secret), but any GitHub personal access token, password, or similar credential should stay out of chat messages and commit history — generate it, use it, and revoke it if it's ever been shared somewhere it shouldn't.
