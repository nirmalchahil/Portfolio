# Nirmal Chahil — Portfolio & CMS

A complete personal portfolio website with a full owner/admin CMS — built as a **single self-contained `index.html`** (no build step, no backend, no external CSS/JS files).

**Live:** https://nirmalchahil.github.io/Portfolio/

## Two sides, one file

### Public site (`#/`)
- Hero with cover photo, profile picture, availability badge and social links
- About, Skills, Services, Projects, Posts, Work Gallery, Experience, Contact and Social sections — all reorderable, renameable, layout-switchable and hideable from the admin panel
- Social-style **posts** (images, video, tags, project links) and **projects** (gallery, tech, features, demo/code/download links)
- Likes (per-browser, honestly labelled), moderated comments, contact form, public search, dark/light theme toggle
- Deep links: `#/post/<id>` and `#/project/<id>` open detail dialogs

### Owner panel (`#/admin`)
Passcode-protected admin dashboard:

| Area | Pages |
|---|---|
| Overview | Stats, recent activity, storage usage |
| Content | My Profile (5-step first-run wizard, pre-filled), Posts, Projects, Services, Skills, Media Library |
| Engagement | Comments moderation, Likes & Analytics |
| Customize | Pages/Sections (drag-reorder/toggle/rename), Custom HTML (owner-only, sandboxed), Appearance (live theme editor), Social Links, Contact & Inbox |
| System | Settings (Cloud Mode, passcode), Backup / Restore, Logout |

The media library supports **drag-and-drop / mobile-picker uploads** with automatic image compression (max 1600 px, WebP/JPEG) and video thumbnails.

## Local Mode vs Cloud Mode (read this)

This file runs **100% in your browser** — it is a static page on GitHub Pages.

- **Local Mode (default):** all data (profile, posts, projects, media metadata, comments, likes, settings) is stored in `localStorage` (prefix `npx.*`) and media files in `IndexedDB`. It works offline and needs no account — **but the data lives only in that browser on that device** and is not shared with other visitors or devices. The UI says so clearly instead of pretending otherwise.
- **Cloud Mode (optional):** the storage layer is an **adapter**. Paste your own Firebase web-app config in *Admin → Settings → Cloud Mode* and the app dynamically loads the official Firebase SDK (Firestore + Storage) and switches the adapter — same UI, no code changes. No credentials are bundled with this repository, and nothing is pre-configured.

Likewise, **email**: a static HTML page cannot send email. The contact form saves messages to the owner's inbox (this device) and offers a mail-client fallback; production email needs a backend (e.g. Formspree/EmailJS/Firebase).

**Owner passcode** is salted + SHA-256 hashed in the browser — a convenient lock for a personal device, **not** real server-side authentication (the UI labels it "Local Owner Mode").

## Backup / restore

*Admin → Backup / Restore* exports **everything** (including media as data URLs, videos ≤ 30 MB) to a single JSON file, and can restore/merge it in any other browser or device. Keep backups — clearing browser data clears Local Mode data.

## Architecture (inside the one file)

- `APP_CONFIG` at the top: `storageMode: "local"`, `backendMode: "none"` — the only place deployment mode is decided
- Adapter pattern: `LocalStorageAdapter` (active) / `FirebaseAdapter` (dormant, real dynamic-SDK-load integration point) behind a `Store` facade
- All persistence goes through the named `Data` service (`saveProfile`, `loadProfile`, `savePost`, `loadPosts`, `saveProject`, `loadProjects`, `uploadMedia`, `deleteMedia`, `saveSettings`, …) — `localStorage` is never touched directly from the UI
- 20 numbered JS sections: config, state, utils/icons, toasts/modals, storage adapters, media, data services, auth, profile, content services, comments, likes, custom HTML, appearance, backup, router, public UI, admin UI, events, init
- Hash routing (`#/`, `#/post/:id`, `#/project/:id`, `#/admin/:page`), global event delegation, SEO dynamic meta/OG/favicon
- Custom HTML sections render in a **sandboxed iframe** (opaque origin — no access to site data, cookies or storage) and are never visible to public visitors in any editor form

## Security notes

- No untrusted visitor input is ever injected as HTML — all user text is escaped
- The Custom HTML editor is owner-only; its output runs in a sandboxed iframe with no same-origin access
- No fake statistics, no dummy data after setup — empty states only

## Deploy

Pure static site → any host works. This repo serves it via GitHub Pages from the `gh-pages` branch (repo root, single file).
