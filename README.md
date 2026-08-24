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

## On auto-importing from Kindle/Audible/Apple accounts

Worth documenting why this app doesn't do this: none of Amazon (Kindle or Audible) or Apple offer a public API for reading the contents of your personal library. There's no "Sign in with Amazon" or "Sign in with Apple" scope that hands over your book list — this isn't a gap in this app, it's not offered to any third-party developer.

What exists instead are unofficial, reverse-engineered clients (mainly for Audible) that mimic the private app's login flow. Using one would mean:
- Running a real backend server — this can't happen from a static page in a browser, both because of cross-origin restrictions and because the login flow involves device-registration cryptography the Amazon apps do internally.
- Handling 2FA/CAPTCHA challenges interactively.
- Accepting that it's against Amazon's terms of service in the fine print, and can break without warning whenever Amazon changes something internally.

That's a meaningfully bigger, riskier project than this app, and not one built into it.

**A lower-risk alternative, if useful later:** a small script you paste into your browser's console while logged into `amazon.com` or `audible.com` yourself, which reads the "Manage Your Content and Devices" or library page you're already looking at and turns it into JSON — then you paste that into this app's Import button. No credentials touch the app, nothing runs as a service, and it's a manual "run it when you want a refresh" step rather than live sync. It's also fragile (breaks if Amazon changes their page layout) and one-directional (pulls a list in, doesn't keep two-way sync). Ask if you'd like this built.

## Features

- **Add by ISBN** — paste an ISBN-10/13 and it pulls title, author, cover, page count, year, and genre from Google Books (falling back to Open Library). Edit anything before saving, or skip the lookup and enter a book by hand.
- **Bulk ISBN import** — paste a whole list of ISBNs (one per line) and it looks them all up and adds them, with a live log of what worked and what didn't.
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
