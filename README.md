# Curate — Private notes & decline templates for Medium editors

**Curate** is a privacy-first Chrome extension for **Medium publication editors**. It adds private, local-only notes to every submission and a library of kind, customizable decline and feedback templates you can insert into Medium's own review flow in one click.

> All data stays on your device. No accounts. No servers. Zero telemetry.

[![Chrome Web Store](https://img.shields.io/badge/Chrome_Web_Store-Add_to_Chrome-1a8917)](https://chromewebstore.google.com/detail/curate/iacamcpominpmepkjlabdedmdbmifffg) [![Website](https://img.shields.io/badge/website-curate-191919)](https://wushu75.github.io/curate/) ![Manifest V3](https://img.shields.io/badge/Manifest-V3-191919) ![Permissions](https://img.shields.io/badge/permissions-storage%20%2B%20medium.com-1a8917) ![Telemetry](https://img.shields.io/badge/telemetry-none-1a8917) ![Dependencies](https://img.shields.io/badge/dependencies-0-191919)

---

## Why Curate exists

Running a Medium publication means reviewing a steady stream of submissions. Medium has said publicly that editors want richer, customizable decline reasons — but that adding open text fields to its platform is difficult because of the risk of harassment.

Curate solves the editor's side of that problem **without adding anything to Medium's servers**:

- Your notes on a submission are **private and never leave your browser**.
- Your decline templates live **locally**, so you can be consistent, specific and kind without retyping the same feedback forty times a week.
- When you're ready, Curate **inserts the text into the field Medium already provides** — or copies it to your clipboard if no field is open.

## Features

| | |
|---|---|
| **Private notes per submission** | A notes field tied to each submission ID. Autosaves as you type, syncs across your open tabs, and shows a small dot on the Curate button when a submission already has a note. |
| **Decline & feedback templates** | Five professional templates included (Off-topic, Needs another pass, AI-generated content concerns, Great piece but not right now, Formatting & sourcing). Create, edit, duplicate, reorder and delete your own. |
| **Smart placeholders** | `{{author}}`, `{{title}}`, `{{publication}}`, `{{editor}}`, `{{date}}` are filled in automatically from the page and your settings, with friendly fallbacks. |
| **One-click insert** | Fills Medium's note / decline field (React-safe), appends or replaces based on your preference, and falls back to the clipboard when no field is available. |
| **Native-feeling UI** | Floating button and slide-in panel styled after medium.com: system fonts, hairline borders, 8px radii, thin line icons. Rendered inside a closed Shadow DOM so it never clashes with Medium's styles. |
| **Keyboard shortcut** | `Alt+Shift+C` toggles the panel. `Esc` closes it. |
| **Export / import** | Back up or share templates as JSON with co-editors. Optionally include notes. Merge or replace on import. |
| **Minimal permissions** | Only `storage` plus host access to `medium.com` and `*.medium.com`. |

## Privacy

Curate was designed so that there is nothing to leak:

- **Storage:** everything is kept in `chrome.storage.local` in your browser profile.
- **Network:** Curate makes **no network requests**. The only `fetch()` loads Curate's own bundled CSS file from inside the extension.
- **Isolation:** the panel lives in a *closed* Shadow DOM. Medium's page scripts can't read it, and keystrokes typed into Curate are stopped from reaching Medium's keyboard handlers.
- **No analytics, no accounts, no remote code.** Template usage counts ("Used 3×") are stored locally only.
- **You're in control:** delete individual notes, clear all notes, or remove the extension to erase everything.

## Installation

### From source (developer mode)

1. Clone or download this repository:
   ```bash
   git clone https://github.com/wushu75/curate.git
   ```
2. Open `chrome://extensions` in Chrome (or any Chromium browser: Edge, Brave, Arc).
3. Turn on **Developer mode** (top-right).
4. Click **Load unpacked** and select the `curate/` folder (the one containing `manifest.json`).
5. The welcome page opens automatically. Pin Curate from the puzzle-piece menu for easy access.

### From the Chrome Web Store

**[Install Curate from the Chrome Web Store](https://chromewebstore.google.com/detail/curate/iacamcpominpmepkjlabdedmdbmifffg)** — free, installs in seconds.

## How to use

1. Go to your publication on medium.com and open a submission (draft, story, or the publication's submissions page).
2. Click the **Curate** button in the bottom-right corner, or press `Alt+Shift+C`.
3. Write a **private note** — it saves automatically against that submission's ID.
4. When you decline or leave feedback, open Medium's field first, then click **Insert** on a template. The status line in the panel tells you whether a Medium field was detected; if not, the text is copied for you to paste.
5. Manage templates, notes and settings from **Manage templates** (or the toolbar popup).

## How it works

```
curate/
├── manifest.json                 Manifest V3, minimal permissions
├── public/icons/                 App icon (SVG source + PNG sizes)
├── src/
│   ├── background/service-worker.js   Seeds defaults, shortcut relay, opens options
│   ├── content/content.js             FAB, panel, detection, insert logic
│   ├── content/content.css            Medium-matching styles (loaded into Shadow DOM)
│   ├── options/                       Templates editor, notes, settings, backup
│   ├── popup/                         Status, stats and quick links
│   └── lib/storage.js                 Shared chrome.storage.local wrapper
└── store/chrome-web-store-listing.md
```

**Submission detection** (in order): a story link inside an open Medium dialog → `/p/<id>` in the URL → a `-<id>` slug suffix → `postId`/`storyId` query parameters → canonical / `og:url` metadata. If no submission is found, the note is saved against the page path instead.

**Field detection** (in order): the last Medium text field you clicked into → note/reason/feedback fields inside an open dialog → any clearly labelled note/reason field on the page → clipboard fallback. As a guard rail, Curate never types into the story body itself.

**Resilience to React re-renders:** Curate never modifies Medium's DOM except to fill a field you chose. Its host element sits on `<html>` and a childList-only observer re-attaches it if removed. The heavier subtree observer (used to notice dialogs opening) only runs while the panel is open and is debounced. SPA navigation is detected with a lightweight URL check plus `popstate` and the Navigation API where available.

## Customizing templates

Templates are plain text with optional placeholders:

| Placeholder | Filled with | Fallback |
|---|---|---|
| `{{author}}` | Writer's first name, detected from the page | `there` |
| `{{title}}` | Story title | `your story` |
| `{{publication}}` | Publication name from Settings | `our publication` |
| `{{editor}}` | Your name from Settings | `The editors` |
| `{{date}}` | Today's date | — |

Unknown placeholders are left untouched, so nothing disappears silently.

### Export format

```json
{
  "app": "curate",
  "format": 1,
  "exportedAt": "2026-10-02T12:00:00.000Z",
  "templates": [
    { "id": "…", "name": "Needs another pass", "category": "Revise", "body": "Hi {{author}}, …" }
  ],
  "settings": { "editorName": "Alex", "publicationName": "The Quiet Desk" }
}
```

A bare JSON array of `{ name, body, category? }` objects can also be imported.

## Compatibility

- Chrome 110+ and Chromium-based browsers (Edge, Brave, Arc, Opera).
- Works on `medium.com` and `*.medium.com` subdomains. Publications on fully custom domains are not covered in v1 because that would require broader host permissions.
- Medium updates its interface frequently. Curate relies on attribute-based detection and always falls back to the clipboard, so it degrades gracefully rather than breaking.

## FAQ

**Does Medium see my notes?** No. Notes never leave your browser. Only template text you explicitly insert is placed into Medium's field — exactly as if you'd typed it.

**Do my notes sync between computers?** Not in v1. This is deliberate: `chrome.storage.sync` would send data through your Google account. Use Export / Import to move data between machines.

**Will Curate break when Medium changes its UI?** Insertion may fall back to the clipboard until selectors are updated, but notes and templates continue to work.

**Is Curate affiliated with Medium?** No. Curate is an independent tool for editors.

## Contributing

Issues and pull requests are welcome at [github.com/wushu75/curate](https://github.com/wushu75/curate). The codebase is intentionally small, dependency-free vanilla JavaScript — no build step. Load the folder unpacked, edit, and click the reload icon on `chrome://extensions`.

## Links

- 🌐 [Website](https://wushu75.github.io/curate/)
- 📦 [GitHub](https://github.com/wushu75/curate)
- 🧩 [Chrome Web Store](https://chromewebstore.google.com/detail/curate/iacamcpominpmepkjlabdedmdbmifffg)

## License

MIT

---

*Keywords: Medium editor tools, Medium publication management, Medium submission review, decline templates, editorial feedback templates, private notes Chrome extension, privacy-first browser extension, Medium writers and editors.*
