# The Shelf

A library catalog for your (and your wife's) books, ebooks, and audiobooks. One `index.html` file — no build step, no framework, no backend required.

## The look

Aged parchment, gilded borders, candlelight — a working grimoire rather than a spreadsheet. Cinzel for big titles, Cormorant Garamond for headers, EB Garamond for body text. The display screen (below) is the centerpiece: an illuminated two-page spread with your book's cover set into the page like a bookplate.

## Display screen

Loading the page with no URL parameters shows the **display screen** by default — meant for a monitor or tablet left running in the library. It auto-rotates through random covers from your collection every few seconds, and shows QR codes for any ebook/audiobook that has a store link saved (see below).

- Click/tap the screen to pause on the current book; click again to resume. (Rotation is timer-based for now since the screen isn't touch-enabled yet — the pause/tap interaction is already built in for whenever it is.)
- The small gear icon in the bottom-right corner opens the full management interface (search, add, edit, sort, etc.) — click "Back to display" there to return to the kiosk view.
- Bookmark `index.html?manage=1` on your own devices to jump straight into management instead of the display.
- **Display settings** (in Manage) lets you set how many seconds each book stays up, and whether physical books are included in the rotation (they never get QR codes, just cover art).

### Store links & QR codes

Add a book's real Amazon (Kindle or Audible), Apple Books, or Libby product link when editing it, and the display screen generates a QR code for each one. Scanning it opens that exact page — if you own the book on the account signed into that app, it opens straight to reading/listening; if not, it shows the buy page. That behavior comes from the store apps themselves, so getting the exact right link matters more than getting *a* link — a "Search Amazon ↗" / "Search Apple Books ↗" helper is included in the edit form to make finding it faster, but there's no reliable way to auto-generate the exact product URL from an ISBN, so these are pasted in by hand.

For Libby, there's no public search link — open the title in the Libby app, tap Share, and copy the link it gives you.

## Importing from Goodreads or StoryGraph

**Toolbar → Goodreads/StoryGraph.** Export your library from either service as a CSV (Goodreads: My Books → Import and Export → Export Library; StoryGraph: Settings → Manage Your Account → Export StoryGraph Library), then pick that file in the import dialog.

Everything comes straight from the file — no lookups needed:
- Title, author (plus any additional authors), rating, page count, year, ISBN
- Shelves become tags (the "read"/"to-read"/"currently-reading" shelf becomes the book's status instead, so it isn't duplicated as a tag)
- Your review and private notes are combined into the notes field
- Date added and date finished carry over as-is

Format (physical/ebook/audiobook) is guessed from the "Binding" column — reliable for "Audiobook" and "Kindle Edition," a reasonable default of "physical" otherwise, worth a glance afterward. There's an optional "fetch cover art by ISBN" checkbox that adds a Google Books/Open Library lookup per book (slower — one request per title), and a "skip books already in your library" checkbox that matches on ISBN or title+author so re-running an export later doesn't create duplicates.

## On importing from Kindle/Audible/Apple accounts

There's no "Sign in with Amazon" or "Sign in with Apple" that hands a third-party app your book list — neither offers that publicly, so a real login-based import isn't realistic here (see below for why). But there are legitimate, sanctioned paths that don't require anything risky:

**Amazon's own data export.** Amazon has an official "Request My Data" tool (in your account's privacy settings) that lets you request a copy of your account data, including Kindle purchases and Audible activity — Audible is part of your Amazon account, so one request covers both. It's free, doesn't touch any credentials in this app, but takes a few days to arrive by email as a downloadable file. Once you have a real sample file, the importer for it can be built the same way the Goodreads one was — bring it back here and it can happen quickly.

**Apple's own data export.** `privacy.apple.com` → "Request a copy of your data" → the "Apple Media Services" category includes App Store/iTunes/Apple Books purchase history. Same idea — official, free, roughly a week's turnaround. If either of you reads Apple Books on a Mac, there's also a faster local option: Apple Books keeps its library in a database file on disk, and the open-source tool `readstor` (`brew install readstor`, or `cargo install readstor`) reads it directly — no waiting on Apple at all, though it's a command-line tool you'd run yourself.

**Why not a real login integration?** Neither Amazon nor Apple offers a public API for reading personal library contents to third-party apps — this isn't a gap in this app, it's simply not offered to any developer. What exists instead are unofficial, reverse-engineered clients (mainly for Audible) that mimic the private app's login flow, which would mean running a real backend server (can't happen from a static page, both for cross-origin reasons and because the login flow involves device-registration cryptography), handling 2FA/CAPTCHA interactively, and accepting it's against the platform's terms of service in the fine print — a meaningfully bigger, riskier project than this app, and not one built into it.

**Lowest-effort fallback, if the above don't pan out:** a small script pasted into your browser's console while logged into `amazon.com` yourself, reading the page you're already looking at into JSON for the Import button. No credentials touch the app, nothing runs as a service — but it's fragile (breaks if Amazon changes their page layout) and a manual one-time pull rather than sync. Ask if you'd like this built instead.

## Features

- **Add by ISBN** — paste an ISBN-10/13 and it pulls title, author, cover, page count, year, and genre from Google Books (falling back to Open Library). Edit anything before saving, or skip the lookup and enter a book by hand.
- **Bulk ISBN import** — paste a whole list of ISBNs (one per line) and it looks them all up and adds them, with a live log of what worked and what didn't.
- **Goodreads/StoryGraph import** — bring in an entire library from a CSV export in one go, ratings/shelves/dates and all (see above).
- **Sort** by title, author, rating, series (grouped, in reading order), publish year, or date added.
- **Filter** by format (physical / ebook / audiobook), owner (mine / wife's / shared), tag, and a "lent out" toggle.
- **Search** across title, author, series, genre, and tags.
- **Two views** — a cover grid, and a "shelf" view that renders your books as spines on a shelf.
- **Tags / collections** — free-form tags like "to lend," "book club," "favorites" for grouping beyond genre.
- **Lending tracker** — mark a book as lent to someone (with a date), see it flagged, mark it returned when it's back.
- **Reading goal** — set a target for the year; the header shows progress based on books marked "read" with a finish date in the current year.
- **Star ratings, reading status** (want to read / reading / read / DNF), series + book number, notes field.
- **Export / Import** — grab a JSON backup any time, or merge/replace your library from one. This is also how you move your library between devices, since everything is stored in the browser's local storage — there's no account or cloud sync.

## Running it locally

Open `index.html` in a browser. That's it.

## Hosting on GitHub Pages

1. Push this folder to a repo.
2. **Settings → Pages** → Source: "Deploy from a branch" → pick your branch and the `/ (root)` folder (or wherever `index.html` lives).
3. GitHub gives you a URL like `https://yourusername.github.io/reponame/` within a minute or two.

## A few notes

- **Cover images** that fail to load fall back to a generated placeholder using the title and author, so the grid never shows broken image icons.
- **ISBN lookups need an internet connection** (public APIs, no key required). Everything else works fully offline once the page has loaded.
- Series numbers accept decimals (e.g. `2.5` for a novella between books 2 and 3).
- The reading goal is based on a book's **"Finished on"** date, which auto-fills to today when you mark a book "Read" — you can back-date it if you're logging older reads.
- **Never paste API tokens, passwords, or other credentials into a public chat or a repo file.** They should stay out of chat messages and commit history — generate them, use them, and revoke them if they're ever shared somewhere they shouldn't be.
