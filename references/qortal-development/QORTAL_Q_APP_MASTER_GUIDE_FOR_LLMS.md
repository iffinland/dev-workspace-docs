# The Qortal Q-App Builder's Guide (for LLMs and humans)

**What this is:** a self-contained, practical reference for building **Q-Apps** (service `APP`) and **QDN websites** (service `WEBSITE`) that work the first time in **Qortal Hub** (desktop) and **Qortal GO** (Android). Paste the whole file into any LLM, then describe what you want to build.

**What it is based on:** Qortal Core source and a live node, Qortal Hub source, and lessons from shipping and maintaining several real Q-Apps (file sharing, mail, shop, minting community) in Hub and GO with real data. Every rule here exists because something broke without it.

**Versions checked against**

| Component | Version | Notes |
|---|---|---|
| Qortal Core | **6.1.9** on a live node; source read at 6.1.4 with the 6.1.4 → 6.1.9 diff checked | `q-apps.js` actions and URL building are the same in 6.1.4 and 6.1.9 |
| Qortal Hub | **3.0.3** (released) and `develop` as of 2026-10-29 | GO 3.0 is built from the same code and has the same action list |
| Qortal GO | 3.0 (Capacitor build of Hub, Android System WebView, minSdk 26) | |

Facts marked **UNVERIFIED** were observed or inferred but not confirmed in source. Exact `qortalRequest` parameters can change between Hub releases: when in doubt, call `SHOW_ACTIONS` (Section 19) and check the official docs.

**Official references**
- Q-Apps docs: https://qortal.dev/docs/q-apps
- Core REST API (Swagger): https://api.qortal.org/api-documentation/ (your own node serves the same at `http://127.0.0.1:12391/api-documentation` once `"apiDocumentationEnabled": true` is set in its `settings.json`; it is off by default)
- Source: https://github.com/Qortal (Core: `qortal`, `q-apps.js` lives at `src/main/resources/q-apps/q-apps.js`; Hub: `Qortal-Hub`)

---

## Contents

0. How to use this guide
1. The Golden Rules
2. Mental model: Core, QDN, names, Hub, GO, how an app is served
3. Choosing the stack and project layout
4. Boot and environment: host detection, theme, language, frame size, Dev Mode
5. Identity and accounts
6. Reading data
7. Writing data
8. Transactions and money
9. Links and navigation
10. Files and media
11. Lists, notifications, chat
12. UI/UX that feels native in Hub and GO
13. Security and trust
14. Performance
15. Testing
16. Packaging and publishing
17. The big mistakes table
18. Templates (single-file app, React + Vite + TS starter, zip script)
19. Reference tables (actions, REST, services, search parameters, categories)
20. A compact system prompt for LLMs

---

## 0. How to use this guide

### 0.1 For the human

1. Paste this whole file into the LLM conversation.
2. Then write one paragraph about your app: what it does, who uses it, what it stores.
3. Answer the LLM's questions (0.3).
4. You get either a single `index.html` (small apps) or a Vite project. Build it, zip it with `index.html` at the zip root, test it in Hub Dev Mode (Section 15), then publish it in Hub as service `APP` under your registered name (Section 16).

### 0.2 For the LLM: reading order

1. Section 1 (Golden Rules) and Section 20 (system prompt) are binding.
2. Section 2 for the mental model.
3. Sections 4 to 11 for whatever your app touches.
4. Section 18 for the template you start from. **Start from a template; do not invent the boot, search, publish or link code from scratch.**
5. Section 17 to self-check before you answer.
6. Section 19 for exact action names and parameters.

### 0.3 For the LLM: ask these before writing code

Ask in one short message, with a proposed default for each so the user can just say "OK":

| Ask | Why it matters | Default if the user doesn't care |
|---|---|---|
| The **registered Qortal name** the app will be published under | It becomes the app's URL `/render/APP/<Name>`, link targets and the fallback name in Dev Mode | Ask; there is no default |
| Single HTML file or full React project? | Small tools fit one file; anything with several screens wants Vite + React + TS | Single file under ~600 lines of logic, else React |
| The features (screens) | Scope | The 3-5 features in the description |
| What data is **stored on QDN**, its shape, and who can write it | Identifiers, services, privacy, cost | Public JSON under an app prefix, one resource per item, written by its author's name |
| Is any data **private**? To whom? | Needs `*_PRIVATE` services and encryption to public keys or a group | Public |
| Does the app **move money** (QORT, assets, foreign coins)? | Extra safety flows (Section 8) | No |
| Must it read/write data of an **existing Qortal app**? | Then its services, identifiers and JSON shapes are fixed (Section 7.12) | No |
| Languages | i18n | English, read `?lang=` for later |
| Mobile priority | GO users are mostly on phones | Mobile-first, always |

Then state the plan in 5-10 lines (services, identifier scheme, JSON shapes, screens) before the code.

### 0.4 What "done" means for the LLM's answer

- It builds (`npm run build`) or opens as one file with no console errors outside Hub (it shows "Open in Qortal Hub or GO" instead of crashing).
- It follows every Golden Rule. Self-check against Section 17.
- It lists: the identifiers and JSON shapes it publishes, what each write costs, and the exact steps to test in Hub Dev Mode and publish.

---

## 1. The Golden Rules

These prevent about 90 % of failures. Each is explained later.

1. **Bundle everything. No outside network.** The page's CSP allows only the node itself (`'self'`): no CDNs, Google Fonts, remote images, analytics or web2 APIs. Inline `<script>`/`<style>` and `eval` are allowed. (4.8)
2. **Use relative asset paths.** Vite `base: ""` (or `"./"`). Never `base: "/"`: `/assets/...` hits Core's own `/assets` API and every load breaks. Don't add your own `<base href>`; Core injects one first. (2.4)
3. **Publish as `APP` under the default identifier, with `index.html` at the zip root.** No `manifest.json` is needed. (16)
4. **Detect the host with `typeof qortalRequest === 'function'`**, never `window.qortalRequest` (it is a global `const`, not a window property). Outside Hub install a stub that rejects with "Open in Qortal Hub or GO" so nothing crashes. (4.1)
5. **Theme: paint before the framework, follow Hub live.** Read `?theme=`, remember it, fall back to the remembered value (a reload without `?theme=` gets `_qdnTheme="light"`), and listen for `THEME_CHANGED`. (4.3)
6. **Size to the frame, not `100vh`.** Phone layout below 600 px wide whatever the device; test at 360×740, 390×844, 844×390, ~700 wide and 1280+. (4.5, 12)
7. **Router basename comes from `_qdnBase`, tolerant of spelling** (`+` vs `%2B`, and non-ASCII names, which the browser percent-encodes in `location.pathname`), and `""` in Dev Mode. `APP` → path routing works; `WEBSITE` → use hash routing. (9.3)
8. **Answer Hub's `NAVIGATE_TO_PATH` immediately with `NAVIGATION_SUCCESS`**, or Hub reloads your app. Never use `window.history` / `navigate(-1)` for an in-app Back button. (9.4)
9. **Every list search sends `mode=ALL` and an explicit `limit`.** The default mode `LATEST` returns one row per (name, service); a missing `limit` or `limit=0` returns *everything*. (6.3)
10. **Search matches are "contains", case-insensitive.** Use `prefix=true` for identifiers, `exactmatchnames=true` for names, then **compare exactly in code**. `prefix=true` turns *every* text term (`query`, `title`, `description`) into starts-with, so never combine it with free-text search (6.3, "Text search inside your app").
11. **Encode values yourself.** `q-apps.js` puts parameters into URLs raw: `+` becomes a space, `&#?%/` break the request. For any user-supplied value, call the REST endpoint with `fetch` + `URLSearchParams` / `encodeURIComponent`. (6.2)
12. **Trust the search row, not the body.** Publisher name and identifier come from the row; validate every body's types and lengths; normalise on read. (13)
13. **Data may not be on the node yet.** Use `?async=true` or check status first, show "Fetching from peers…", retry with backoff, then a clear "Not available right now". Never an endless spinner; never remember a failure as "doesn't exist". (6.6)
14. **Identifiers:** app-prefixed, `[A-Za-z0-9_-]`, ≤ 64 bytes, unique per item. New kinds of data get a new prefix. (7.1)
15. **Base64 must be UTF-8 safe.** Never `btoa(JSON.stringify(x))`. (7.3)
16. **Every publish costs 0.01 QORT and needs a registered name.** Check the name before the user fills a form, show the cost, and **never publish automatically** (not at boot, not on a failed read, not in a loop). (7.5)
17. **Max 25 unconfirmed transactions per account.** Publish more than ~20 resources in batches and wait for a block between them. (7.7)
18. **A timeout is "unknown", not "failed".** Hub stops waiting after 30 s but its dialog stays up to 60 s and a late Accept still runs. Look the write up on QDN/chain before offering Retry. **Never pay or publish twice.** (7.9, 8.3)
19. **A decline is a quiet cancel**, in any of Hub's 12 languages. Normalise all error shapes into text; never show `[object Object]`. (4.9)
20. **Money moves only on an explicit user tap**, after validation, with the amount and recipient shown; save the pending state before asking Hub. (8)
21. **`*_PRIVATE` services must hold encrypted data; public services must not.** Core checks this only for some services, so your app must always encrypt before publishing to a `_PRIVATE` service. Title/description/tags are always public, even on encrypted resources. (7.11)
22. **Never use a resource's `created` time for deadlines**; it is publisher-signed (up to 24 h old). Use block time. (7.8)
23. **Public nodes refuse some actions** (lists, hosted data, foreign coins…). Most GO users are on a public node: ask `IS_USING_PUBLIC_NODE` once at start-up and degrade quietly. Don't detect it from error text (it is translated). (2.6)
24. **All Q-Apps on a node share one origin**: prefix every `localStorage`/IndexedDB key with your app slug; never store secrets there. (13)
25. **Sanitise user HTML with DOMPurify (current 3.x) and post-process on the DOM**, never with regex over HTML. Prefer plain text. (13)
26. **http(s) links don't open** (q-apps.js blocks them; no popups). Copy the URL to the clipboard and say so. (9.6)
27. **`qortal://` links: app name raw** (`+` literal, spaces as `%20`), each path segment `encodeURIComponent`. (9.1)
28. **Build `/arbitrary/...` URLs yourself** for images and avatars, lazily (IntersectionObserver). Never `<img src="qortal://…">` in a framework app. (10.3)
29. **Poll only while visible, with backoff, and stop on unmount.** Merge identical in-flight requests; cache for the session. (14)
30. **Survive app updates:** a missing JS chunk returns `index.html` with status 200, so catch chunk-load errors and reload once. (14.6)
31. **Feature-detect** uncommon actions with `SHOW_ACTIONS` (short timeout). Unknown actions are never answered; they just time out. (19.1)

---

## 2. Mental model

### 2.1 The pieces

| Piece | What it is | What it means for you |
|---|---|---|
| **Qortal Core** (the node) | Java node: blockchain + QDN (data network) + REST API on port 12391 | Your app is served by a node and talks to it with same-origin `fetch` |
| **QDN** | Qortal Data Network: files referenced by on-chain ARBITRARY transactions, data fetched from peers | Storage is public, signed, costs a fee per publish, and is *eventually* available |
| **Name** | Registered name (1.25 QORT), owned by an address. Accounts may hold several | **All QDN data belongs to a name**, not an address. No name → can't publish |
| **Service** | Type of resource (`APP`, `WEBSITE`, `JSON`, `DOCUMENT`, `IMAGE`, `THUMBNAIL`, `VIDEO`, `MAIL_PRIVATE`…) with size and validation rules | Choose the right service; see 19.3 |
| **Identifier** | Free string ≤ 64 bytes; empty / `default` = the default resource | Your "primary key" within a name+service |
| **Resource** | (service, name, identifier). **Latest publish wins**; old versions aren't readable via the normal API | "Update" = republish the same coordinates. "Delete" = publish a tombstone |
| **Qortal Hub** | Desktop client (Electron). Renders apps in an iframe, answers signing/publishing requests with approval dialogs | Your app never sees keys; it asks Hub |
| **Qortal GO** | Hub built for Android (Capacitor, System WebView) | Same API; phone screen; usually a **public node** |
| **Public node** | A node run by someone else that GO/Hub can use | Reads work; some actions are refused (2.6) |
| **Gateway** | Core serving sites over plain HTTP to normal browsers (`_qdnContext="gateway"`) | Read-only unless the Qortal browser extension is present |

### 2.2 How an app is served

```
Hub/GO tab
 └─ <iframe src="http://<node>/render/APP/<Name>/<path>?theme=dark&lang=en&identifier=">
      Core finds the latest APP resource of <Name>, builds it, and serves index.html with:
        <meta charset="UTF-8">
        <base href="/render/APP/<Name>/">
        <script> var _qdnContext="render", _qdnTheme, _qdnLang, _qdnService, _qdnName,
                 _qdnIdentifier, _qdnPath, _qdnBase, _qdnBaseWithPath </script>
        <script src="/apps/q-apps.js?time=…">     ← defines qortalRequest()
        …your index.html…
      + header Content-Security-Policy (4.8)
```

- **First load of data the node doesn't have yet**: Core answers **503 with its own loading page**, which polls the build status and reloads when ready. You don't handle this.
- **`/render/APP/...` vs `/render/WEBSITE/...`**: for `APP`, any path that is not a file returns `index.html` (status 200), so client-side routes work. For `WEBSITE`, unknown paths return **404**.
- **A missing asset under APP also returns `index.html` with 200** (not 404). A stale lazy chunk then fails with "Unexpected token <" or "Failed to fetch dynamically imported module". (14.6)
- Core re-serialises your HTML (Jsoup): doctype lowercased, pretty-printed, a second `<meta charset>` appears. Harmless.
- Only HTML files (`.html`, `.htm`, directory index) get the injection. Index candidates: `index.html`, `index.htm`, `default.html`, `default.htm`, `home.html`, `home.htm`.
- Non-HTML files are served with `Content-Security-Policy: default-src 'self'` and no cache headers.

### 2.3 Who answers a `qortalRequest`

```
your code ── qortalRequest({action, ...}) ──► q-apps.js (inside your frame)
   ├─ read actions: q-apps.js does fetch() to the node itself → Core answers (no dialog, no Hub)
   └─ everything else: postMessage to Hub/GO → Hub shows a dialog if needed → answers
         └─ unknown action: no answer at all → your promise times out
```

- **Core-answered (in-frame) actions**: `GET_ACCOUNT_DATA`, `GET_ACCOUNT_NAMES`, `SEARCH_NAMES`, `GET_NAME_DATA`, `GET_QDN_RESOURCE_URL`, `LINK_TO_QDN_RESOURCE`, `LIST_QDN_RESOURCES`, `SEARCH_QDN_RESOURCES`, `FETCH_QDN_RESOURCE`, `GET_QDN_RESOURCE_STATUS`, `GET_QDN_RESOURCE_PROPERTIES`, `GET_QDN_RESOURCE_METADATA`, `SEARCH_CHAT_MESSAGES`, `LIST_GROUPS`, `GET_BALANCE`, `GET_AT`, `GET_AT_DATA`, `LIST_ATS`, `FETCH_BLOCK`, `FETCH_BLOCK_RANGE`, `SEARCH_TRANSACTIONS`, `GET_PRICE`. They behave identically in Hub, GO, Dev Mode and on public nodes.
- **Hub-answered**: identity, publishing, encryption, payments, groups, polls, chat sending, lists, files, notifications, tabs.
- Hub only listens to **your app's own frame**. An iframe nested inside your app cannot use `qortalRequest` through Hub.

### 2.4 Paths and `<base>`

- Core's `<base href="/render/APP/<Name>/">` comes **first**, so it always wins. Adding your own `<base>` does nothing useful.
- On deep links the base is still the app root, so `./assets/x.js` works from `/render/APP/<Name>/a/b/c`.
- Consequence: relative URLs like `img/x.png`, `?q=`, `#anchor` resolve against the **app root**, not the current route. Use router links.
- In Hub Dev Mode (proxy) the base is `/` and `_qdnBase` is `""`.
- If you publish an app under a **non-default identifier**, Core adds `?identifier=` only to relative `<script src>` and `<link href>` in the HTML; dynamic `import()` chunks, CSS `url()` and `fetch` resolve to the *default* resource. **Publish apps under the default identifier.**

### 2.5 APP vs WEBSITE

| | `APP` | `WEBSITE` |
|---|---|---|
| Unknown path | serves `index.html` (200) | 404 |
| Routing | `BrowserRouter` with basename works | hash routing (`#/page`) or real files per route |
| Index file required at publish | no (only 50 MB cap) | yes, in the root |
| `qortalRequest` available | yes | yes (WEBSITE also gets q-apps.js) |
| Shown in Hub as | an app | a website (Hub lists websites with `default=true`) |
| Use for | interactive apps | static sites, docs, landing pages |

### 2.6 Public nodes (the GO default)

Hub/GO treats a node as public when no local node is configured or the URL is Qortal's public node. On a public node Hub **refuses**: `GET_LIST_ITEMS`, `ADD_LIST_ITEMS`, `DELETE_LIST_ITEM`, `GET_HOSTED_DATA`, `DELETE_HOSTED_DATA`, `ADMIN_ACTION`, foreign-server and foreign-fee actions, `START_CROSSCHAIN_SERVER`, `SEND_COIN` for non-QORT coins, ARRR wallet reads, and publishes over **500 MB** (2 GB elsewhere). Ask `IS_USING_PUBLIC_NODE` (no dialog) and hide or explain those features; treat the refusal as "unavailable here", not as an error.

### 2.7 Platform targets

- Hub 3.0.3 ships Electron 32 (Chromium 128); Hub `develop` moves to Electron 44 (Chromium 144).
- GO runs on Android 8+ System WebView, which can be **old** (WebView 124 seen in an emulator).
- Each app ships its own framework, so Hub's versions don't constrain you, but **build for `es2020`** and avoid bleeding-edge CSS/JS (e.g. check `:has()`, View Transitions, Popover API support before relying on them).
- The iframe sandbox is `allow-scripts allow-same-origin allow-forms allow-downloads allow-modals`; `allow` is `fullscreen; clipboard-read; clipboard-write; screen-wake-lock`. So: **no popups** (`window.open`, `target=_blank` do nothing), **no camera/microphone/geolocation** (GO denies them too), clipboard works, fullscreen video works, `<input type=file>` works (including camera capture via `accept="image/*" capture` in GO).
- UNVERIFIED (unreleased): Hub `develop` runs apps in an Electron `<webview>` guest where `window.parent === window`. Don't detect "inside Hub" by checking for a parent frame; use `typeof qortalRequest`.

---

## 3. Choosing the stack

### 3.1 Single file vs project

| Choose | When | Notes |
|---|---|---|
| **Single `index.html`** (vanilla JS, inline CSS) | One screen, a form and a list, < ~600 lines of logic, no rich text | Zero build step; still follow every rule. Template A |
| **Vite + React + TypeScript** | Several screens, routing, editors, media, tests | The proven setup for real Q-Apps. Template B |
| Vite + Vue/Svelte/Solid/Preact | Your preference | Same rules: `base: ""`, relative assets, bundle everything, the same helpers |
| UI kit (MUI etc.) | Optional | Read the MUI pitfalls in 12.9. A kit is not required |

Never: Next.js/Nuxt server features (no server), service workers (shared origin, CSP, caching across app versions), anything that needs a backend, CDN `<script>` tags, remote fonts.

### 3.2 Recommended project layout (React + TS)

```
my-app/
  index.html                 # boot script at top of <head> (theme, background, frame height)
  vite.config.ts             # base: "", target es2020, manual chunks
  package.json
  tsconfig.json              # "include": ["src"]  (not ".")
  scripts/zip.sh             # dist/* → release.zip with index.html at root
  public/                    # favicon etc. (copied as-is)
  src/
    main.tsx                 # stub outside Hub, router basename, frame height, render
    App.tsx                  # routes (lazy screens)
    config.ts                # APP_SLUG, PUBLISHED_NAME, ID prefixes — app identity in ONE place
    qortal/
      env.ts                 # host detection, _qdn* globals, app name
      request.ts             # qortal() wrapper, errorMessage, isDecline, isTimeout
      search.ts              # QDN search: encoded, paged, merged, cached, exact lookup
      resource.ts            # fetch bodies, base64 UTF-8, status, "fetching from peers"
      publish.ts             # single + batched publish, verify-before-retry
      image.ts               # resize photos + thumbnails, publish them with their JSON
      links.ts               # qortal:// links
    hub/
      routerBase.ts          # basename from _qdnBase
      navigation.tsx         # NAVIGATE_TO_PATH → NAVIGATION_SUCCESS; in-app back stack (useSafeBack)
      hostMessages.ts        # THEME_CHANGED / LANGUAGE_CHANGED
      frame.ts               # --app-height
    lib/ poll.ts lazyWithRetry.ts sanitize.ts
    data/                    # your JSON shapes, validators (pure, tested)
    screens/ components/     # components/ErrorBoundary.tsx wraps routes and cards
    test/setup.ts            # qortalRequest + fetch mocks
```

Tooling trap: if your folder path contains `+`, a tsconfig `"include": ["."]` silently matches almost nothing. Use `"src"`.

---

## 4. Boot and environment

### 4.1 Detecting the host (and not crashing outside it)

`q-apps.js` declares `const qortalRequest = …` and `const qortalRequestWithTimeout = …` at the top level of a classic script. These are **global lexical bindings, not `window` properties**:

```js
typeof qortalRequest === 'function'   // true inside Hub/GO/Dev Mode   ✔
window.qortalRequest                  // undefined even inside Hub      ✘
globalThis.qortalRequest              // undefined even inside Hub      ✘
```

Put this **first** in your entry script (before anything calls it):

```js
// Outside Hub (plain browser, vite dev without Hub) give a stub that rejects instead of throwing
// ReferenceError, which in React would unmount the whole tree.
if (typeof qortalRequest !== 'function') {
  window.qortalRequest = () => Promise.reject(new Error('Open this app in Qortal Hub or Qortal GO.'));
  window.qortalRequestWithTimeout = (req) => window.qortalRequest(req);
  window.__QORTAL_STUB__ = true;
}
export const inHost = () => !window.__QORTAL_STUB__;
```

In TypeScript declare the globals so a bare reference compiles:

```ts
declare const qortalRequest: (req: Record<string, unknown>) => Promise<any>;
declare const qortalRequestWithTimeout: (req: Record<string, unknown>, ms: number) => Promise<any>;
```

Other environment checks:

```ts
const w = window as any;
export const isFramed  = () => { try { return window.parent !== window; } catch { return true; } };
export const isDevMode = () => inHost() && w._qdnContext === 'proxy';   // Hub Dev Mode → Server; _qdnBase === ''
export const isGateway = () => w._qdnContext === 'gateway';             // read-only web gateway
export const isGo      = () => /Android/i.test(navigator.userAgent);    // GO = Hub in Android WebView
```

- `WHICH_UI` returns `'HUB_ELECTRON'` or `'HUB_WEB'`; **GO returns `'HUB_WEB'`**, so it cannot identify GO. Decide layout by viewport and pointer (`(pointer: coarse)`), not by host.
- Use `isGo()` only for behaviour that truly differs (file saving, 10.1).

### 4.2 Injected globals and URL parameters

| Global (`var`, so on `window`) | Value |
|---|---|
| `_qdnContext` | `"render"` (Hub/GO), `"proxy"` (Hub Dev Mode → Server), `"gateway"` (web gateway), `"domainMap"` |
| `_qdnTheme` | from `?theme=`; **`"light"` when the URL has no `?theme=`** |
| `_qdnLang` | from `?lang=`; `"en"` when missing |
| `_qdnService` | `"APP"` or `"WEBSITE"` |
| `_qdnName` | publisher name; spaces as `%20`, everything else raw (`+` stays `+`). `""` in Dev Mode. **Always `decodeURIComponent` it** (in try/catch) |
| `_qdnIdentifier` | identifier or `""`/`"default"` |
| `_qdnPath` | requested path, e.g. `/item/abc` |
| `_qdnBase` | `/render/APP/<Name>` (spaces `%20`); `""` in Dev Mode |
| `_qdnBaseWithPath` | base plus directory of the requested file |

Hub loads the frame as `/render/APP/<Name>/<path>?theme=<dark|light>&lang=<code>&identifier=<id>`; plus `&time=` on refresh and `&isManualNavigation=false` when Hub reloads the frame for back/forward. GO widgets add `&goWidget=1`. **Strip these keys** (`theme`, `lang`, `identifier`, `time`, `isManualNavigation`, `goWidget`) before you build share links or read your own query parameters.

### 4.3 Theme: light/dark, live switching, persistence

Facts:
- Hub passes `?theme=dark|light` when it opens the tab.
- Any in-app navigation drops that query; a **reload then gets `_qdnTheme="light"`** whatever Hub shows.
- Hub's light/dark switch **does not reload** the app; it posts `{action:'THEME_CHANGED', theme}`.

Recipe (precedence): `?theme=` (and remember it) → remembered value → OS preference → dark.

Inline at the very top of `<head>` (before any CSS/JS bundle) so the first paint is right:

```html
<script>
(function () {
  var KEY = 'myapp-hub-mode';                      // app-prefixed: localStorage is shared by all Q-Apps
  var q = new URLSearchParams(location.search).get('theme');
  var mode = null;
  try {
    if (q === 'light' || q === 'dark') { mode = q; localStorage.setItem(KEY, q); }
    else mode = localStorage.getItem(KEY);
  } catch (e) {}
  if (mode !== 'light' && mode !== 'dark')
    mode = window.matchMedia && matchMedia('(prefers-color-scheme: light)').matches ? 'light' : 'dark';
  var bg = mode === 'light' ? '#ffffff' : '#121212';  // = your theme's background tokens
  var root = document.documentElement;
  root.setAttribute('data-theme', mode);
  root.style.colorScheme = mode;
  root.style.background = bg;
})();
</script>
```

Then follow Hub live (4.6) and re-theme without reloading. Define every colour as a CSS variable per theme (`[data-theme="light"] { --bg: …; --text: …; }`) or in your UI kit's theme; never hard-code colours in components. Check 4.5:1 text contrast in both themes.

### 4.4 Language

- Boot: `?lang=` → remembered (`myapp-hub-lang`) → `navigator.language` → `en`.
- Hub posts `{action:'LANGUAGE_CHANGED', language}`; switch live.
- Hub itself ships 12 languages: ar, de, en, es, et, fi, fr, it, ja, pt, ru, zh. Support RTL (`dir="rtl"`) if you translate to Arabic.
- Decide on purpose whether numbers and dates follow the UI language or the device locale, and format **consistently** (a phone showed "43,69" next to "12,800.00"). Use `Intl.NumberFormat`/`Intl.DateTimeFormat`, not moment.js.

### 4.5 Frame size and mobile

- The app lives in an iframe **shorter than the screen** (Hub's tab bar; GO's bars). `100vh` overshoots and hides your bottom bar.
- GO hides its own bars on scroll (frame height changes, e.g. 717 ↔ 832 px) and **shrinks the frame for the on-screen keyboard** (to ~450-505 px).
- A phone held sideways in Hub gives a frame only **~266 px tall**.

```ts
// frame.ts — call once before render
export function watchFrameHeight() {
  const apply = () => {
    let framed = true; try { framed = window.parent !== window; } catch {}
    document.documentElement.style.setProperty('--app-height',
      framed && window.innerHeight > 0 ? `${Math.round(window.innerHeight)}px` : '100dvh');
  };
  apply();
  window.addEventListener('resize', apply);
  return () => window.removeEventListener('resize', apply);
}
```

```css
:root { --app-height: 100dvh; }
html, body { margin: 0; background: var(--bg); color: var(--text); }
#root { min-height: var(--app-height); overflow-x: clip; }   /* clip on #root, NOT hidden, NOT on html/body */
```

Add `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">` so `env(safe-area-inset-*)` works. Layout rules are in Section 12.

### 4.6 Messages from Hub into your frame

| `data.action` | Payload | You must |
|---|---|---|
| `THEME_CHANGED` | `theme: 'dark'\|'light'` | re-theme live, remember it |
| `LANGUAGE_CHANGED` | `language: '<code>'` | switch language, remember it |
| `NAVIGATE_TO_PATH` | `path` (often without leading `/`) | navigate and reply `NAVIGATION_SUCCESS` **at once** (9.4) |
| `PUBLISH_STATUS` | `publishLocation{name,identifier,service}` (may be URI-encoded), `chunks`, `totalChunks`, `retry`, `filename`, `processed` | optional progress UI (7.7) |
| `PERFORMING_NON_MANUAL`, `NAVIGATE_FORWARD` | | ignore (q-apps.js handles them) |
| `SET_TAB_SUCCESS` | `payload{name}` | ignore (q-apps.js) |
| `FULL_SCREEN_CHANGED` | `payload{enabled:false}` | GO only: update your full-screen state |
| anything else | | **ignore unknown messages** (GO posts its own internal ones) |

Accept messages only from an ancestor window (Hub may nest your frame inside another frame, so `e.source === window.parent` is too strict):

```ts
export function fromHost(e: MessageEvent): boolean {
  let w: Window = window;
  while (w !== window.top) { w = w.parent; if (e.source === w) return true; }
  return false;
}

export function listenToHost(h: { theme?: (m: 'light'|'dark') => void; language?: (l: string) => void }) {
  const onMessage = (e: MessageEvent) => {
    if (!fromHost(e)) return;
    const d = e.data as any;
    if (d?.action === 'THEME_CHANGED' && (d.theme === 'light' || d.theme === 'dark')) {
      try { localStorage.setItem('myapp-hub-mode', d.theme); } catch {}
      (window as any)._qdnTheme = d.theme;          // q-apps.js copies _qdnTheme into links it builds
      document.documentElement.setAttribute('data-theme', d.theme);
      document.documentElement.style.colorScheme = d.theme;
      document.documentElement.style.background = '';   // drop the boot script's inline colour (4.3)
      h.theme?.(d.theme);
    }
    if (d?.action === 'LANGUAGE_CHANGED' && typeof d.language === 'string' && /^[\w-]{1,20}$/.test(d.language)) {
      try { localStorage.setItem('myapp-hub-lang', d.language); } catch {}
      (window as any)._qdnLang = d.language;
      h.language?.(d.language);
    }
  };
  window.addEventListener('message', onMessage);
  return () => window.removeEventListener('message', onMessage);
}
```

Reply to Hub with `window.parent.postMessage(msg, '*')` (that's what q-apps.js does).

### 4.7 Contexts: render, Dev Mode, previews, gateway

| Context | URL | `_qdnBase` / `_qdnName` | Quirks |
|---|---|---|---|
| Published app in Hub/GO | `/render/APP/<Name>/…` | `/render/APP/<Name>` / `<Name>` | the normal case |
| Hub Dev Mode → **Server** | Core dev proxy on port **12393** → your Vite dev server | `""` / `""`, `_qdnContext="proxy"` | Only `/` and `*.html` get q-apps.js injected: reload at `/`, navigate with your router. `/assets/*` (and any first segment equal to a Core API root) returns 404 under the proxy. GETs only; Vite HMR websocket can't pass. CSP has `connect-src 'self' ws:` (no `wss:`/`blob:` fetch) |
| Hub Dev Mode → **Zip / Directory** (preview) | `/render/hash/<hash>/…?secret=…` | base is the hash path; name is the hash | No SPA fallback (deep-link reload 404s); q-apps.js skips route reporting |
| Web gateway | `/<Name>/…` or `/<SERVICE>/<Name>/…` | `_qdnContext="gateway"` | Interactive actions show a "read-only" modal unless the browser extension is present |

Dev Mode consequences:
- `_qdnName` is `""`, so links you copy need a fallback to your **published name** (keep it in `config.ts`).
- Hub hides "Copy link" in Dev Mode.
- In Dev Mode, don't tick "always allow" in Hub's dialogs, and load only builds you trust.
- `LINK_TO_QDN_RESOURCE` to another app: Hub ignores the new-tab request in Dev Mode, and after 200 ms q-apps.js navigates **your dev frame** away to that app.

### 4.8 Content Security Policy (verified header)

```
default-src 'self' 'unsafe-inline' 'unsafe-eval'; font-src 'self' data:;
media-src 'self' data: blob: http://127.0.0.1:* http://localhost:*;
img-src 'self' data: blob:; connect-src 'self' wss: blob:;
```

- Scripts, styles, fonts, images, `fetch`/XHR: **only the node** (plus `data:`/`blob:` where listed). Remote images fail silently.
- Inline scripts/styles and `eval` are allowed.
- Workers: no `worker-src`, so it falls back to `'self' …`; a same-origin worker file is the safe choice. UNVERIFIED: `blob:` workers and WebAssembly inside a worker (the worker file itself gets `default-src 'self'`, without `unsafe-eval`). Prefer running libraries on the main thread (e.g. pdf.js with its worker on the main thread, 10.2).
- WebSockets to the node (`wss:`, and `ws:` same-origin) work: e.g. `/websockets/blocks`.

### 4.9 Errors, timeouts and declines: one wrapper for every call

What a rejected `qortalRequest` looks like depends on who answered:

| Source | Reject value |
|---|---|
| Core (in-frame actions) | `{error: <number>, message}`, e.g. `{error:102, message:"invalid address"}`; or `{error:"Resource does not exist"}` |
| Hub | `{error: <string or object>, message: <string>}` (localised) |
| GO only: multi-publish Cancel (during upload) | `{message:"Publish cancelled", error:{cancelled:true, unsuccessfulPublishes:[…]}}`. Desktop Hub 3.0.3 never returns this shape; a decline in its dialog arrives as "user declined request" |
| Hub partial multi-publish failure | `{message, error:{unsuccessfulPublishes:[{reason, identifier, service, name}]}}` |
| Hub timeout | `{error:"Request timed out after 30000 ms (action: X)", message: same}`. This text is fixed English (not translated), so `/timed out/i` is safe; decline and public-node texts **are** translated |
| `GET_USER_ACCOUNT` failure (decline included) | Hub replaces every failure with the translated "unable to fetch user account" (e.g. de "Benutzerkonto konnte nicht abgerufen werden"), which contains no decline word. Treat any non-timeout rejection of `GET_USER_ACCOUNT` as a quiet "not signed in" |
| q-apps.js timeout | the **string** `"The request timed out"` |
| `SAVE_FILE` | a **string** (decline: "user declined to save file") |
| Gateway | `{error:"Interactive features were requested, but these are not yet supported when viewing via a gateway…"}` |

Resolve-side traps:
- Core-answered actions `JSON.parse` the response when they can; **any parsed object with a top-level `error` key rejects**. So a stored JSON document with a top-level `"error"` field is unreadable via `FETCH_QDN_RESOURCE`. Never use `error` as a top-level key in your data.
- Non-JSON error bodies (`Bad parameter "service"`, an HTML 500 page) **resolve as strings**. Check that results are arrays/objects before using them.
- `SEND_CHAT_MESSAGE` can resolve `false` on failure.

Timeouts (the effective limit is the smaller of q-apps.js's and Hub's):

| Action | q-apps.js default | Hub limit | Effective |
|---|---|---|---|
| Core-answered reads (`SEARCH_QDN_RESOURCES`, …) | 30 s | – | 30 s |
| `FETCH_QDN_RESOURCE` | 60 s | – | 60 s |
| `GET_QDN_RESOURCE_STATUS` with `build:true` | 30 s | – | use `qortalRequestWithTimeout(req, 10*60_000)` or a direct fetch |
| `PUBLISH_QDN_RESOURCE` | 1 h | 1 h | 1 h |
| `PUBLISH_MULTIPLE_QDN_RESOURCES` | 1 h | n × 30 min | use `qortalRequestWithTimeout(req, n * 30*60_000)` |
| `SAVE_FILE` | 1 h | none | 1 h |
| `GET_USER_ACCOUNT`, `SIGN_TRANSACTION`, `DECRYPT_DATA` | 1 h | 30 s | **30 s** |
| `SEND_COIN`, `JOIN_GROUP`, polls, trades, `DEPLOY_AT` | 5 min | 30 s | **30 s** |
| `SEND_CHAT_MESSAGE` | 60 s | 30 s | 30 s |
| Every other Hub action | 30 s | 30 s | 30 s |

- **Hub's approval dialog auto-declines after 60 s**, and nothing cancels the action when Hub's 30 s timer fires. If the user accepts between second 30 and 60, **the action still runs** (coin sent, transaction signed, account authenticated) while your app already got a timeout. (7.9, 8.3)
- `qortalRequestWithTimeout` races the request against your own ms; it does **not** cancel the request.
- q-apps.js runs at most **30 requests at once** and queues the rest. It does **not** dedupe `qortalRequest` calls (it merges only identical in-flight GET URLs). Hub merges identical in-flight read-only requests (e.g. two parallel `GET_USER_ACCOUNT` give one dialog).
- In GO, at most 20 open/queued dialog requests per app; an identical value-moving request is declined at once (double-tap guard); "Decline all" mutes the app for 15 s. **Send dialog requests one at a time.**

The wrapper (full version in Template B, `src/qortal/request.ts`):

```ts
export function errorMessage(e: unknown, fallback = 'Something went wrong'): string {
  if (typeof e === 'string') return e || fallback;
  const x = e as { error?: unknown; message?: unknown } | null;
  if (typeof x?.message === 'string' && x.message) return x.message;
  if (typeof x?.error === 'string' && x.error) return x.error;
  if (e instanceof Error) return e.message || fallback;
  return fallback;
}
// Roots of Hub's "user declined" text in all 12 Hub languages, plus "cancel".
const DECLINE = /declin|cancel|rechaz|refus|ablehn|abgelehnt|отклон|拒否|拒绝|kieltäyt|keeldus|rifiut|recus|رفض/i;
export function isDecline(e: unknown): boolean {
  const x = e as { error?: { cancelled?: unknown } } | null;
  if (x && typeof x === 'object' && x.error && typeof x.error === 'object' && x.error.cancelled) return true;
  return DECLINE.test(errorMessage(e, ''));
}
export const isTimeout = (e: unknown) => /timed out/i.test(errorMessage(e, ''));
// Fallback only: Hub's public-node refusal in its 12 languages. Primary: ask IS_USING_PUBLIC_NODE at start-up.
const PUBLIC_NODE = /public node|öffentlichen Node|nodo público|nœud public|nodo pubblico|nó público|публичный узел|パブリックノード|公共节点|julkisen solmun|avaliku haarde|نود عام/i;
export const isPublicNodeRefusal = (e: unknown) => PUBLIC_NODE.test(errorMessage(e, ''));
export const HUB_DIALOG_GRACE_MS = 35_000;   // wait this long after a timeout before checking/retrying
```

Ask once whether the user is on a public node (no dialog, resolves a boolean; older hosts may not know the action, so time-box it):

```ts
export async function usingPublicNode(): Promise<boolean> {
  try { return (await qortalRequestWithTimeout({ action: 'IS_USING_PUBLIC_NODE' }, 5000)) === true; }
  catch { return false; }   // unknown → assume local; a later refusal is still caught by isPublicNodeRefusal
}
```

UI rules:
- **Decline** → quiet: close the busy state, no red toast. For `GET_USER_ACCOUNT`, every non-timeout rejection counts as a decline (see table).
- **Timeout** on a read → "Taking longer than usual" + Retry.
- **Timeout** on a write → "Waiting for confirmation…", then check (7.9).
- **Public node** (from `IS_USING_PUBLIC_NODE`, or a refusal) → hide/explain the feature for the session.
- Anything else → plain-words message + Retry; log the raw error to the console.

---

## 5. Identity and accounts

### 5.1 Sign-in flow

```ts
const acct = await qortal({ action: 'GET_USER_ACCOUNT' });          // {address, publicKey} — no name!
const primary = await qortal({ action: 'GET_PRIMARY_NAME', address: acct.address }); // string, '' if none
const names = await (await fetch(`/names/address/${acct.address}?limit=100`)).json(); // [{name, owner, …}]
```

- `GET_USER_ACCOUNT` shows Hub's "authenticate this app?" dialog, with an "always authenticate automatically" tick stored per app name. Accepting also grants, for this tab session, silent `GET_USER_WALLET`, `GET_WALLET_BALANCE`, `GET_USER_WALLET_INFO`, `GET_USER_WALLET_TRANSACTIONS`, `GET_LIST_ITEMS` and a few more. Read wallet addresses, balances or transactions only when a feature the user opened needs them, say why on screen, and never publish or store them.
- **It is fine to call it on load** (many apps do; trusted apps get no dialog), but always keep a visible **Sign in** button, and handle decline and timeout:
  - Hub turns every failure, a decline included, into the translated "unable to fetch user account"; so treat **any rejection that is not a timeout** as a quiet "not signed in" (no error toast);
  - on **timeout**, retry once after `HUB_DIALOG_GRACE_MS` (a late Accept means the retry answers instantly without a dialog);
  - a locked Hub can leave the call hanging until the timeout (reported, not traced in source): show "Waiting for Hub…" with a Cancel that just shows the Sign-in button.
- Read-only apps can skip sign-in entirely; ask only when the user does something that needs identity.
- Never ask for seeds, passwords or private keys. There is no API that gives them to you.

### 5.2 Names

| Fact | Detail |
|---|---|
| Publishing needs a name | Check **before** the user fills a form: "Register a name in Hub to post (1.25 QORT)" |
| Primary name | the earliest-acquired name the account still holds: `GET_PRIMARY_NAME {address}` (Hub, string or `''`) or `GET /names/primary/{address}` → `{name?, owner}` (no `name` key if none). **Not** `names[0]` |
| Many names | accounts with 80+ names exist. Offer a name switcher; add a search box above ~15 names; remember the chosen name per address (prefixed key) |
| `GET_ACCOUNT_NAMES {address}` | q-apps.js ignores `limit`/`offset`; call `/names/address/{address}?limit=&offset=` directly to page |
| Characters | 3-40 bytes UTF-8, NFKC-normalised, spaces allowed. Real names contain spaces, `+`, `/`, `#`, `?`, `%`, `'`, `\|`, `_`, Greek letters and emoji. **Test with names like `Νίκος ΑΩ`, `Shop-/8/`, `My App+`, `A \| B`** |
| Case | `/names/{name}` is case-insensitive (reduced name); resources match names case-insensitively |
| Ownership changes | Names can be sold. When resolving a name to an account (votes, permissions), read `/names/{name}` → `owner` at read time, so one account's several names count once |
| Look-alikes | impostors register a real name plus an invisible character (e.g. U+2800). Flag names that contain invisible/format characters after NFKC (e.g. draw a line-through in the error colour); never alter the name itself |
| Name search order | `/names/search?query=` returns plain character order (capitals first). Rank results yourself: exact (case-insensitive) first, then shorter, then A-Z; if a full page lacks the typed name, look up `/names/{name}` directly |

```ts
// Flag names containing invisible or formatting characters (a simple, conservative check).
// Written with \u escapes on purpose: literal invisible characters get lost when code is copied.
const INVISIBLE = /[\u00AD\u034F\u061C\u115F\u1160\u17B4\u17B5\u180B-\u180F\u200B-\u200F\u202A-\u202E\u2060-\u206F\u2800\u3164\uFE00-\uFE0F\uFEFF\uFFA0]/u;
// Real emoji names use U+FE0F (emoji style) and U+200D (joiner, e.g. family emoji): allow them next to an emoji.
const EMOJI_GLUE = /(?<=[\p{Extended_Pictographic}\p{Emoji_Modifier}\u20E3][\uFE0E\uFE0F]?)[\u200D\uFE0E\uFE0F]|[\uFE0E\uFE0F](?=\u20E3)/gu;
export const looksSuspicious = (name: string) =>
  INVISIBLE.test(name.normalize('NFKC').replace(EMOJI_GLUE, ''));
```

### 5.3 Accounts without names, and other people

- Show addresses shortened (`Qabc…xyz9`) with copy; let them read and browse.
- Public key of anyone: `GET /addresses/{address}` (JSON with `publicKey`, `level`, `blocksMinted`) or `GET /addresses/publickey/{address}` (**plain text**, base58). An account that never sent a transaction has no public key on chain, so you can't encrypt to it yet.
- Name → owner address: `GET /names/{encodeURIComponent(name)}` → `owner`.
- `OPEN_USER_LOOKUP {user}` opens Hub's own user panel (name or address).

### 5.4 Session state

- Keep `{address, publicKey, names, activeName}` in one store; clear everything derived from the account (caches, decrypted data) when it changes.
- Cache **decrypted** results per viewer address only; switching accounts must never show another account's data.

---

## 6. Reading data

### 6.1 REST `fetch` vs `qortalRequest`

Both hit the same node. Relative `fetch('/arbitrary/...')` is same-origin, allowed by the CSP, and in GO on a public node "same origin" is that public node.

| Use | When |
|---|---|
| `fetch('/…')` with `URLSearchParams` | **Default for reads**: you control encoding, caching, paging, parallelism; works for endpoints with no action (`searchsimple`, `/transactions/unconfirmed`, `/assets/...`) |
| `qortalRequest` read actions | Fine when every value is safe ASCII you control (your own prefixes, numbers), or when you want q-apps.js's behaviour exactly |
| `qortalRequest` (Hub actions) | Everything that needs the user: account, publishing, signing, encryption, files, tabs |

Neither knows who the viewer is; only `GET_USER_ACCOUNT` does.

### 6.2 Encoding: the #1 silent bug

q-apps.js builds URLs by **string concatenation without encoding**, for `SEARCH_NAMES`, `GET_NAME_DATA`, `GET_ACCOUNT_NAMES`, `SEARCH_QDN_RESOURCES`, `FETCH_QDN_RESOURCE`, `GET_QDN_RESOURCE_STATUS/PROPERTIES/METADATA/URL`:

| Value | What Core receives | Result |
|---|---|---|
| `name: "Notes+"` in a query string | `Notes ` (space) | `[]` |
| `query: "a&b"` | two parameters | wrong results |
| `name: "Shop-/8/"` in a path | extra path segments | 404 |
| `#`, `?`, `%` anywhere | broken URL | error / wrong data |

Rules:
- For anything user-supplied (names, search text, identifiers you didn't generate), call REST yourself: `URLSearchParams` for query strings, `encodeURIComponent` per path segment.
- In **path segments** a literal `+` is fine (`/names/Notes+` works) but `encodeURIComponent` is always safe.
- Never decode names with `URLSearchParams` (it turns `+` into a space). Use `decodeURIComponent` in try/catch.

### 6.3 QDN search semantics (`GET /arbitrary/resources/search`)

| Parameter | Behaviour (verified) |
|---|---|
| `service` | **Always send it.** With it, search runs on Core's in-memory cache (fast); without it, a slower SQL path. Case-sensitive enum (`service=app` → 400 `Bad parameter`) |
| `mode` | **Default `LATEST` = one row per (name, service)**, the newest. `ALL` = one row per (service, name, identifier). **Lists of items need `mode=ALL`.** Case-sensitive (`mode=all` → 400) |
| `identifier` | **Contains**, case-insensitive. With `prefix=true`: starts-with, still case-insensitive |
| `name` (repeatable) | **Contains**, case-insensitive (`name=Bob` matches `Bob`, `Bobby`, `Bob Smith`). With `exactmatchnames=true`: exact, case-insensitive. Repeat `name=` to search many names in one request |
| `query` | **one** substring (not split into words), contained in the publisher name, identifier, title or description; with `default=true` it matches the name only |
| `title`, `description` | contains (needs metadata on the node) |
| `keywords` (repeatable) | any keyword appearing in the description |
| `prefix` | applies starts-with to **all** of identifier/name/title/description/query at once. So `identifier=myapp_post_&prefix=true&query=curry` finds only items whose name, identifier, title or description *start with* "curry". See "Text search inside your app" below |
| `default` | `true` = only default resources. `default=true` + `identifier` → error 125 |
| `limit` | **missing or `0` = no limit (everything)**. Always send it: ~20 for UI pages, ≤100 for background work |
| `offset` | applied after filtering |
| `reverse` | default order is **oldest first** by `created`; `reverse=true` = newest first |
| `before`, `after` | ms timestamps compared with **`created`** (first publish), strict `<`/`>` |
| `minlevel` | publisher account level ≥ n |
| `includemetadata` | adds `metadata{title, description, tags, category, categoryName}`; only once this node has fetched it (background, every ~5 min). Send only when you display it |
| `includestatus` | adds a **cached** status that can be stale; use the status endpoint for truth |
| `excludeblocked`, `followedonly` | use this node's block/follow lists; apply only when `true` (6.1.6+) |
| `namefilter` | a node-local list name; empty/missing list → `[]` |

Result: a **bare JSON array** of rows:

```json
{"name":"Alice","service":"JSON","identifier":"myapp_post_k3x9a","latestSignature":"3NSS6Q…","size":412,
 "created":1761730000000,"updated":1761731000000,
 "metadata":{"title":"…","description":"…","tags":["…"],"category":"…"}}
```

- **Default resources have no `identifier` key**: read a missing identifier as `"default"`.
- `created` = first publish of that identifier; `updated` = latest (or absent). **An edited item does not move up the list.** If ordering by "last activity" matters, publish a new identifier per event.
- Errors are not arrays: `Array.isArray(x) ? x : []` plus try/catch.
- Unknown parameters are ignored, not errors.

**Exact lookups.** A `limit=1` "does it exist?" check can return `my-shop-2` when you asked for `my-shop`, or another app's `xmyapp_post_…`. Always:

```ts
const rows = await searchQdn({ service, identifier: id, prefix: true, name, exactmatchnames: true, limit: 50 });
const hit = rows.find(r => (r.identifier ?? 'default') === id && r.name.toLowerCase() === name.toLowerCase());
```

and re-check prefixes too (`q-store-order-my-shop-` also matches `q-store-order-my-shop-2-…`).

**Text search inside your app.** Because `prefix=true` makes every term starts-with, search your own items like this:

```ts
// "chicken curry" typed by the user, inside identifiers starting with myapp_recipe_
const words = text.trim().toLowerCase().split(/\s+/).filter(Boolean);
const rows = await searchQdn({
  service: 'JSON',
  identifier: 'myapp_recipe_',      // NO prefix:true here: contains-match, re-checked below
  query: words[0],                   // or title: words[0]; one substring only
  mode: 'ALL', includemetadata: true, reverse: true, limit: 50,
});
const hits = rows.filter(r => {
  if (!(r.identifier ?? '').startsWith('myapp_recipe_')) return false;   // exact prefix in code
  const hay = `${r.metadata?.title ?? ''} ${r.metadata?.description ?? ''}`.toLowerCase();
  return words.every(w => hay.includes(w));
});
```

- `query` also matches the publisher name and the identifier: re-check the words in code, as above.
- Title and description match only once this node has the item's metadata, so a brand-new item may show up by title a few minutes late.
- Put the searchable words in `title`/`description` when you publish (7.4).

### 6.4 Other listing endpoints

| Endpoint | Use | Notes |
|---|---|---|
| `GET /arbitrary/resources/searchsimple?service=&identifier=&name=&prefix=&caseInsensitive=&before=&after=&limit=&offset=&reverse=` | cheap exact listing, one row per resource (like `mode=ALL`), not one per version | identifier and name **exact and case-sensitive** unless `prefix`/`caseInsensitive`; no LATEST grouping; no metadata/status. No qortalRequest action: use `fetch`. Has found records `search` missed |
| `GET /arbitrary/resources?service=&name=&identifier=&default=&limit=&offset=&reverse=` (`LIST_QDN_RESOURCES`) | list by exact name/identifier | ordered **by name**, not date |
| `GET /arbitrary/metadata/{service}/{name}/{identifier}` | title, description, tags, `files` list | |
| `GET /arbitrary/resource/properties/{service}/{name}/{identifier}` | size, mime, filename | identifier is required (use `default`) |

### 6.5 Fetching a resource body

`GET /arbitrary/{service}/{name}/{identifier}` (or `FETCH_QDN_RESOURCE {service, name, identifier, encoding?, filepath?}`):

- Data local → the file bytes (`Range` supported for media).
- **Data not local → Core blocks while it fetches from peers (5 attempts by default; `?attempts=n` changes that), then fails with HTTP 400 `{"error":125,"message":"Data unavailable. Please try again later."}`**, not 404. Treat both 404 and "Data unavailable" as "not here yet" (Template B `fetchResource` does). How long a cold read blocks is UNVERIFIED (seconds to tens of seconds; one live cold fetch took 5.7 s and succeeded). The `/render` path for apps waits ~3 s between its attempts.
- `?async=true` → starts a background download and answers at once (404 `{"error":1401}` until ready).
- `?encoding=base64` → base64 text (use for encrypted data).
- Multi-file resources need `?filepath=` ("filepath is required for resources containing more than one file").
- Unknown resource → 404 `{"error":1401,"message":"File not found"}`.
- `FETCH_QDN_RESOURCE` auto-parses JSON (a top-level `error` key rejects; a body `123` arrives as a number). An empty body rejects with `{error:"Empty response"}`.

Resource status (`GET /arbitrary/resource/status/{service}/{name}/{identifier}[?build=true]`, or `GET_QDN_RESOURCE_STATUS`):

| Status | Meaning | UI |
|---|---|---|
| `PUBLISHED` | known, not downloaded | "Fetching from peers…" |
| `DOWNLOADING` | chunks arriving (`localChunkCount/totalChunkCount`, `percentLoaded`) | progress |
| `DOWNLOADED` | all chunks, not built. **Stays here until asked with `build=true`** | call status with `build=true` |
| `BUILDING` | | "Preparing…" |
| `READY` | fetch it now | |
| `NOT_PUBLISHED` | doesn't exist (yet: a just-published item may reach this node later) | retry for ~60 s if you expect it, then "Not found" |
| `MISSING_DATA` | some chunks unavailable from peers | "Not available right now" + Retry |
| `BUILD_FAILED`, `UNSUPPORTED`, `BLOCKED` | failed / blocked on this node | stop |

- `build=true` makes Core build synchronously; through `qortalRequest` use `qortalRequestWithTimeout(req, 10*60_000)` (the 30 s default drops the answer). Without it, freshly published files can sit at "Preparing… 100 %" forever.
- Tiny resources (≤ 256 bytes) are stored on chain (`RAW_DATA`) and report odd chunk counts (e.g. 239/2, no `percentLoaded`). Don't compute progress blindly.
- `GET_QDN_RESOURCE_URL` costs a status round trip and returns `/arbitrary/<service>/<raw name>/<id>` with the name **unencoded**. Build URLs yourself instead.

### 6.6 "Fetching from peers": the pattern

For list bodies and images:

1. Try `GET …?async=true` (or check status first for big files).
2. On 404/"unavailable": show "Fetching from peers…" and retry at **2, 4, 8, 16 s** (paused while the tab is hidden).
3. After that, show the search row's title/date with "Not available on your node right now" and a Retry button. **Never an endless skeleton.**
4. A body of `"D"` (or `{deleted:true}` for JSON) means deleted (7.13): say so and drop it from lists.
5. **Never cache a failure as "doesn't exist"** for the session.

For large files, poll status every ~5 s (paused while hidden), call `build=true` when `DOWNLOADED` (at most every ~7.5 s), show `percentLoaded`, give up after ~6 consecutive status errors with Retry. Full code: Template B `resource.ts`.

### 6.7 Avatars

- URL: `/arbitrary/THUMBNAIL/${encodeURIComponent(name)}/qortal_avatar?async=true`.
- With `async=true`, Core answers **404 while it fetches or builds** an avatar it doesn't have yet (seen: 404, 404, then the image ~4 s later). Most names have no avatar at all.
- Pattern: render the letter fallback; load the `<img>` only when it enters the viewport; on error, check `/arbitrary/resource/status/THUMBNAIL/<name>/qortal_avatar` once: `NOT_PUBLISHED` → keep the letter for the session; otherwise retry with backoff (2, 4, 8, 15, 30 s), paused while hidden, **one shared timer per name**.
- Do **not** use a UI kit `Avatar src=` that loads on mount, or `GET_QDN_RESOURCE_URL` per row.
- Hub's app library shows the **publisher name's avatar** as the app icon: set an avatar for your publishing name.

### 6.8 Paging, dedupe and caching

- Page every list (`limit` + `offset`, "Load more" or infinite scroll with a sentinel).
- One search helper for the whole app that:
  - builds the URL with `URLSearchParams` (defaults `mode=ALL`, `reverse=true`, explicit `limit`);
  - **merges identical in-flight requests** (same URL → same promise);
  - caches results for the session with a **1-3 min TTL**, and lets you invalidate after a publish;
  - never caches failures.
- Merge per-item probes: one search with several `name=` parameters instead of one search per name (one real app went from 413 to 126 first-load searches this way).
- Fetch bodies only for rows near the viewport, **~5 at a time**.
- "Everything" means bounded paging (e.g. 100 per page, max 5 pages) with a "partial" flag.

Full code: Template B `search.ts`.

### 6.9 Encrypted reads

1. Fetch with `encoding=base64` (`FETCH_QDN_RESOURCE {…, encoding:'base64'}` or `?encoding=base64`).
2. Check for a tombstone.
3. `DECRYPT_DATA {encryptedData}` (no dialog; returns **base64** plaintext). For the older single-recipient format (`qortalEncryptedData`) also pass `publicKey` of the **other party** (sender's key when you're the recipient). Group-format data (`qortalGroupEncryptedData`, which `ENCRYPT_DATA` produces) needs no key. If unsure, pass the publisher's `publicKey` anyway.
4. Decode base64 → UTF-8 → JSON.
5. On failure mark the item "Unable to decrypt", don't throw.
6. Decrypt only what's on screen; cache per viewer address.
7. Group-key data: `DECRYPT_QORTAL_GROUP_DATA {groupId, base64}`.

`DECRYPT_DATA` does **not** decrypt CHAT direct messages, and no action does.

### 6.10 Other reads you'll need

| Need | Call |
|---|---|
| Balance (QORT) | `GET /addresses/balance/{address}` (plain number) or `GET_BALANCE {address}`. **`GET_BALANCE` with `assetId` is broken** (q-apps.js builds a bad URL): use `GET /assets/balances?address=…&assetid=…` |
| Asset info | `GET /assets/info?assetId=` (field is `name`, not `assetName`). Asset quantities can exceed 2^53: keep them as strings/BigInt |
| Name record | `GET /names/{name}` → `owner`, `data`, … |
| Group admins | `GET /groups/members/{id}?onlyAdmins=true` (**the paged `isAdmin` flag is wrong after page one**) |
| Group members | `GET /groups/members/{id}?limit=100&offset=…` (ignore `isAdmin`) |
| An account's groups | `GET /groups/member/{address}` |
| Transactions | `GET /transactions/search?txType=…&address=…&confirmationStatus=BOTH&limit=20&reverse=true`. `confirmationStatus` is **required** (missing → HTML 500); needs `txType` or `address` or `limit ≤ 20`; `startBlock`/`blockLimit` only with `CONFIRMED` and can be very slow |
| Never call | `/transactions/address/{a}` without limit (returns everything), `/blocks/signers` (slow) |
| Unconfirmed by me | `GET /transactions/unconfirmed?creator=<base58 publicKey>&limit=100` |
| Block time | `GET /blocks/byheight/{h}` (`includeOnlineSignatures=false` for lighter responses) |
| Price | `GET /crosschain/price/{LITECOIN\|BITCOIN\|…}?inverse=` → integer × 1e-8 |
| Polls | `GET /polls/{name}`, `GET /polls/votes/{name}?onlyCounts=true` |
| Trades | `/crosschain/trades?reverse=true` (paging can repeat/skip rows: read once, scoped); trade offers can be years old |

---

## 7. Writing data

### 7.1 Designing identifiers

| Rule | Why |
|---|---|
| Prefix with your app slug and a type: `myapp_post_`, `myapp_comment_<postId>_` | Searches by prefix; other apps' data never matches |
| Charset `[A-Za-z0-9_-]` | `/` breaks `qortal://` and gateway paths; `+`, spaces, `#`, `?`, `%` need encoding everywhere |
| ≤ **64 bytes UTF-8** | Core's hard limit (bytes, not characters) |
| Unique per item: `prefix + base36(time) + random` | Same (name, service, identifier) = overwrite of your own item |
| Remember substring matching | `post_` also matches `mypost_…`; choose distinctive prefixes and compare exactly |
| New kind of data → new prefix | Old readers never misread it; stays compatible |
| Exact lookups are **case-sensitive**, searches are not | Generate lowercase ids |
| One resource per event when time order matters (votes, comments) | `created` never changes on republish |
| Avoid embedding user names in identifiers | names contain `_`, spaces and can be long; if you must, put them last and parse carefully |

```ts
export const newId = (prefix: string) =>
  `${prefix}${Date.now().toString(36)}${Math.random().toString(36).slice(2, 8)}`;  // e.g. myapp_post_mh8x0k2qf3k1z
```

Nobody can overwrite **your** (name, service, identifier), but anyone can publish the same identifier under **their** name: filter by publisher where it matters.

### 7.2 JSON shapes and versioning

```json
{ "v": 1, "type": "post", "text": "Hello", "tags": ["intro"], "replyTo": null }
```

- Put a version field `v` in every document; readers accept every version they know and ignore unknown fields.
- Changes are **additive**: new optional fields only. A breaking change gets a new prefix.
- Validate on read: types, lengths, enums, sane numbers (a `createdAt` above 8.64e15 makes `Intl` throw). Normalise into your own type; one malformed document must not crash a page.
- Don't store the author or date in the body as truth: they come from the search row.
- Don't put a top-level `error` key in any document (q-apps.js treats it as a failure).
- Keep images out of JSON: base64 data URLs (seen at 25 kB to 1.8 MB each) make lists slow. Publish images as separate `THUMBNAIL`/`IMAGE` resources (prepared as in 10.3) and reference their identifiers.
- Give sibling resources **one shared suffix**: `myapp_recipe_<s>` (JSON), `myapp_thumb_<s>` (THUMBNAIL), `myapp_img_<s>` (IMAGE). A grid can then build `/arbitrary/THUMBNAIL/<name>/myapp_thumb_<s>?async=true` straight from the search row, without fetching each body first. Keep the identifiers in the JSON too.
- Pin your publish payloads with a pure builder function plus a unit test before changing any publish code.
- Rich-text editors change their HTML between major versions (e.g. Quill 2 list markup); normalise to one shape so older readers still work, and strip `&nbsp;` noise.

### 7.3 Encoding: UTF-8 safe base64

```ts
export function bytesToBase64(bytes: Uint8Array): string {
  let bin = ''; const CH = 0x8000;
  for (let i = 0; i < bytes.length; i += CH) bin += String.fromCharCode(...bytes.subarray(i, i + CH));
  return btoa(bin);
}
export function base64ToBytes(b64: string): Uint8Array {
  const bin = atob(b64); const out = new Uint8Array(bin.length);
  for (let i = 0; i < bin.length; i++) out[i] = bin.charCodeAt(i);
  return out;
}
export const objectToBase64 = (o: unknown) => bytesToBase64(new TextEncoder().encode(JSON.stringify(o)));
export const base64ToObject = <T>(b64: string): T => JSON.parse(new TextDecoder().decode(base64ToBytes(b64)));
```

`btoa(JSON.stringify(x))` throws on any non-Latin-1 character (accents are fine, emoji/CJK are not).

For binary data use `file` (a `File`/`Blob`) instead of base64: no 33 % inflation.

### 7.4 Metadata

| Field | Limit | Behaviour |
|---|---|---|
| `title` | 80 bytes UTF-8 | silently truncated |
| `description` | 240 bytes UTF-8 | silently truncated |
| `tags` / `tag1`…`tag5` | max 5, each 1-20 characters | **longer tags are dropped**, not cut |
| `category` | one enum value (19.5) | |
| `filename` | | used for downloads and MIME type |

- Metadata is **always public**, even on `*_PRIVATE` resources. Never put secrets, recipients' private info, or decrypted text in it.
- Metadata reaches other nodes later than the transaction (background fetch every ~5 min), so title/description search won't find a brand-new item right away.
- `description` is the only field Hub notification rules can match (11.2).

### 7.5 Fees and prerequisites

| Item | Cost |
|---|---|
| Any QDN publish (one resource = one ARBITRARY transaction, any size, any service incl. APP) | **0.01 QORT** |
| Multi-publish of n resources | n × 0.01 QORT (Hub checks the balance first) |
| Name registration | 1.25 QORT |
| Payment, poll, vote, group transactions | 0.01 QORT typically (read `GET /transactions/unitfee?txType=X`, value in 1e-8 QORT) |
| Chat message (`SEND_CHAT_MESSAGE`) | proof-of-work instead of a normal fee; Hub requires a balance of ≥ 4 QORT |

Before any publish UI: check (1) signed in, (2) owns a name, (3) balance ≥ fee. Show "Costs 0.01 QORT" on the button or right next to it.

**Never publish automatically**: not at boot, not to "initialise" an index, not when a read fails (a real app republished an empty index whenever a read failed, wiping its users' data), not in loops. Every publish follows an explicit user action.

### 7.6 Single publish

```ts
const tx = await qortal({
  action: 'PUBLISH_QDN_RESOURCE',
  name: activeName,               // a name the user owns; defaults to the primary name
  service: 'JSON',                // or DOCUMENT, IMAGE, …
  identifier: newId('myapp_post_'),
  data64: objectToBase64(doc),    // or base64:, or file: File/Blob
  filename: 'post.json',
  title: title.slice(0, 80),
  description: summary.slice(0, 240),
  tags: ['myapp'],
});                                // resolves to the ARBITRARY transaction JSON (has .signature)
```

- Hub shows a dialog with service, identifier, name and fee; it **encodes name and identifier itself**, so don't pre-encode them.
- Missing `identifier` → `default`. Identifiers starting `symmetric-qchat-group-` are refused.
- Optional: `category`, `encrypt` + `publicKeys` (7.11), `isMultiFileZip: true` with a zip `file` (publish an APP/WEBSITE from inside an app), `appFee` + `appFeeRecipient` (Hub adds a QORT payment after the publish, with an extra confirmation).
- A raw `data64` of a zip fails WEBSITE validation (Core validates before unzipping): use `file` + `isMultiFileZip`.
- `SESSION_PERMISSIONS {permissions:['PUBLISH_QDN_RESOURCE']}` asks once and then skips the dialog for this tab session. Use only for many small, user-initiated publishes, and say so in the UI. With this grant Hub no longer confirms each publish, so your app must: publish only from a user tap, never retry automatically, cap the number of publishes per action, and show the running cost. Never request it at sign-in.

### 7.7 Several resources: batching under the unconfirmed limit

- `PUBLISH_MULTIPLE_QDN_RESOURCES {resources:[{service, name, identifier, data64|base64|file, filename, title, description, tags, …}], encrypt?, publicKeys?}`: **one dialog**, but each resource is its own transaction and fee, published one at a time (1 s apart).
- Core refuses the **26th unconfirmed transaction** per account (`TOO_MANY_UNCONFIRMED`), and Hub doesn't split for you.
- Batch rule: ≤ 20 resources per call; between batches poll `GET /transactions/unconfirmed?creator=<publicKey>` until there's room (a block is ~1 minute). Put the "index" document (the one that lists the others) **last**, so it never points at files that didn't land.
- Timeout: `qortalRequestWithTimeout(req, resources.length * 30 * 60_000)`.
- Partial failure rejects with `error.unsuccessfulPublishes[]`; everything not listed **did** publish. Retry only the failed identifiers, with the **same identifiers**.
- GO only: a Cancel during upload rejects with `error.cancelled: true` (already-published local copies are cleaned up). Desktop Hub 3.0.3 has no such shape: a decline in its dialog arrives as "user declined request" (caught by `isDecline`). Handle both.
- Progress: listen for `PUBLISH_STATUS` (4.6), keyed by `decodeURIComponent(publishLocation.identifier)`; mark a row done on `processed: true`. In GO, Hub's own publish window may hide your modal while it runs; close your progress modal promptly when the promise settles.

Full code: Template B `publish.ts`.

### 7.8 Timestamps

- A resource's `created`/`updated` (and every transaction timestamp) is **chosen and signed by the publisher**. Core accepts transactions up to **24 h old** and **30 min in the future**.
- Fine for display and rough ordering; **not** for deadlines (auctions, votes, "first to claim"). For those, find the transaction (`/transactions/search?txType=ARBITRARY&address=<owner>&confirmationStatus=CONFIRMED…`), take its `blockHeight`, and use `/blocks/byheight/{h}` → `timestamp`.
- Note `/transactions/search` reports `service` as a **number** (APP = 1000, JSON = 1110…; see 19.3), while resource search returns names.

### 7.9 Timeouts and Retry: never pay twice

```ts
export async function writeThenVerify(send: () => Promise<unknown>, landed: () => Promise<boolean>) {
  try { await send(); return 'done' as const; }
  catch (e) {
    if (isDecline(e)) return 'declined' as const;
    if (isTimeout(e)) {
      await new Promise(r => setTimeout(r, HUB_DIALOG_GRACE_MS));     // the dialog may still be up
      return (await landed().catch(() => false)) ? 'done' as const : 'unknown' as const;
    }
    return { failed: errorMessage(e) };
  }
}
// landed() for a publish: findExact(service, name, identifier) with a fresh (uncached) search.
// 'unknown' → show "We couldn't confirm it yet" + "Check again" (runs landed() again), not a blind Retry.
```

### 7.10 Propagation and optimistic UI

- A new resource appears in **your node's** search as soon as the transaction is imported (even unconfirmed). Other nodes see it when they receive the transaction; the **data** is pulled from peers when someone views it, so the publisher's node must stay online until others have fetched it. Plan for 1-2 blocks (~1-2 min) before others reliably see it.
- Show the user's new item immediately ("Publishing…" then "On its way"), keep it in local state, and merge/dedupe by (name, identifier) when it appears in search. Invalidate your search cache after a publish.
- Optimistic likes/votes/follows with rollback or Undo.

### 7.11 Encryption and private data

| Service type | Data must be | Core error otherwise (single-file services only) |
|---|---|---|
| `*_PRIVATE` (`MAIL_PRIVATE`, `MESSAGE_PRIVATE`, `ATTACHMENT_PRIVATE`, `QCHAT_ATTACHMENT_PRIVATE`, `VIDEO_PRIVATE`, `AUDIO_PRIVATE`, `VOICE_PRIVATE`, and also `DOCUMENT_PRIVATE`, `IMAGE_PRIVATE`, `FILE_PRIVATE`) | encrypted (starts with `qortalEncryptedData` / `qortalGroupEncryptedData`) | `DATA_NOT_ENCRYPTED` |
| public services | not encrypted | `DATA_ENCRYPTED` |

- Core runs this check only for **single-file** services (e.g. `MAIL`, `MESSAGE`, `ATTACHMENT`, `QCHAT_ATTACHMENT`, `VIDEO`, `AUDIO`, `VOICE`, `JSON`, `THUMBNAIL` and their `_PRIVATE` forms). On Core 6.1.9, `FILE`, `IMAGE`, `DOCUMENT` and their `_PRIVATE` forms are **not** single-file, so Core accepts plain data in `DOCUMENT_PRIVATE`/`IMAGE_PRIVATE`/`FILE_PRIVATE`. **Your app must always encrypt before publishing to any `_PRIVATE` service**; nothing else will stop a plain-text "private" document.
- Hub's single `PUBLISH_QDN_RESOURCE` doesn't check the service either; only `PUBLISH_MULTIPLE_QDN_RESOURCES` refuses `encrypt` on a non-`_PRIVATE` service and plain data on a `_PRIVATE` one.

Ways to encrypt:
- **Publish with `encrypt: true, publicKeys: [recipientKey, …]`** on a `*_PRIVATE` service: Hub encrypts for those keys **and the publisher**. `encrypt` without a non-empty `publicKeys` is refused; `encrypt` on a public service is refused by `PUBLISH_MULTIPLE_QDN_RESOURCES` only, so always pair `encrypt` with a `_PRIVATE` service yourself. To keep a private copy for yourself only (e.g. favourites), pass your own public key: `publicKeys: [myPublicKey]`.
- `ENCRYPT_DATA {data64|base64|file|blob, publicKeys}` → base64 ciphertext (sender can always decrypt); then publish that base64 yourself to a `*_PRIVATE` service.
- `ENCRYPT_QORTAL_GROUP_DATA {groupId, data64}` / `DECRYPT_QORTAL_GROUP_DATA`: uses the group's shared key (published by group admins; fails with "no group key found" if none).
- `ENCRYPT_DATA_WITH_SHARING_KEY` / `DECRYPT_DATA_WITH_SHARING_KEY {encryptedData, key}`: content that recipients can later share.

Rules:
- Recipients need a public key on chain (they must have sent at least one transaction).
- Metadata stays public.
- Encrypted attachments must be **saved as a decrypted blob**; a location-based save writes ciphertext.
- Never let data you read trigger decrypt-and-resend of other resources: bind actions to what the user can see.

### 7.12 Compatibility with an existing app's data

If you read or write another app's data (or rebuild an app):
- Use **exactly** the same services, identifier schemes and prefixes, JSON shapes, encryption, and (for apps built on the `qapp-core` library) the same `publicSalt`/app identity used for hashed identifiers, or the app stops finding its own data.
- New data is additive only (extra fields, or new identifiers with their own prefix).
- List the identifiers in your plan before changing data code; pin shapes with tests.
- Tolerate variants written by other builds (e.g. a field that is sometimes a string, sometimes an array) and recognise other apps' records under a shared prefix, skipping them quietly instead of showing them as broken.

### 7.13 Updating and "deleting"

- **Update** = publish again to the same (service, name, identifier). Latest transaction wins (unconfirmed included). `created` stays the first publish time.
- **There is no network delete.** `DELETE_HOSTED_DATA` only removes your own node's copy.
- "Delete" = publish a **tombstone**: `"D"` for non-JSON services, `{"v":1,"deleted":true}` for `JSON` (the JSON service must parse), and filter tombstones when reading. Some apps use `{deleted:true}`/`cleared:true` flags; accept all you know.
- Old versions may still exist on other nodes; deletion is a convention, not a guarantee. Tell users "removed from the app", not "erased".

### 7.14 Read-modify-write

- **Read fresh** (bypass cache) right before rebuilding a document that aggregates data (an index, settings, a product list). A copy loaded with the page drops items added from another device.
- Never publish state before the published state has loaded (you'd overwrite it with defaults).
- Settings sync to QDN: save/restore on explicit buttons only; read tolerantly (object, JSON string or base64 JSON; drop unknown fields).

---

## 8. Transactions and money

### 8.1 Principles

1. Money moves **only after an explicit tap** on a button that states amount, coin and recipient. Never on load, never in a loop, never "retry automatically".
2. Validate everything **before** Hub opens (8.4); Hub sometimes hides Core's reason when a transaction fails.
3. **Save the pending state locally before asking Hub** (order id, amount, recipient, start time). Finish follow-up publishes later with the same identifiers.
4. A timeout is "unknown": look on chain before offering Retry (8.3).
5. Never show or charge in a coin before its exchange rate is known; never mix up units (QORT vs foreign coin).
6. Amounts: decimal strings with 8 decimals (`total.toFixed(8)` only after integer-safe maths). Asset quantities can exceed 2^53: never floats for them.
7. Pin money code with unit tests (exact request shapes) before you change it.
8. Some value-moving dialogs require the user to tick "I have read this request" (desktop Hub 3.0.3: QORT `SEND_COIN` and `SESSION_PERMISSIONS`; GO 3.0 also asset transfers, `SIGN_TRANSACTION`, trade orders, name sales/purchases and a few more; see 19.1). Expect it to take time, inside Hub's 30 s limit.

### 8.2 Paying in QORT

```ts
// Seller is a name: resolve to the CURRENT owner at pay time
const nameData = await (await fetch(`/names/${encodeURIComponent(sellerName)}`)).json();  // {owner, …}
saveLocally({ orderId, state: 'paying', amount, to: nameData.owner, startedAt: Date.now() });
const tx = await qortal({
  action: 'SEND_COIN',
  coin: 'QORT',
  destinationAddress: nameData.owner,   // `recipient` also accepted; QORT also accepts a name
  amount: amount.toFixed(8),
});                                      // QORT: transaction JSON with .signature
saveLocally({ orderId, state: 'paid', signature: tx.signature });
```

- `SEND_COIN` for foreign coins (`BTC`, `LTC`, `DOGE`, `DGB`, `RVN`, `ARRR`) needs a **local node**; refused on a public node. There is no memo for QORT payments.
- Assets: `TRANSFER_ASSET {assetId, amount, recipient}` (integer `assetId` ≥ 0). Check `/assets/info` first: refuse unknown or untransferable assets and fractional amounts of indivisible assets.
- Multi-recipient payments only exist as `MULTI_ASSET_PAYMENT_WITH_PRIVATE_DATA` (pays and publishes encrypted data per recipient; subject to Hub's 30 s limit, so verify afterwards).

### 8.3 After a timeout: look before Retry

Hub answers after 30 s but its dialog can still be accepted until 60 s. So after a timeout:

```ts
// Look for a matching payment for ~1 minute before offering Retry.
async function findPayment(buyer: string, to: string, amount: string, since: number, usedSignatures: Set<string>) {
  const url = `/transactions/search?txType=PAYMENT&address=${buyer}&confirmationStatus=BOTH&limit=20&reverse=true`;
  const rows = await (await fetch(url)).json();
  const hits = (Array.isArray(rows) ? rows : []).filter((t: any) =>
    t.recipient === to && Number(t.amount) === Number(amount) &&
    t.timestamp >= since - 60_000 && t.timestamp <= since + 30 * 60_000 &&
    !usedSignatures.has(t.signature));
  return hits.length === 1 ? { found: hits[0] } : hits.length > 1 ? { several: hits } : null;
}
```

- `found` → mark paid. `several` → let the user pick. Nothing after ~1 min → "Payment not found. Pay again?" (explicit).
- For assets use `txType=TRANSFER_ASSET` and match `assetId`.
- A payment signature counts for **one** order only.

### 8.4 Validate before Hub opens

| Check | How |
|---|---|
| Signed in, correct account | `GET_USER_ACCOUNT` |
| Balance ≥ amount + fee | `/addresses/balance/{address}`; fee from `/transactions/unitfee?txType=PAYMENT` |
| < 25 unconfirmed transactions | `/transactions/unconfirmed?creator=<publicKey>` |
| Recipient valid | `/addresses/validate/{address}`; name → `owner` at pay time |
| Asset exists, transferable, divisibility | `/assets/info?assetId=` |
| Not on a public node (foreign coins) | `IS_USING_PUBLIC_NODE` |

### 8.5 Verifying someone else's payment (seller side)

- Look the signature up on chain (`/transactions/signature/{sig}`) and check type, sender, recipient, amount, asset id **from Core**, never from the payer's JSON (a worthless asset named "QORT" displayed as "100 QORT").
- Never let one payment count for two orders.
- Words matter: call reviews "Ordered here", not "Verified buyer" (anyone can publish an order id for a fee). Treat favourites, likes and votes as soft signals.

### 8.6 Polls

- `CREATE_POLL {pollName, pollDescription, pollOptions, pollOwnerAddress}`: all four are required.
- **Trap (Hub 3.0.3/GO):** Hub uses only `pollOptions[0]` and splits it on `", "`. So `pollOptions: ['Yes', 'No']` creates a poll with **one** option. Send `pollOptions: ['Yes, No, Maybe']`; option texts can't contain `", "`. Re-check on newer Hub by previewing the dialog.
- Core rules (sizes in **UTF-8 bytes**): poll name unique (`POLL_ALREADY_EXISTS`), 3-400 bytes and already in normalised form (`NAME_NOT_NORMALIZED`); description 1-4000 bytes; 1-100 options, each 1-400 bytes, no duplicates (`DUPLICATE_OPTION`); **no close date** (enforce windows in your app using block time).
- `VOTE_ON_POLL {pollName, optionIndex}` (0 allowed). One vote per account; voting for a different option replaces it; voting again for the **same** option fails with `ALREADY_VOTED_FOR_THAT_OPTION` (disable the current choice in your UI).
- Read: `/polls/{name}`, `/polls/votes/{name}?onlyCounts=true`.

### 8.7 Groups

| Action | Params |
|---|---|
| `JOIN_GROUP` | `groupId` |
| `LEAVE_GROUP` | `groupId` |
| `CREATE_GROUP` | `groupName`, `type`, `approvalThreshold`, `minBlock`, `maxBlock`, `description?` |
| `UPDATE_GROUP` | `groupId`, `newOwner`, `type`, `approvalThreshold`, `minBlock`, `maxBlock`, `description?` (send the current description to keep it: a missing one is sent as empty), `txGroupId?` |
| `INVITE_TO_GROUP` | `groupId`, **`inviteeAddress`**, `inviteTime` (seconds, **must be non-zero**) |
| `CANCEL_GROUP_INVITE`, `KICK_FROM_GROUP` (`reason?`), `BAN_FROM_GROUP` (`reason?`, `banTime?`), `CANCEL_GROUP_BAN`, `ADD_GROUP_ADMIN`, `REMOVE_GROUP_ADMIN` | `groupId`, `qortalAddress` |

- Pass **addresses, never names**. Hub refuses `groupId: 0` (falsy check).
- Each shows a dialog with the fee, every time.
- `qortal://use-group/action-join/groupid-N` links (used by Qortal chat) are **not** resource links: q-apps.js throws on them and the frame can navigate away. Intercept such clicks and call `JOIN_GROUP` yourself.

### 8.8 Names as transactions

`REGISTER_NAME {name, description?}`, `UPDATE_NAME {oldName, newName, description?}`, `SELL_NAME {nameForSale, salePrice}`, `CANCEL_SELL_NAME {nameForSale}`, `BUY_NAME {nameForSale}`: each a dialog with fee.

### 8.9 Anything else: `SIGN_TRANSACTION`

No action exists for e.g. asset issue/orders, reward shares, plain multi-payments, `MESSAGE`. For those an app can build the **unsigned bytes** itself and ask Hub to sign:

- `SIGN_TRANSACTION {unsignedBytes: <base58>, process: true}` → Hub decodes the bytes via Core, shows the transaction type and JSON with a warning (GO 3.0 also requires the confirm tick), signs and (with `process:true`) broadcasts; returns the transaction JSON. `process:false` returns signed bytes (base58).
- Core's "build raw transaction" POST endpoints refuse on mainnet nodes by default, so build bytes in the app: common header (big-endian) `int32 type | int64 timestamp(ms) | int32 txGroupId | 64-byte last reference | 32-byte creator public key`, then the type's own fields, then `int64 fee` (1e-8 QORT). Inputs: `/addresses/lastreference/{address}`, `/transactions/unitfee?txType=X`, the public key from `GET_USER_ACCOUNT`.
- Check your bytes with the read-only `POST /transactions/decode?ignoreValidityChecks=true` and with tests that rebuild real transactions byte-for-byte.
- Prove writes on a private **testnet** (refuse to write unless `/admin/info` says `isTestNet: true`). This is advanced; most apps never need it.

### 8.10 After any transaction

- Show "Confirming…" at once.
- Poll politely (visible tab only, backoff) until the transaction has a `blockHeight` (`/transactions/signature/{sig}`), then refresh affected data once.
- Unconfirmed transactions can linger and expire 24 h after their timestamp.

---

## 9. Links and navigation

### 9.1 `qortal://` link format

```
qortal://<SERVICE>/<Name>[/<identifier>][/<path>]
qortal://APP/My App+/item/Alice/myapp_post_abc     ← how users see it
qortal://APP/My%20App+/item/Alice/myapp_post_abc   ← how to write it: spaces %20, "+" literal
```

- **Write the app name raw**: only spaces as `%20`. Hub never decodes this segment: `qortal://APP/My%20App%2B/…` opens a tab at `/render/APP/My%20App%2B/…` that doesn't match `_qdnBase="/render/APP/My%20App+"`, and the app renders **blank** unless its router base tolerates it (9.3). `qortal://APP/My%20App+/…` works. Hub's own "Copy link" produces this same form.
- **Encode each path segment** with `encodeURIComponent` (user names in paths contain `/`, `#`, spaces).
- A third segment counts as an identifier only if that resource exists (q-apps.js checks with a **synchronous** status request on every click). Avoid first path segments that could equal a real identifier of the same name.
- App names containing `/` can't be linked reliably.
- Keep target app names (your own published name; other apps you link to) in one constants module. In Dev Mode `_qdnName` is `""`: fall back to the published name.

```ts
export const PUBLISHED_NAME = 'My App+';
export function appName(): string {
  const n = (window as any)._qdnName as string | undefined;
  if (n && n.trim()) { try { return decodeURIComponent(n); } catch { return n; } }
  return PUBLISHED_NAME;
}
const appSegment = (name: string) => name.replace(/ /g, '%20');
export const appLink = (path: string, name = appName()) =>
  `qortal://APP/${appSegment(name)}/${path.replace(/^\/+/, '')}`;
export const itemLink = (publisher: string, id: string) =>
  appLink(`item/${encodeURIComponent(publisher)}/${encodeURIComponent(id)}`);
```

### 9.2 What q-apps.js does with clicks

- `<a href="qortal://…">` → `LINK_TO_QDN_RESOURCE`. Same app: full-frame navigation (a reload; use your router for in-app moves). Another app: Hub opens/switches to a tab (`SET_TAB`); if Hub doesn't confirm within 200 ms, q-apps.js navigates your frame there.
- `<a href="http(s)://…">` and `//…` → **blocked** (`preventDefault`, nothing happens).
- Relative `<a href>` navigates the frame to another node page: use router links instead.
- `<img src="qortal://…">` is rewritten only for images present at `DOMContentLoaded`, and only the **first** `<img>` on the page gets updated. Don't use it.

### 9.3 Router basename (APP)

- `BrowserRouter` (or a data router) with `basename` = `_qdnBase` works, because Core serves `index.html` for unknown APP paths.
- Hub loads the frame with the name **as the link spelled it** (`My%20App%2B`), while `_qdnBase` has Core's spelling (`My%20App+`). React Router renders **nothing** when the basename doesn't prefix the path. Non-ASCII names add a second trap: Core injects `_qdnBase` raw (`/render/APP/Νίκος`; only spaces become `%20`) while `location.pathname` is always percent-encoded (`/render/APP/%CE%9D…`). Compare decoded, rewrite the URL to Core's spelling, and if the two still differ only by encoding, use the URL's own spelling:

```ts
const dec = (v: string) => { try { return decodeURIComponent(v).toLowerCase(); } catch { return v.toLowerCase(); } };
export function routerBase(qdnBase = (window as any)._qdnBase as string | undefined): string {
  if (!qdnBase) return '';                                    // Dev Mode or plain browser
  const n = qdnBase.split('/').length;
  const head = () => location.pathname.split('/').slice(0, n).join('/');
  let prefix = head();
  if (prefix === qdnBase || dec(prefix) !== dec(qdnBase)) return qdnBase;
  // Same app, other spelling ("My%20App%2B"): rewrite the URL to Core's spelling, no new history entry.
  try { history.replaceState(history.state, '', qdnBase + location.pathname.slice(prefix.length) + location.search + location.hash); } catch {}
  // Core injects non-ASCII names raw ("/render/APP/Νίκος"), but the browser keeps location.pathname
  // percent-encoded, so the rewrite can't make them equal. Then use the URL's own spelling, and
  // update _qdnBase (a plain var) so q-apps.js can still strip it when it reports routes to Hub.
  prefix = head();
  if (prefix !== qdnBase && dec(prefix) === dec(qdnBase)) { (window as any)._qdnBase = prefix; return prefix; }
  return qdnBase;
}
```

- `WEBSITE`: use `HashRouter` (`#/page`) or a real file per route; `qortal://WEBSITE/Name/#/page` works.
- In the `/render/hash/…` preview there is no SPA fallback: reload at the root.
- Prefer **path or hash routes over query-string routes**: q-apps.js reports only `pathname + hash` to Hub, so Back and Copy link lose `?query` state.

### 9.4 Back: `NAVIGATE_TO_PATH` and your own stack

- q-apps.js reports every route change to Hub automatically (Navigation API). You don't post routes yourself.
- For its Back button, Android Back in GO, and deep links into an open tab, Hub posts `{action:'NAVIGATE_TO_PATH', path}` and waits **~200 ms** (250 ms for other navigation) for `{action:'NAVIGATION_SUCCESS', path}`. No answer → Hub **reloads the frame** at that path and the app loses its state.

```ts
// Mount once inside the router
export function useHubNavigation() {
  const navigate = useNavigate();
  useEffect(() => {
    const onMessage = (e: MessageEvent) => {
      if (!fromHost(e)) return;
      const d = e.data as { action?: string; path?: unknown };
      if (d?.action === 'NAVIGATE_TO_PATH' && typeof d.path === 'string') {
        navigate('/' + d.path.replace(/^\/+/, ''));                                  // path may lack a leading '/'
        window.parent.postMessage({ action: 'NAVIGATION_SUCCESS', path: d.path }, '*'); // echo exactly, at once
      }
    };
    window.addEventListener('message', onMessage);
    return () => window.removeEventListener('message', onMessage);
  }, [navigate]);
}
```

- **Never use `window.history.back()`, `history.length` or `navigate(-1)` for your Back button.** All Hub tabs share one browser history; you can move a hidden tab. Keep your own stack of in-app paths and fall back to a parent route with `replace` (code: `useSafeBack` in Template B `src/hub/navigation.tsx`).
- **Known limitation (traced in source, not yet seen in a running Hub):** an in-app Back is a new navigation, and q-apps.js reports it to Hub like any other route. After Feed → Recipe → in-app Back, Hub's history is `[/, /recipe, /]`, so the next Hub or Android Back returns to the recipe the user just left. On phones, make Android Back the primary way back and avoid layouts that need an in-app Back button; add "in-app Back, then Android Back" to your test matrix (15.4).
- New page → scroll to top on PUSH; restore the saved position on POP.
- Full-screen dialogs/sheets on phones: give each its own route (or history entry) so Back closes it; close on POP only after it has seen its own entry (otherwise overlays that mount already open close at once).
- Swipe-back gestures (optional): close the top in-app layer; ignore vertical drags, long presses, horizontal scrollers and marked zones.

### 9.5 Opening other apps and resources

```ts
await qortal({ action: 'OPEN_NEW_TAB', qortalLink: 'qortal://APP/Other%20App/item/x' });  // Hub; returns true
await qortal({ action: 'LINK_TO_QDN_RESOURCE', service: 'APP', name: 'Other App', path: 'item/x' });
```

- `OPEN_NEW_TAB` accepts only `qortal://` links (`qortal://SERVICE/Name/path?identifier=X`; a bare `qortal://Name` means WEBSITE), plus `navigateIfAlreadyOpen?`. Web URLs are rejected.
- Link to the app you mean on purpose (the original public app vs a fork).

### 9.6 Web (http/https) links

No action opens a web URL, link clicks are blocked, and popups are not allowed. Copy the URL to the clipboard and show "Link copied, paste it in your browser":

```ts
document.addEventListener('click', (e) => {
  const a = (e.target as Element).closest?.('a[href^="http"]') as HTMLAnchorElement | null;
  if (!a) return;
  e.preventDefault();
  navigator.clipboard?.writeText(a.href).then(() => toast('Link copied: paste it in your browser'));
});
```

---

## 10. Files and media

### 10.1 Saving files

| Situation | Call |
|---|---|
| Desktop Hub, small file you have as a Blob | `SAVE_FILE {filename, blob, mimeType}` → saves to Downloads (one dialog) |
| GO, or any file ≥ ~100 MB, or a file already on QDN | `SAVE_FILE {filename, mimeType?, location:{service, name, identifier}}` → Hub/GO streams from the node (GO writes natively to Documents/Qortal Go with progress); on desktop adds a Save-As dialog |
| Encrypted QDN file | decrypt first, then save the **blob** (a location save writes ciphertext) |

- `filename` is required; Hub needs a MIME type (`blob.type` or `mimeType`), or it throws.
- **A declined save rejects with a string ("user declined to save file")**: treat as quiet cancel.
- GO strips location-save file names to `A-Z a-z 0-9 space _ -`; a name with none of those becomes a hidden `.pdf`-style file: give files ASCII names.
- Generating several files: build a store-only ZIP (CRC-32, UTF-8 names) as a Blob; no library needed. Cap total size (e.g. 200 MB).

### 10.2 PDFs

- Inline `<iframe src=…pdf>`/`<embed>` stays **blank**: Hub's Electron window has no PDF plugin and GO's WebView has no viewer.
- Use `SHOW_PDF_READER {blob}` (Hub/GO); the blob's type must be exactly `application/pdf` (`blob.slice(0, blob.size, 'application/pdf')`). Check for the `%PDF-` header first.
- Fallback: bundled pdf.js (~1.7 MB, its own lazy chunk) with the worker on the main thread (import the worker module and set `globalThis.pdfjsWorker`). Lazily loaded worker files have failed to load in Hub (observed).

### 10.3 Images

- Build URLs: `/arbitrary/IMAGE/${encodeURIComponent(name)}/${encodeURIComponent(id)}?async=true` (or `THUMBNAIL`).
- Lazy-load with IntersectionObserver (native `loading="lazy"` is fine on a plain `<img>`, but UI-kit avatar/image components may preload on mount).
- `async=true` returns 404 while the node fetches: retry with backoff (6.6/6.7).
- **Never publish the raw camera file.** Downscale on a `<canvas>` first: e.g. photo 1600 px / ≤ 2 MB → `IMAGE` (Core cap 10 MiB), thumbnail 400 px / ≤ 450 kB → `THUMBNAIL` (cap 500 KiB), WebP with a JPEG fallback, lowering quality until under the size. Publish them with the JSON in **one** `PUBLISH_MULTIPLE_QDN_RESOURCES` call, JSON last, with `file:` Blobs and sibling identifiers (7.2). Code: Template B `src/qortal/image.ts`.
- Cap decoded image size (refuse > 50 MP) to avoid out-of-memory on phones.
- Fix EXIF orientation by drawing through `createImageBitmap(file, {imageOrientation: 'from-image'})` where supported (fall back to an `<img>`).
- File inputs for photos: `accept="image/*"` without `capture`, so phones offer both gallery and camera.

### 10.4 Video and audio

- `<video src="/arbitrary/VIDEO/<encoded name>/<encoded id>" controls preload="metadata">`: Core serves byte ranges. Check status first for big files and show "Fetching from peers…".
- Core doesn't clamp an over-long `Range` end to the file size and doesn't expose `Content-Range` cross-origin (observed): cut custom ranges to the known size.
- Fullscreen video works in Hub and GO.
- `PLAY_ENCRYPTED_MEDIA` works only in Electron Hub (not GO).
- No microphone or camera access (`getUserMedia` is denied); use `<input type=file accept="video/*" capture>` to record in GO.

### 10.5 Picking and uploading files

- Use a plain `<input type="file">` (works in GO, incl. camera capture). Not the File System Access API.
- HTML5 drag-and-drop doesn't work by touch: implement reordering with pointer events; keep a non-drag alternative (up/down buttons).
- Publish binary with `file:` (File/Blob), not base64.
- Show size against the service cap before publishing.

### 10.6 Size limits

| Limit | Value |
|---|---|
| Hub publish cap per file | 2 GB; **500 MB on a public node** |
| `APP` | 50 MB (zip contents) |
| `IMAGE` | 10 MB |
| `THUMBNAIL`, `COMMENT`, `BLOG_COMMENT`, `QCHAT_IMAGE` | 500 KB |
| `JSON` | 25 KB, must be valid JSON |
| `MAIL_PRIVATE` | 5 MB |
| `ATTACHMENT(_PRIVATE)` | 50 MB |
| `FILE`, `DOCUMENT`, `VIDEO`, `AUDIO` | no service cap (Hub's cap applies) |
| Core chunk size | 512 KB chunks; max file 2 GB |

Full service table: 19.3.

---

## 11. Lists, notifications, chat

### 11.1 Lists (node-local)

- `GET_LIST_ITEMS {list_name}`, `ADD_LIST_ITEMS {list_name, items}`, `DELETE_LIST_ITEM {list_name, item}` (singular `item`; some Hub versions also take `items`).
- Lists live **on the node** (`/lists/<name>`), shared by every app on that node, not per account, and **don't follow the user** to another node or device.
- **Refused on public nodes** (GO default). Degrade quietly, and use the favourites pattern below instead.
- GO rejects list names over 128 characters, containing `/` or `\`, or starting with `.`; keep names short ASCII with your app slug as prefix. Core's search `followedonly`/`excludeblocked` read the node's own follow/block lists.
- `ADD`/`DELETE` show a dialog every time.

**Favourites / follows pattern (works on public nodes):**
1. Keep them **local first** in `localStorage` under `myapp-fav-<address>`. Say in the UI that this copy is per node and per device: the app runs at `<node>/render/...`, so its storage belongs to that node's origin, and switching Hub/GO between nodes shows an empty list.
2. Offer an explicit **"Save favourites to QDN"** button (0.01 QORT, needs a name; hide it for accounts without one): publish one `DOCUMENT_PRIVATE` with `encrypt: true, publicKeys: [myPublicKey]` (encrypted to yourself only).
3. Before each save, read the QDN copy fresh and merge (7.14); on a new device, offer "Restore from QDN".

### 11.2 Notifications (Hub/GO alert rules)

```ts
await qortal({ action: 'NOTIFICATION_PERMISSION' });          // must be called in each tab session before ADD
await qortal({ action: 'NOTIFICATION_ADD', notifications: [{
  notificationId: 'myapp-replies',
  link: 'qortal://APP/My%20App+/inbox',
  filters: { service: 'JSON', identifier: 'myapp_reply_', prefix: true, description: '~to:3f9a1c2e~', mode: 'ALL' },
  message: { en: 'New reply in My App', de: 'Neue Antwort in My App' },
}]});
```

- Rules fire when Core reports a newly published resource matching the filter (a search spec: `service`, `query`, `identifier`, `names[]`, `title`, `description`, `keywords[]`, `prefix`, `defaultResource`, `followedOnly`, `excludeBlocked`, `after`, `before`, `mode`; unknown keys throw). They match **metadata, never the body**. The user's own publishes are skipped.
- To alert one person, put a marker in the **description** at the start (e.g. `~to:<16 hex of SHA-256(name)>~`) and match it with `prefix`. This marker is public and can be reversed (anyone can hash every registered name), so anyone can see who is alerted. Use it only where the recipient is not private. For private messaging, don't put a recipient marker in metadata: alert on a per-recipient identifier prefix only if that is acceptable, or skip notifications.
- `NOTIFICATION_HAS_PERMISSION` (no dialog), `NOTIFICATION_GET`, `NOTIFICATION_REMOVE {notificationIds?}` (none = all of this app's rules), `NOTIFICATION_MARK_SEEN`.
- Rules belong to the **app name**; rules registered under an older app name stay with it.
- Ask for permission only when the user turns alerts on, never at sign-in. Feature-detect with `SHOW_ACTIONS`.

### 11.3 Chat

- `SEND_CHAT_MESSAGE {message | fullMessageObject, recipient | destinationAddress | groupId, chatReference?}`:
  - plain `message` is wrapped by Hub into Qortal chat's rich-text format (`{messageText, images:[], repliedTo:'', version:3}`);
  - direct messages are encrypted and need the recipient's public key on chain; **group messages sent this way are not encrypted**;
  - needs ≥ 4 QORT balance; dialog with "always allow" per app; may resolve `false` on failure.
- `SEARCH_CHAT_MESSAGES {txGroupId | involving:[addrA, addrB], before, after, limit, offset, reverse, encoding?}`: exactly one of `txGroupId` or two `involving` addresses; `before`/`after` ≥ 1.5e12. **Direct message bodies come back encrypted and no action decrypts them.**
- Core 6.1.5+ keeps chat in memory and drops messages older than ~24 h (from source; active groups returned `[]` live). Rate limits apply (Core 6.1.4: 250 messages/account/hour, ≤ 4000 bytes). **Never use chat as storage**; use QDN.
- Live updates: websocket `/websockets/chat/messages` on the node (same origin).

---

## 12. UI/UX that feels native in Hub and GO

### 12.1 Sizes and breakpoints

Design at 360-430 px first. Check every screen at:

| Size | Represents |
|---|---|
| 360 × 740 | small Android phone (GO) |
| 390 × 844 | common phone |
| 844 × 390 | phone landscape: the Hub/GO frame is only **~266 px** tall |
| ~700 wide | Hub with a narrow pane / tablet portrait |
| 1280-1440 wide | desktop Hub |

- **Phone layout below 600 px wide, whatever the device**, plus touch screens shorter than 500 px:
  `const PHONE = '(max-width: 599.95px), ((pointer: coarse) and (max-height: 500px))'`
- Landscape phone: bar ≤ 52 px, no floating button, one pane at a time.
- Don't switch layout when the keyboard opens: GO shrinks the frame for the keyboard, so "height < 500" is also true for a portrait phone while typing. Detect the keyboard as "a text field has focus and the frame is well under its tallest height at this width". A compact bar only when short **and** wide: `(pointer: coarse) and (max-height: 500px) and (min-width: 600px)`.
- No sideways page scroll at any size (test it).

### 12.2 Frame height, safe areas, keyboard

- Height from `--app-height` (4.5), never `100vh`.
- `viewport-fit=cover` + `env(safe-area-inset-bottom)` padding on bottom bars: `height: calc(56px + env(safe-area-inset-bottom, 0px))`.
- Keep the focused input and its Send button above the keyboard with `visualViewport`:

```ts
const read = () => ({ height: Math.round(window.visualViewport?.height ?? window.innerHeight),
                      offsetTop: Math.round(window.visualViewport?.offsetTop ?? 0) });
// subscribe to visualViewport 'resize' + 'scroll' and window 'resize';
// a full-screen phone dialog uses: position:fixed; top:offsetTop px; height:height px
```

### 12.3 Scroll containers and sticky headers

- `overflow-x: hidden` on `html`/`body` turns `body` into a scroll container and **sticky headers scroll away**.
- `overflow-x: clip` on `html` makes the page **jump ~16 px** when a UI kit's scroll lock opens a menu/dialog.
- **Use `overflow-x: clip` on `#root` only.**
- Prefer one page scroll (the document) over nested scroll areas on phones; nested scrollers fight with GO's hide-on-scroll bars.

### 12.4 Hide-on-scroll headers

They oscillate near the end of short pages (hiding frees height → content pulls back → reads as scroll-up; seen flipping 68-128 times in GO). If you build one: ignore pull-backs, ignore scroll for ~300 ms while the header animates, don't hide on pages shorter than three header heights, always show it at the top.

### 12.5 Touch

- Tap targets ≥ 44 × 44 px (some UI-kit menu items are 36 px on wide screens: override).
- **Nothing hover-only.** Row actions visible (or in a ⋮ menu) on touch.
- Long-press menus: 500 ms, cancelled by a 10 px move. iOS WebViews fire no `contextmenu`; implement long-press yourself.
- Use `touch-action: manipulation` on buttons to avoid double-tap zoom delay.
- Inputs: `font-size ≥ 16px` on phones (prevents zoom on focus), right `inputmode`/`type` (`decimal` for amounts), `enterkeyhint`.

### 12.6 Overlaps

- Floating buttons and toasts must never cover content or the last list item: leave ~88 px clearance under lists when a FAB is shown; put toasts above the bottom bar.
- One FAB at most; none in landscape phone.

### 12.7 Dialogs on phones

- Full-screen dialogs or bottom sheets with actions **kept in view** (sticky footer). Words in buttons stay whole (no mid-word wraps).
- Back closes the top dialog (9.4).
- Before any Hub dialog (publish, pay), show your own summary: what will be published/paid, the fee, and that Hub will ask to confirm. Then disable the button while waiting.

### 12.8 Themes and contrast

- All colours from theme tokens; both light and dark checked; **4.5:1** text contrast.
- User-generated rich text: **strip inline `color`/`background` styles on display** (white text pasted from dark pages is invisible in light theme).
- Respect `prefers-reduced-motion`; keep motion 150-200 ms.
- Hub's look: rounded cards, clear hierarchy, compact bars. Don't fight the host's chrome; your app is a tab inside it.

### 12.9 UI-kit pitfalls (MUI and similar)

| Pitfall | Fix |
|---|---|
| `SwipeableDrawer` forces `keepMounted`: one sheet per row added 458 of 2,020 DOM nodes | mount the drawer on first open |
| `Avatar src=` loads on mount regardless of `loading="lazy"` | a plain `<img>` that gets its `src` when in view |
| `useMediaQuery` or a `Tooltip` per row in long lists (300 rows: 119 ms vs 20 ms) | CSS breakpoints; mount tooltips on first hover/focus |
| Absolute-positioned Popper inside a frame whose body is taller than the viewport opens ~250 px off | `position: fixed` strategy |
| Menus with `disablePortal` make the kit `aria-hide` the page | keep the portal |
| Autocomplete `freeSolo` never shows `noOptionsText` | render your own empty row |
| MUI `sx` `width: 1` means 100 % | write `'1px'` |
| MUI 9 removed `…Outline` icon names and system props on components | use `…Outlined` icons, `sx` |
| Scroll lock + `html { overflow-x: clip }` shifts page | clip `#root` (12.3) |
| Scrollable Tabs can look scrolled to the end in full-page screenshots | check `scrollLeft` before "fixing" |

### 12.10 Loading, empty and error states

- **Loading**: skeletons that match the final layout; for QDN data, "Fetching from peers…" with progress when known (6.6).
- **Empty**: say what this is and offer the next action ("No posts yet. Write the first one").
- **Error**: plain words + Retry. Never raw JSON, never `[object Object]`.
- **Error boundaries per card/route**: one malformed document must not blank the page for everyone (Template B `ErrorBoundary.tsx`).
- **Not signed in / no name / public node**: explicit, friendly states, not errors.
- First paint: inline the logo (SVG/data URI) and the background colour in `index.html`.

### 12.11 Accessibility

- Every icon button has an `aria-label` that **contains its visible label** if any.
- Visible focus rings; keyboard navigation works in desktop Hub.
- `lang` attribute follows the UI language; `dir="rtl"` for Arabic.
- Images: meaningful `alt`; decorative `alt=""`.
- Live regions for toasts (`role="status"`).

### 12.12 GO full screen (optional)

`SET_FULL_SCREEN {enabled: true|false}` hides GO's bars for the tab (GO only). Desktop Hub has no handler, so the request hangs: send it with `qortalRequestWithTimeout(req, 1500)` (ignore errors) and only on phones. GO posts `FULL_SCREEN_CHANGED {payload:{enabled:false}}` when the user leaves full screen.

---

## 13. Security and trust

| Threat | Rule |
|---|---|
| Spoofed authorship | Publisher = the **search row's** `name`. A body can claim any author, date or role. Key caches by name + identifier |
| Lookalike identifiers | Anyone can publish your prefix under their name: filter by expected publisher (owner name, group admin list) at read time |
| Malformed bodies | Validate every field (type, length, range); render only string fields as text; normalise on read |
| Stored XSS | Prefer plain text (`textContent`). For HTML: DOMPurify (current 3.x; ≥ 3.4.17 per recent advisories), forbid `style`, `form`, `input`, `button`, `select`, `textarea`, `map`, `area`; post-process **on the DOM** (TreeWalker), never with regex over sanitised HTML (a real stored XSS came from exactly that); drop relative/same-origin links; render inside a container with `contain: paint` |
| Generated HTML (e.g. an order email body) | HTML-escape every typed value |
| Hostile links | `qortal://` links → normalise lowercase scheme; http(s) → copy-on-click; others removed |
| Referenced data triggering actions | Bind sensitive actions (decrypt, forward, pay) to what the user sees, never to IDs read from a body |
| Eligibility | Re-check roles at read time (group membership, name ownership); ignore records whose identifier doesn't match the body's claimed id |
| Keys and seeds | Never ask for, display, store or transmit them. All signing is done by Hub |
| Shared origin | Every Q-App on the node shares `localStorage`/IndexedDB/cookies. Prefix keys (`myapp-…`), store no secrets; tell users if something they save (e.g. a shipping address) is readable by other apps |
| Secrets in metadata | Title/description/tags are public even on encrypted resources |
| Coupon/discount codes | Publish salted SHA-256 fingerprints of **long random codes** (≥ 12 random characters) with a per-app-instance salt; short or guessable codes (`SUMMER10`) can be recovered offline because the salt must be public. Keep the plain list encrypted to the owner |
| Writes | Confirm before every write with what/where/cost; never auto-publish or auto-pay |
| Impersonation | Flag invisible-character names (5.2); show the address on hover/tap for important identities |
| Votes/likes/reviews | Soft signals only: anyone with a name and 0.01 QORT can publish |

```ts
import DOMPurify from 'dompurify';
const ALLOWED_URI = /^(?:(?:(?:f|ht)tps?|mailto|tel|qortal):|[^a-z]|[a-z+.-]+(?:[^a-z+.:-]|$))/i;
const COLOUR = /^(?:color|-webkit-text-fill-color|background(?:-.+)?)$/;
export function sanitizeHtml(html: string): string {
  const frag = DOMPurify.sanitize(html, {
    USE_PROFILES: { html: true }, ALLOWED_URI_REGEXP: ALLOWED_URI, RETURN_DOM_FRAGMENT: true,
    FORBID_TAGS: ['style', 'form', 'input', 'button', 'select', 'option', 'textarea', 'map', 'area'],
    FORBID_ATTR: ['color', 'bgcolor'],
  }) as DocumentFragment;
  frag.querySelectorAll<HTMLElement>('[style]').forEach(el => {
    for (let i = el.style.length - 1; i >= 0; i--) if (COLOUR.test(el.style[i])) el.style.removeProperty(el.style[i]);
    if (!el.style.length) el.removeAttribute('style');
  });
  frag.querySelectorAll('a').forEach(a => {
    const href = a.getAttribute('href') ?? '';
    if (/^qortal:\/\//i.test(href)) { a.setAttribute('href', 'qortal://' + href.slice(9)); return; }
    if (/^https?:/i.test(href)) { a.setAttribute('data-copy-link', ''); a.removeAttribute('target'); return; }
    a.replaceWith(...Array.from(a.childNodes));
  });
  const box = frag.ownerDocument.createElement('div');
  box.append(frag);
  return box.innerHTML;
}
```

---

## 14. Performance

### 14.1 Budgets

| Item | Target |
|---|---|
| First-load QDN searches | as few as possible; count them in a test |
| Bodies fetched on first load | only those on screen (one real app: 60 → 12 GETs; another: 20 → 6) |
| Parallel body fetches | ~5; heavy address searches ~2; name lookups ~4 |
| Largest JS chunk | well under 1 MB (real apps went 1.12 MB → 265 kB, 1,030 kB → 341 kB) |
| Zip | as small as possible: it is fetched from **peers** before first paint |
| Search page size | ~20 in UI, ≤ 100 in background |

### 14.2 Requests

- One search helper with in-flight merging and a 1-3 min session cache (6.8); invalidate after publishing.
- Narrow every query: `name` + `exactmatchnames`, `identifier` + `prefix`, `includemetadata` only when shown, `after=` for recent windows.
- Merge per-item lookups into multi-`name` searches.
- Never `limit: 0` / missing `limit`. Never unpaged `/transactions/address/{a}`.
- Avoid `GET_QDN_RESOURCE_URL` per row (extra status request); build URLs.
- Use `?async=true` for images/avatars so missing data doesn't hold node threads for seconds.

### 14.3 Polling

```ts
export function poll(task: () => Promise<boolean | void>, base: number, max = base * 8) {
  let timer: any, stopped = false, running = false, failures = 0, missed = false;
  const delay = () => Math.round(Math.min(max, base * 2 ** Math.min(failures, 10)) * (1 + 0.1 * Math.random()));
  const tick = async () => {
    if (stopped) return;
    if (document.visibilityState === 'hidden') { missed = true; return; }
    if (!running) {
      running = true;
      try { failures = (await task()) === false ? failures + 1 : 0; } catch { failures++; }
      finally { running = false; }
    }
    if (!stopped) timer = setTimeout(tick, delay());
  };
  const onVis = () => { if (document.visibilityState !== 'hidden' && missed) { missed = false; tick(); } };
  document.addEventListener('visibilitychange', onVis);
  timer = setTimeout(tick, delay());
  return () => { stopped = true; clearTimeout(timer); document.removeEventListener('visibilitychange', onVis); };
}
```

- No fixed short `setInterval`. Stop on unmount. Prefer websockets for blocks/chat where available.
- UNVERIFIED whether a background Hub tab reports `visibilityState === 'hidden'`; `requestAnimationFrame` loops stop in hidden documents, so keep a timer fallback for anything that must progress.

### 14.4 Bundle

- Lazy-load routes and heavy libraries (rich-text editor, pdf.js, charts, emoji, video player, country lists, crypto).
- Replace moment.js with `Intl`; subset fonts (one font went 337 KB → 58 KB) or use system fonts; ship only `woff2`.
- Don't import a big module into the main chunk for one constant (put shared constants in a tiny module).
- No source maps and no `.gz`/`.br` files in the zip (Core doesn't serve precompressed files; they only add weight).
- Split vendor chunks (React, UI kit) so app updates re-download less.
- Inline the logo for a request-free first paint.

### 14.5 Rendering

- Virtualise lists over a few hundred rows.
- Avoid per-row media queries, tooltips and heavy components.
- Decrypt, parse and format only visible rows.

### 14.6 Surviving updates (stale chunks)

After you republish, open tabs request old chunk names; Core answers `index.html` with **200** (APP fallback) or 503 while preparing the new build, and `import()` fails. Retry once, then reload once (guarded), then show an error boundary with a Reload button:

```ts
import { lazy, type ComponentType } from 'react';
const isLoadError = (e: unknown) =>
  /dynamically imported module|Importing a module script failed|error loading dynamically imported|Unable to preload CSS|Loading (CSS )?chunk|Unexpected token '<'/i
    .test(e instanceof Error ? e.message : String(e ?? ''));
function reloadOnce(): boolean {
  try {
    const last = Number(sessionStorage.getItem('myapp-chunk-reload') ?? 0);
    if (Date.now() - last < 60_000) return false;
    sessionStorage.setItem('myapp-chunk-reload', String(Date.now()));
  } catch { return false; }
  location.reload(); return true;
}
export function lazyWithRetry<T extends ComponentType<any>>(load: () => Promise<{ default: T }>) {
  return lazy(async () => {
    try { return await load(); }
    catch (e) {
      if (!isLoadError(e)) throw e;
      await new Promise(r => setTimeout(r, 2000));
      try { return await load(); }
      catch (again) { if (isLoadError(again) && reloadOnce()) return new Promise<never>(() => {}); throw again; }
    }
  });
}
window.addEventListener('vite:preloadError', (e) => { if (reloadOnce()) e.preventDefault(); });
```

---

## 15. Testing

### 15.1 Outside Hub (fast loop)

- **Unit tests** (vitest + jsdom): a global `qortalRequest` mock that answers by action and records calls, plus a `fetch` mock routed by URL regex (Template B `test/setup.ts`). Test: payload builders (exact shapes), validators, identifier parsing, search URL building (encoding!), error normalisation, and **a first-load test that counts searches**.
- **Mock every Core path** your app uses; built assets also live under `assets/`, so route Core's `/assets/<x>` API calls precisely.
- **Dev preview against a real node, read-only**: run `vite dev` with a **GET-only** proxy of `/arbitrary`, `/names`, `/addresses`, `/transactions`, `/groups`, `/blocks`, `/admin/info`, `/crosschain`, `/polls` to `http://127.0.0.1:12391` (Template B `vite.config.ts`), plus a dev-only `qortalRequest` shim that answers read actions and **refuses every write** (Template B `devShim.ts`).
- **Open the production build in a real browser** after any dependency upgrade: some libraries pass type-check and build yet blank the app at runtime (e.g. a React 18-only audio player under React 19). Grep `dist` for `__SECRET_INTERNALS`: expect 0 hits with React 19.
- Screenshot run (Playwright) of the **built** app with mocks: 5 sizes × light/dark, `hasTouch` + `(hover: none)`/`(pointer: coarse)` emulation, checks for console errors, horizontal overflow, unlabelled buttons, targets < 44 px, axe. Playwright hides scrollbars, so it can't show scroll-lock shifts; check those by hand.

### 15.2 Hub Dev Mode

Needs Hub connected to a **local node** (Dev Mode refuses a public node) and, for previews, a registered name.

| Mode | How | Use for | Quirks |
|---|---|---|---|
| **Server** | Run `npm run dev` (Vite on `127.0.0.1:5173`), Hub → Dev Mode → Server → `127.0.0.1:5173` → Add | Day-to-day work against real Hub | Core's dev proxy (port 12393) forwards GETs only; q-apps.js injected only at `/` and `*.html` (**reload at `/`**); `/assets/*` 404s through the proxy (so `vite preview` of a build fails: use `vite dev`, or build with `--assetsDir static` for this test only); **no HMR** (reload the tab after edits); after Vite re-optimises deps, two Reacts may load ("Invalid hook call": restart Vite and re-add); the proxy dies when Core or Vite restarts (re-add the server); `_qdnName=""`; CSP has no `wss:`/`blob:` connect |
| **Zip** | Hub → Dev Mode → Zip → pick `release.zip` | **The exact release zip**, without publishing | Served at `/render/hash/<hash>/…`: no SPA fallback, so reload at root |
| **Directory** | Hub → Dev Mode → Directory → pick `dist/` | quick check of a build | same as Zip |

- Don't tick "always allow" in Hub's dialogs while testing in Dev Mode, and load only builds you trust.
- Writes in Dev Mode are **real** (real fees, real data on mainnet). Use a test name, and confirm every publish/payment deliberately.
- Hot reload can re-show your own prompts (e.g. "Load saved settings?"); answer carefully so test data isn't applied.

### 15.3 GO testing

- Turn Dev Mode on first (GO: Settings → System → Dev Mode; off by default), then load the zip through GO's Dev Mode file picker (Zip), or publish and open it. GO builds the picked zip on the phone and serves it at `<node>/render/hash/<id>`, so the preview quirk applies (no SPA fallback: reload at the root). Unlike Hub Dev Mode, it also works on a public node.
- Optional: point GO at a desktop node via `adb reverse tcp:12391 tcp:12391`; inspect the WebView via Chrome DevTools (`chrome://inspect`, WebView debugging socket).
- GO blocks screenshots (FLAG_SECURE): screencaps are black.
- Test with the keyboard open, landscape, the bars hiding on scroll, Android Back, a public node (lists refused) and an old WebView.

### 15.4 Release test matrix

| Area | Check |
|---|---|
| Boot | outside Hub: friendly message, no console errors; in Hub: correct theme on first paint, after reload, and after Hub's light/dark switch |
| Routing | deep link opens the right page; reload on a deep route; Hub Back and Android Back (no app reload); in-app Back, then Android Back (9.4); Copy link round-trips; app names with `+`/space/non-ASCII in links |
| Identity | sign-in on load and by button; decline; timeout; account switch clears derived data; no name; many names |
| Reads | lists show **many items per publisher** (mode ALL); paging; names with `+`, space, `/`, non-ASCII; data not on node → "Fetching from peers" → shows or "not available"; deleted items hidden |
| Writes | fee shown; decline is quiet; timeout → verify, no double publish; > 20 resources → batches; new item appears optimistically then from search |
| Money | validation messages; decline; timeout → payment lookup; no double charge |
| Layout | 360×740, 390×844, 844×390, ~700, 1280+; keyboard open; no sideways scroll; tap targets; dialogs' buttons visible |
| Themes | light/dark contrast; user content readable in both |
| Public node | `IS_USING_PUBLIC_NODE` checked; list/hosted-data features hidden or explained, in a non-English Hub language too |
| Perf | count first-load searches/fetches; biggest chunk; zip size |
| Update | open tab across a republish recovers (reload once) |

---

## 16. Packaging and publishing

### 16.1 Build config

- Vite: `base: ""`, `build.target: 'es2020'`, `sourcemap: false`, vendor chunks (Template B).
- `index.html` contains the boot script and `<meta name="viewport" …viewport-fit=cover>`; no CDN tags; no `<base>`.
- Fonts, icons, images bundled (or `data:` URIs). Total zip well under the 50 MB APP cap; smaller is faster.
- App identity in one module: published name, link name, storage-key prefix, settings identifier, version.

### 16.2 Zip layout

```
release.zip
├── index.html          ← at the ZIP ROOT (not inside dist/)
├── favicon.ico
└── assets/
    ├── index-3f9a1c.js
    └── index-a7b2e4.css
```

No `node_modules`, no `.map`, no `.gz`/`.br`, no `.env`, no `manifest.json` needed (nothing reads it). Script: Template C.

### 16.3 Publishing

1. In Hub, open the app-publishing screen (Apps → publish an app), choose service **APP**, your **name**, the **zip**, and fill title, description, category and tags (these become the app's QDN metadata, shown in Hub's app library). Fee 0.01 QORT.
2. Publish under the **default identifier**.
3. Set an avatar for the publishing name (it is the app's icon in Hub's library).
4. Open `qortal://APP/<Name>` and run the smoke tests (15.4) on the live build.
5. WEBSITE: same flow with service WEBSITE; the zip root must contain an index file.
6. From code: `PUBLISH_QDN_RESOURCE {service:'APP'|'WEBSITE', name, file: new File([zipBlob], 'app.zip'), filename:'app.zip', isMultiFileZip:true, title, description}`.

### 16.4 Verifying the live build

- `GET /arbitrary/metadata/APP/<encoded name>/default` → `files` list. Compare (sorted with `LC_ALL=C sort`) with your `dist` file list: content-hashed names make equal lists mean the same build.
- `GET /arbitrary/resource/status/APP/<name>?build=true` → `READY`.
- History of publishes: `/names/<name>` → `owner`, then `/transactions/search?txType=ARBITRARY&address=<owner>&confirmationStatus=BOTH&limit=20&reverse=true` (service shows as number `1000` for APP).

### 16.5 Versions and updates

- Semantic versions from 1.0.0; bump the patch per published update; keep `package.json`, the About screen and a changelog in step.
- Each publish replaces the app for everyone; there's no rollback except republishing the old zip (keep your release zips).
- Open tabs may hold old chunks: 14.6.
- Data changes stay backward compatible (7.2): old app versions still running in other tabs/nodes must keep working with new data.

---

## 17. The big mistakes table

| # | Symptom | Cause | Fix |
|---|---|---|---|
| 1 | Blank/grey app in Hub, assets 404 or Core JSON | Vite `base: "/"` → `/assets/…` hits Core's `/assets` API | `base: ""` |
| 2 | Blank app, works locally | CDN script/font/CSS blocked by CSP | bundle everything |
| 3 | White screen outside Hub | `qortalRequest` undefined → ReferenceError unmounts React | stub when `typeof qortalRequest !== 'function'` |
| 4 | "Not in Hub" detected inside Hub | checked `window.qortalRequest` (it's a global `const`) | bare `typeof qortalRequest` |
| 5 | App blank when opened from a link, or for a non-ASCII app name | link spelled `%2B`, or browser-encoded `location.pathname` vs raw `_qdnBase`; router basename mismatch | write `+` literally; tolerant `routerBase()` (9.3) |
| 6 | App reloads (loses state) on Hub Back | didn't answer `NAVIGATE_TO_PATH` | reply `NAVIGATION_SUCCESS` at once |
| 7 | Back button moves another tab / does nothing | `history.back()` / `navigate(-1)` | own back stack + parent fallback |
| 8 | Deep link 404 on a website | WEBSITE has no SPA fallback | hash routes or real files |
| 9 | Dark Hub, app turns light after reload | `_qdnTheme` defaults to `"light"` without `?theme=` | remember the mode; precedence URL → stored |
| 10 | App ignores Hub's theme switch | no `THEME_CHANGED` listener | listen and re-theme live |
| 11 | Flash of wrong background | theme applied by framework after load | inline boot script in `<head>` |
| 12 | Bottom bar hidden behind GO's bar | `100vh` | `--app-height` from `innerHeight` |
| 13 | Sticky header scrolls away | `overflow-x: hidden` on html/body | `overflow-x: clip` on `#root` |
| 14 | Page jumps 16 px when a menu opens | `overflow-x: clip` on `html` + scroll lock | clip `#root` only |
| 15 | Feed shows one post per user | default `mode=LATEST` | `mode=ALL` |
| 16 | Search slow / huge response | no `limit` or `limit=0` (= all) | explicit `limit`, paging |
| 17 | "Exists" check finds the wrong item | contains matching | `prefix=true`, `exactmatchnames`, compare exactly |
| 18 | Other apps' items appear in your list | short/generic prefix + contains | distinctive prefix, `prefix=true`, validate body |
| 19 | Names with `+`/space/`&` return nothing | q-apps.js doesn't encode | `fetch` with `URLSearchParams`/`encodeURIComponent` |
| 20 | Names decoded wrong | `URLSearchParams` turns `+` into space | `decodeURIComponent` |
| 21 | Edited item doesn't move to top | ordering by `created` (first publish) | new identifier per event, or sort client-side by `updated` |
| 22 | Item missing right after publishing | propagation / metadata not fetched yet | optimistic insert, merge later; fresh search |
| 23 | Endless spinner on some items | data not on node, sync GET timed out | `async=true`/status, "Fetching from peers", backoff, final message |
| 24 | File stuck at "Preparing 100 %" | status stays `DOWNLOADED` | status with `build=true` (long timeout) |
| 25 | Avatars never appear for some users | first 404 cached as "no avatar" | status check; NOT_PUBLISHED only is final; retry others |
| 26 | Burst of 100 avatar requests | UI-kit Avatar loads on mount | lazy `<img>` via IntersectionObserver |
| 27 | `<img src="qortal://…">` broken | q-apps.js rewrites only the first, initial image | build `/arbitrary/...` URLs |
| 28 | Garbled emoji/accents or `btoa` exception | `btoa(JSON.stringify())` | UTF-8 base64 helpers |
| 29 | JSON doc unreadable via FETCH | top-level `"error"` key → treated as failure | never use `error` as a top-level key |
| 30 | Publish rejected `DATA_NOT_ENCRYPTED` / `DATA_ENCRYPTED`, or a "private" document readable by all | private service with plain data / public with encrypted; Core doesn't check FILE/IMAGE/DOCUMENT(_PRIVATE) | match service to encryption; always encrypt before a `_PRIVATE` publish |
| 31 | `JSON` publish rejected | > 25 KB or invalid JSON (e.g. `"D"` tombstone) | DOCUMENT for bigger; `{deleted:true}` |
| 32 | Tag missing from metadata | tag > 20 chars dropped | ≤ 5 tags of ≤ 20 chars |
| 33 | Big multi-publish partly fails | > 25 unconfirmed per account | batches ≤ 20, wait for room |
| 34 | Large publish "times out" yet completes | q-apps.js flat 1 h vs Hub n × 30 min | `qortalRequestWithTimeout(req, n*30min)` |
| 35 | User charged twice | blind Retry after a timeout; late Accept applied | verify on QDN/chain first |
| 36 | Red error when user pressed Decline | decline treated as failure; English-only check; Hub reports a declined sign-in as "unable to fetch user account" | `isDecline()` multilingual, quiet; any non-timeout `GET_USER_ACCOUNT` failure = not signed in |
| 37 | `[object Object]` toast | error shapes vary | `errorMessage()` |
| 38 | Data wiped | app published automatically after a failed read | never auto-publish; read fresh before RMW |
| 39 | Other device's changes lost | rebuilt index from stale copy | fresh read right before rebuilding |
| 40 | Deadline cheated | trusted publisher-signed `created` | block time of the confirming block |
| 41 | Anyone can post "as" someone | author taken from body | author = search row's `name` |
| 42 | One bad document crashes the page | no validation / error boundary | validate + normalise on read; boundaries |
| 43 | Stored XSS | regex over sanitised HTML / `innerHTML` with user data | DOMPurify + DOM post-processing; `textContent` |
| 44 | Invisible text in light theme | inline white colours from pasted HTML | strip colour styles on display |
| 45 | Web link does nothing | q-apps.js blocks http(s); no popups | copy to clipboard + toast |
| 46 | Frame navigates away on a chat link | `qortal://use-group/...` crashes q-apps.js link parsing | intercept, call `JOIN_GROUP` |
| 47 | Follow/block feature errors in GO | lists refused on public node; refusal text is translated | `IS_USING_PUBLIC_NODE` at start-up; local + QDN fallback (11.1), hide quietly |
| 48 | PDF preview blank | no PDF plugin in Hub/GO | `SHOW_PDF_READER` or bundled pdf.js |
| 49 | Save fails "mime type" | blob without type | pass `mimeType` |
| 50 | Saved attachment unreadable | location save of encrypted data | decrypt then save blob |
| 51 | App broken after update in open tab | old chunk → `index.html` 200 | `lazyWithRetry` + reload once |
| 52 | Settings leak between apps / overwritten | unprefixed `localStorage` keys (shared origin) | `myapp-` prefix |
| 53 | Poll created with one option | Hub uses `pollOptions[0]` split on `", "` | `['A, B, C']` |
| 54 | `GET_BALANCE` with `assetId` → "invalid address" | q-apps.js URL bug | `/assets/balances?address=&assetid=` |
| 55 | Wrong group admins | paged `isAdmin` flag wrong after page 1 | `?onlyAdmins=true` |
| 56 | Transaction search HTML 500 | missing `confirmationStatus` | always send it |
| 57 | Request never answered | action unknown to this Hub (e.g. GO-only `SET_FULL_SCREEN` in desktop Hub) | `SHOW_ACTIONS`, short timeouts |
| 58 | Layout flips while typing in GO | keyboard shrinks frame, "height < 500" rule | focus-aware keyboard detection |
| 59 | Toast/FAB covers Submit | overlaps | clearance, toasts above bars |
| 60 | Slow long lists | tooltip/media query per row | CSS breakpoints, lazy tooltips, virtualisation |
| 61 | Dev Mode loads without `qortalRequest` | proxy injects only at `/` and `*.html` | reload at `/` |
| 62 | Dev Mode `vite preview` 404s on assets | proxy collides with Core `/assets` API | `vite dev` or `--assetsDir static` (test only) |
| 63 | Copied links in Dev Mode lack the app name | `_qdnName=""` | fallback to published name |
| 64 | Dialog storm / GO auto-declines | many dialog requests in parallel | one dialog request at a time |
| 65 | App published under non-default identifier misses chunks | Core adds `?identifier=` only to static tags | publish apps under `default` |
| 66 | In-app text search finds almost nothing | `prefix=true` with `query`/`title` makes every term starts-with | identifier as contains-match + `query`, filter in code (6.3) |
| 67 | Emoji names flagged as impostors | invisible-character check includes U+FE0F / U+200D | allow them next to an emoji (5.2) |
| 68 | Phone runs out of memory / huge publish | raw camera photo published | downscale + thumbnail first (10.3) |
| 69 | Wrong item opened for names with `%` | `useParams()` values decoded a second time | use router params as they are |
| 70 | Favourites gone after switching node | `localStorage` belongs to the node's origin | local copy + explicit QDN backup (11.1) |

---

## 18. Templates

### 18A. A complete single-file Q-App ("Notice Board")

One `index.html`, no build. Anyone signed in with a name can post a short public notice (service `JSON`, identifier `noticeboard_note_<id>`); everyone sees the newest first, with avatars, paging, "Fetching from peers…", optimistic posting, decline/timeout handling, verify-before-retry, delete via tombstone, live theme and Hub Back support.

Change `PUBLISHED_NAME` to the name you'll publish under, and `APP`/`PREFIX` to your own slug. Zip it alone (`zip release.zip index.html`) and publish as `APP`.

```html
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Notice Board</title>
<script>
/* Theme before first paint: ?theme= (remember it) → remembered → OS → dark. */
(function () {
  var KEY = 'noticeboard-hub-mode';
  var q = new URLSearchParams(location.search).get('theme');
  var mode = null;
  try {
    if (q === 'light' || q === 'dark') { mode = q; localStorage.setItem(KEY, q); }
    else mode = localStorage.getItem(KEY);
  } catch (e) {}
  if (mode !== 'light' && mode !== 'dark')
    mode = window.matchMedia && matchMedia('(prefers-color-scheme: light)').matches ? 'light' : 'dark';
  var root = document.documentElement;
  root.setAttribute('data-theme', mode);
  root.style.colorScheme = mode;
  root.style.background = mode === 'light' ? '#f6f7f9' : '#14161a';
})();
</script>
<style>
:root[data-theme="dark"]  { --bg:#14161a; --surface:#1e2127; --text:#e8eaed; --muted:#a3a9b3; --border:#2e333b; --accent:#5aa9ff; --accent-text:#0b1220; --danger:#ff8a80; }
:root[data-theme="light"] { --bg:#f6f7f9; --surface:#ffffff; --text:#14161a; --muted:#545b66; --border:#d5dae0; --accent:#0b62d6; --accent-text:#ffffff; --danger:#b3261e; }
:root { --app-height: 100dvh; }
* { box-sizing: border-box; }
html, body { margin: 0; background: var(--bg); color: var(--text);
  font: 16px/1.45 system-ui, -apple-system, "Segoe UI", Roboto, sans-serif; }
#root { min-height: var(--app-height); overflow-x: clip; display: flex; flex-direction: column; }
header { position: sticky; top: 0; z-index: 2; display: flex; align-items: center; gap: 8px;
  padding: 6px 16px; min-height: 56px; background: var(--surface); border-bottom: 1px solid var(--border); }
header h1 { font-size: 18px; margin: 0; flex: 1; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
main { width: 100%; max-width: 720px; margin: 0 auto; padding: 16px;
  padding-bottom: calc(32px + env(safe-area-inset-bottom, 0px)); flex: 1; }
button { font: inherit; min-height: 44px; min-width: 44px; padding: 0 16px; border-radius: 10px;
  border: 1px solid var(--border); background: var(--surface); color: var(--text); cursor: pointer; touch-action: manipulation; }
button.primary { background: var(--accent); color: var(--accent-text); border-color: transparent; }
button.danger { color: var(--danger); }
button:disabled { opacity: .55; cursor: default; }
button:focus-visible, textarea:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; }
textarea { width: 100%; min-height: 96px; font: inherit; font-size: 16px; padding: 12px; border-radius: 10px;
  border: 1px solid var(--border); background: var(--surface); color: var(--text); resize: vertical; }
.card { background: var(--surface); border: 1px solid var(--border); border-radius: 12px; padding: 12px 14px; margin: 10px 0; }
.row { display: flex; align-items: center; gap: 10px; flex-wrap: wrap; }
.spread { justify-content: space-between; }
.grow { flex: 1; min-width: 0; }
.name { font-weight: 600; overflow-wrap: anywhere; }
.muted { color: var(--muted); font-size: 14px; }
.avatar { width: 36px; height: 36px; border-radius: 50%; overflow: hidden; display: inline-grid; place-items: center;
  background: var(--border); flex: none; font-weight: 600; }
.avatar img { width: 100%; height: 100%; object-fit: cover; }
.text { white-space: pre-wrap; overflow-wrap: anywhere; margin: 8px 0 0; }
.notice { padding: 12px 14px; border-radius: 10px; border: 1px dashed var(--border); color: var(--muted); }
.toast { position: fixed; left: 50%; transform: translateX(-50%); bottom: calc(16px + env(safe-area-inset-bottom, 0px));
  background: var(--text); color: var(--bg); padding: 10px 16px; border-radius: 10px; max-width: calc(100% - 32px); z-index: 10; }
.skeleton { height: 76px; border-radius: 12px; background: var(--border); opacity: .5; margin: 10px 0; }
@media (prefers-reduced-motion: no-preference) {
  .skeleton { animation: pulse 1.2s ease-in-out infinite; }
  @keyframes pulse { 50% { opacity: .25; } }
}
</style>
</head>
<body>
<div id="root">
  <header>
    <h1>Notice Board</h1>
    <button id="refresh" aria-label="Refresh" title="Refresh">&#x21bb;</button>
    <span id="who" class="muted"></span>
    <button id="signin">Sign in</button>
  </header>
  <main>
    <p id="hostNote" class="notice" hidden>Open this app in Qortal Hub or Qortal GO to sign in and post.</p>
    <section id="composer" hidden>
      <label for="text" class="muted">Your notice (up to 500 characters)</label>
      <textarea id="text" maxlength="500" enterkeyhint="send"></textarea>
      <div class="row spread" style="margin-top:8px">
        <span class="muted grow">Posting costs 0.01 QORT. Hub will ask you to confirm.</span>
        <button class="primary" id="post">Post</button>
      </div>
    </section>
    <p id="noName" class="notice" hidden>To post, register a name in Qortal Hub first (1.25 QORT).</p>
    <div id="status" class="notice" hidden></div>
    <div id="list"></div>
    <div class="row" style="justify-content:center;margin-top:12px"><button id="more" hidden>Load more</button></div>
  </main>
</div>
<div id="toast" class="toast" role="status" aria-live="polite" hidden></div>
<script>
'use strict';
/* ---------------- 0. App identity: keep in one place ---------------- */
var APP = 'noticeboard';                 // prefix for localStorage keys (all Q-Apps share one origin)
var PUBLISHED_NAME = 'Notice Board';     // the registered name you publish this app under
var SERVICE = 'JSON';                    // public, ≤ 25 KB, must be valid JSON
var PREFIX = 'noticeboard_note_';        // identifier prefix for notes (lowercase, [a-z0-9_])
var PAGE = 20, MAX_TEXT = 500;
var RETRY_MS = [2000, 4000, 8000, 16000];
var HUB_DIALOG_GRACE_MS = 35000;

/* ---------------- 1. Host detection ---------------- */
// qortalRequest is a global const from q-apps.js, NOT window.qortalRequest.
var IN_HOST = typeof qortalRequest === 'function';
function q(req, ms) {
  if (!IN_HOST) return Promise.reject(new Error('Open this app in Qortal Hub or Qortal GO.'));
  return ms && typeof qortalRequestWithTimeout === 'function' ? qortalRequestWithTimeout(req, ms) : qortalRequest(req);
}

/* ---------------- 2. Errors ---------------- */
function errorMessage(e, fallback) {
  fallback = fallback === undefined ? 'Something went wrong' : fallback;
  if (typeof e === 'string') return e || fallback;
  if (e && typeof e.message === 'string' && e.message) return e.message;
  if (e && typeof e.error === 'string' && e.error) return e.error;
  return fallback;
}
var DECLINE = /declin|cancel|rechaz|refus|ablehn|abgelehnt|отклон|拒否|拒绝|kieltäyt|keeldus|rifiut|recus|رفض/i;
function isDecline(e) {
  if (e && typeof e === 'object' && e.error && typeof e.error === 'object' && e.error.cancelled) return true;
  return DECLINE.test(errorMessage(e, ''));
}
function isTimeout(e) { return /timed out/i.test(errorMessage(e, '')); }
function sleep(ms) { return new Promise(function (r) { setTimeout(r, ms); }); }
function whenVisible() {
  if (document.visibilityState !== 'hidden') return Promise.resolve();
  return new Promise(function (resolve) {
    document.addEventListener('visibilitychange', function h() {
      if (document.visibilityState !== 'hidden') { document.removeEventListener('visibilitychange', h); resolve(); }
    });
  });
}

/* ---------------- 3. Small UI helpers ---------------- */
function $(id) { return document.getElementById(id); }
var toastTimer;
function toast(msg) {
  var t = $('toast'); t.textContent = msg; t.hidden = false;
  clearTimeout(toastTimer); toastTimer = setTimeout(function () { t.hidden = true; }, 4000);
}
function el(tag, cls, text) {
  var n = document.createElement(tag);
  if (cls) n.className = cls;
  if (text != null) n.textContent = text;            // textContent only: never innerHTML with QDN data
  return n;
}
var dateFmt = new Intl.DateTimeFormat(undefined, { dateStyle: 'medium', timeStyle: 'short' });
function fmtDate(ms) { return typeof ms === 'number' && ms > 0 && ms < 8.64e15 ? dateFmt.format(ms) : ''; }

/* ---------------- 4. Frame height, Hub messages ---------------- */
function applyHeight() {
  var framed = true; try { framed = window.parent !== window; } catch (e) {}
  document.documentElement.style.setProperty('--app-height',
    framed && innerHeight > 0 ? Math.round(innerHeight) + 'px' : '100dvh');
}
applyHeight(); addEventListener('resize', applyHeight);

function fromHost(e) {
  var w = window;
  while (w !== window.top) { w = w.parent; if (e.source === w) return true; }
  return false;
}
addEventListener('message', function (e) {
  if (!fromHost(e)) return;
  var d = e.data || {};
  if (d.action === 'THEME_CHANGED' && (d.theme === 'light' || d.theme === 'dark')) {
    try { localStorage.setItem(APP + '-hub-mode', d.theme); } catch (err) {}
    window._qdnTheme = d.theme;
    document.documentElement.setAttribute('data-theme', d.theme);
    document.documentElement.style.colorScheme = d.theme;
    document.documentElement.style.background = '';
  }
  if (d.action === 'NAVIGATE_TO_PATH' && typeof d.path === 'string') {
    // Single page: nothing to route, but answer at once or Hub reloads the frame.
    window.parent.postMessage({ action: 'NAVIGATION_SUCCESS', path: d.path }, '*');
  }
});

/* ---------------- 5. Node reads (encoded, in-flight merged) ---------------- */
var inflight = {};
function getJson(url) {
  if (inflight[url]) return inflight[url];
  var p = fetch(url).then(function (r) {
    return r.text().then(function (t) {
      var body = t; try { body = JSON.parse(t); } catch (e) {}
      if (!r.ok || (body && typeof body === 'object' && 'error' in body)) {
        var err = new Error(errorMessage(body, 'The node answered ' + r.status)); err.status = r.status; throw err;
      }
      return body;
    });
  });
  inflight[url] = p;
  var forget = function () { delete inflight[url]; };
  p.then(forget, forget);
  return p;
}
function searchUrl(params) {
  var p = new URLSearchParams();
  p.set('service', SERVICE);
  p.set('mode', 'ALL');                         // default LATEST = one row per (name, service)!
  p.set('limit', String(params.limit || PAGE)); // never omit: missing/0 = everything
  p.set('offset', String(params.offset || 0));
  p.set('reverse', 'true');                     // newest first
  p.set('excludeblocked', 'true');
  if (params.identifier) { p.set('identifier', params.identifier); p.set('prefix', 'true'); }
  if (params.name) { p.append('name', params.name); p.set('exactmatchnames', 'true'); }
  return '/arbitrary/resources/search?' + p.toString();
}
function bodyUrl(row) {
  return '/arbitrary/' + SERVICE + '/' + encodeURIComponent(row.name) + '/' +
    encodeURIComponent(row.identifier) + '?async=true';
}
// Exact lookup: search matches are "contains", so compare in code. `since` filters older versions.
function findExact(name, identifier, since) {
  return getJson(searchUrl({ identifier: identifier, name: name, limit: 20 }) + '&_=' + Date.now()).then(function (rows) {
    return (Array.isArray(rows) ? rows : []).some(function (r) {
      return (r.identifier || 'default') === identifier && String(r.name).toLowerCase() === name.toLowerCase() &&
        (!since || (r.updated || r.created || 0) >= since - 120000);
    });
  });
}
function limiter(n) {
  var active = 0, queue = [];
  function next() {
    if (active >= n || !queue.length) return;
    active++;
    var job = queue.shift();
    job.fn().then(job.resolve, job.reject).then(function () { active--; next(); });
  }
  return function (fn) { return new Promise(function (resolve, reject) { queue.push({ fn: fn, resolve: resolve, reject: reject }); next(); }); };
}
var limit5 = limiter(5);

/* ---------------- 6. Data shape: validate everything ---------------- */
function parseNote(row, body) {
  if (!body || typeof body !== 'object' || Array.isArray(body)) return null;
  if (body.deleted === true) return { deleted: true };
  if (body.v !== 1 || typeof body.text !== 'string') return null;
  var text = body.text.slice(0, MAX_TEXT);
  if (!text.trim()) return null;
  // Author and dates come from the search ROW, never from the body.
  return { text: text, name: row.name, identifier: row.identifier, created: row.created };
}

/* ---------------- 7. UTF-8 safe base64 ---------------- */
function objectToBase64(obj) {
  var bytes = new TextEncoder().encode(JSON.stringify(obj)), bin = '';
  for (var i = 0; i < bytes.length; i += 0x8000) bin += String.fromCharCode.apply(null, bytes.subarray(i, i + 0x8000));
  return btoa(bin);
}

/* ---------------- 8. Account ---------------- */
var state = { account: null, name: '', ownNames: {}, offset: 0, done: false, loading: false, seen: {} };
function renderAccount() {
  var signedIn = !!state.account;
  $('signin').hidden = signedIn || !IN_HOST;
  $('who').textContent = signedIn ? (state.name || state.account.address.slice(0, 6) + '…') : '';
  $('composer').hidden = !(signedIn && state.name);
  $('noName').hidden = !(signedIn && !state.name);
  $('hostNote').hidden = IN_HOST;
}
function signIn(auto) {
  $('signin').disabled = true;
  return q({ action: 'GET_USER_ACCOUNT' }).then(function (acct) {
    state.account = acct;
    try { localStorage.setItem(APP + '-signed-in', '1'); } catch (e) {}
    return getJson('/names/address/' + encodeURIComponent(acct.address) + '?limit=100').catch(function () { return []; })
      .then(function (names) {
        state.ownNames = {};
        (Array.isArray(names) ? names : []).forEach(function (n) { if (n && n.name) state.ownNames[n.name.toLowerCase()] = true; });
        return q({ action: 'GET_PRIMARY_NAME', address: acct.address }).catch(function () {
          return getJson('/names/primary/' + encodeURIComponent(acct.address)).then(function (r) { return (r && r.name) || ''; }, function () { return ''; });
        });
      })
      .then(function (primary) {
        state.name = typeof primary === 'string' ? primary : '';
        renderAccount();
        Array.prototype.forEach.call(document.querySelectorAll('#list article'), function (c) {
          if (c._loaded) addOwnActions(c, c._row);            // own notes get their Delete button
        });
      });
  }).catch(function (e) {
    if (isTimeout(e)) {
      // Hub's dialog may still be up; a late Accept makes this retry answer without a dialog.
      setTimeout(function () { if (!state.account) signIn(true); }, HUB_DIALOG_GRACE_MS);
    } else if (!auto && !IN_HOST) {
      // In Hub every other GET_USER_ACCOUNT failure, a decline included, arrives as a translated
      // "unable to fetch user account": treat it as a quiet "not signed in". Outside Hub, say why.
      toast(errorMessage(e));
    }
    renderAccount();
  }).then(function () { $('signin').disabled = false; });
}

/* ---------------- 9. List rendering ---------------- */
var io = 'IntersectionObserver' in window ? new IntersectionObserver(function (entries) {
  entries.forEach(function (en) { if (en.isIntersecting) { io.unobserve(en.target); en.target._load(); } });
}, { rootMargin: '400px 0px' }) : null;
function whenNear(node, fn) { node._load = fn; if (io) io.observe(node); else fn(); }

var avatarJobs = {};
function avatarReady(name) {       // one shared job per name; resolves to a URL or null
  if (avatarJobs[name]) return avatarJobs[name];
  var url = '/arbitrary/THUMBNAIL/' + encodeURIComponent(name) + '/qortal_avatar?async=true';
  var statusUrl = '/arbitrary/resource/status/THUMBNAIL/' + encodeURIComponent(name) + '/qortal_avatar';
  var attempt = 0;
  avatarJobs[name] = new Promise(function (resolve) {
    (function tryOnce() {
      var img = new Image();
      img.onload = function () { resolve(url); };
      img.onerror = function () {        // 404 also means "still fetching": ask the status once
        getJson(statusUrl).then(function (s) { return s && s.status === 'NOT_PUBLISHED'; }, function () { return false; })
          .then(function (none) {
            if (none) return resolve(null);                                       // really no avatar
            if (attempt >= 4) { delete avatarJobs[name]; return resolve(null); } // give up for now, not forever
            setTimeout(function () { whenVisible().then(tryOnce); }, [2000, 4000, 8000, 15000][attempt++]);
          });
      };
      img.src = url;
    })();
  });
  return avatarJobs[name];
}
function avatar(name) {
  var span = el('span', 'avatar', (Array.from(name)[0] || '?').toUpperCase());
  span.setAttribute('aria-hidden', 'true');
  whenNear(span, function () {
    avatarReady(name).then(function (url) {
      if (!url) return;
      var img = new Image(); img.alt = ''; img.src = url;
      span.textContent = ''; span.appendChild(img);
    });
  });
  return span;
}

function makeCard(row, localText) {
  var card = el('article', 'card');
  var head = el('div', 'row');
  var meta = el('div', 'grow');
  meta.appendChild(el('div', 'name', row.name));
  var when = el('div', 'muted', localText ? 'Publishing…' : fmtDate(row.created));
  meta.appendChild(when);
  head.appendChild(avatar(row.name));
  head.appendChild(meta);
  card.appendChild(head);
  var text = el('p', 'text muted', localText || 'Loading…');
  if (localText) text.classList.remove('muted');
  card.appendChild(text);
  card._when = when; card._text = text; card._row = row;
  if (!localText) whenNear(card, function () { loadBody(card, row); });
  return card;
}
function loadBody(card, row, attempt) {
  attempt = attempt || 0;
  return limit5(function () { return getJson(bodyUrl(row)); }).then(function (body) {
    var note = parseNote(row, body);
    if (!note || note.deleted) { card.remove(); return; }   // invalid or deleted: drop quietly
    card._text.classList.remove('muted');
    card._text.textContent = note.text;
    card._loaded = true;
    addOwnActions(card, row);
  }, function () {
    if (attempt < RETRY_MS.length) {
      card._text.textContent = 'Fetching from peers…';
      return sleep(RETRY_MS[attempt]).then(whenVisible).then(function () { return loadBody(card, row, attempt + 1); });
    }
    card._text.textContent = 'Not available on your node right now.';
    var retry = el('button', null, 'Retry');
    retry.onclick = function () { retry.remove(); card._text.textContent = 'Loading…'; loadBody(card, row, 0); };
    card.appendChild(retry);
  });
}
function addOwnActions(card, row) {
  if (!state.account || !state.ownNames[row.name.toLowerCase()] || card._hasActions) return;
  card._hasActions = true;
  var del = el('button', 'danger', 'Delete');
  var armed = false, disarm;
  del.onclick = function () {
    if (!armed) {       // two taps instead of confirm(): clear and touch friendly
      armed = true; del.textContent = 'Tap again to delete (0.01 QORT)';
      disarm = setTimeout(function () { armed = false; del.textContent = 'Delete'; }, 4000);
      return;
    }
    clearTimeout(disarm); del.disabled = true; del.textContent = 'Deleting…';
    var started = Date.now();
    publishVerified(row.name, row.identifier, { v: 1, deleted: true }, started).then(function (r) {
      if (r === 'done') { card.remove(); toast('Deleted. Other nodes will catch up within a few minutes.'); }
      else if (r === 'declined') { armed = false; del.disabled = false; del.textContent = 'Delete'; }
      else if (r === 'unknown') { del.textContent = 'Not confirmed yet: refresh in a minute'; }
      else { armed = false; del.disabled = false; del.textContent = 'Delete'; toast(r.failed); }
    });
  };
  card.appendChild(del);
}

function setStatus(msg, withRetry) {
  var s = $('status'); s.textContent = msg || ''; s.hidden = !msg;
  if (withRetry) { var b = el('button', null, 'Retry'); b.style.marginLeft = '8px'; b.onclick = refresh; s.appendChild(b); }
}
function loadPage() {
  if (state.loading || state.done) return Promise.resolve();
  state.loading = true; $('more').disabled = true;
  var list = $('list');
  if (!state.offset) { list.textContent = ''; for (var i = 0; i < 3; i++) list.appendChild(el('div', 'skeleton')); }
  return getJson(searchUrl({ identifier: PREFIX, offset: state.offset })).then(function (rows) {
    rows = Array.isArray(rows) ? rows : [];
    if (!state.offset) list.textContent = '';
    state.offset += rows.length;
    state.done = rows.length < PAGE;
    rows.forEach(function (r) {
      r.identifier = r.identifier || 'default';
      if (r.identifier.indexOf(PREFIX) !== 0) return;            // prefix search is case-insensitive: re-check
      var key = r.name + '\n' + r.identifier;
      if (state.seen[key]) return;
      state.seen[key] = true;
      list.appendChild(makeCard(r));
    });
    setStatus(list.children.length ? '' : 'No notices yet. Be the first to post one.');
  }, function (e) {
    if (!state.offset) list.textContent = '';
    setStatus(IN_HOST ? 'Could not load notices (' + errorMessage(e) + ').' : 'Open this app in Qortal Hub or Qortal GO.', IN_HOST);
  }).then(function () {
    state.loading = false; $('more').disabled = false; $('more').hidden = state.done;
  });
}
function refresh() { state.offset = 0; state.done = false; state.seen = {}; return loadPage(); }

/* ---------------- 10. Publishing: fee shown, decline quiet, timeout verified ---------------- */
function publishVerified(name, identifier, doc, started) {
  return q({
    action: 'PUBLISH_QDN_RESOURCE', name: name, service: SERVICE, identifier: identifier,
    data64: objectToBase64(doc), filename: 'note.json', title: 'Notice'
  }).then(function () { return 'done'; }, function (e) {
    if (isDecline(e)) return 'declined';
    if (isTimeout(e)) {          // unknown: Hub may still publish. Look before anyone retries.
      return sleep(HUB_DIALOG_GRACE_MS).then(function () { return findExact(name, identifier, started); })
        .then(function (found) { return found ? 'done' : 'unknown'; }, function () { return 'unknown'; });
    }
    return { failed: errorMessage(e) };
  });
}
function post() {
  var text = $('text').value.trim().slice(0, MAX_TEXT);
  if (!text) return;
  if (!state.account) { signIn(false); return; }
  if (!state.name) { toast('Register a name in Qortal Hub to post.'); return; }
  var identifier = PREFIX + Date.now().toString(36) + Math.random().toString(36).slice(2, 8);
  var started = Date.now();
  var row = { name: state.name, identifier: identifier, created: started };
  var card = makeCard(row, text);                       // optimistic
  $('list').insertBefore(card, $('list').firstChild);
  setStatus('');
  $('post').disabled = true;
  publishVerified(state.name, identifier, { v: 1, text: text }, started).then(function (r) {
    $('post').disabled = false;
    if (r === 'done') {
      $('text').value = '';
      card._when.textContent = 'Posted. Others will see it within a minute or two.';
      state.seen[row.name + '\n' + identifier] = true;
      addOwnActions(card, row);
    } else if (r === 'declined') {
      card.remove();
    } else if (r === 'unknown') {
      card._when.textContent = 'Not confirmed yet.';
      var check = el('button', null, 'Check again');
      check.onclick = function () {
        check.disabled = true;
        findExact(state.name, identifier, started).then(function (found) {
          check.disabled = false;
          if (found) { check.remove(); card._when.textContent = 'Posted.'; $('text').value = ''; }
          else toast('Still not found. Wait a minute before posting again, so you are not charged twice.');
        }, function () { check.disabled = false; });
      };
      card.appendChild(check);
    } else {
      card.remove(); toast(r.failed);
    }
  });
}

/* ---------------- 11. Wire up ---------------- */
$('signin').onclick = function () { signIn(false); };
$('post').onclick = post;
$('more').onclick = loadPage;
$('refresh').onclick = refresh;
renderAccount();
refresh();
try { if (IN_HOST && localStorage.getItem(APP + '-signed-in') === '1') signIn(true); } catch (e) {}
</script>
</body>
</html>
```

Data contract of this template (what an LLM must state for any app):

| Item | Value |
|---|---|
| Service | `JSON` (public) |
| Identifier | `noticeboard_note_<base36 time><6 random>` (~32 chars) |
| Body | `{"v":1,"text":"…≤500 chars"}`; tombstone `{"v":1,"deleted":true}` |
| Author/date | from the search row (`name`, `created`) |
| Cost | 0.01 QORT per post or delete |
| Reads | `search?service=JSON&identifier=noticeboard_note_&prefix=true&mode=ALL&limit=20&offset=n&reverse=true`, bodies with `?async=true`, ≤ 5 at a time, only near the viewport |

### 18B. React + Vite + TypeScript starter

A minimal but complete skeleton: boot, host stub, router basename, Hub messages, safe Back, search/fetch/publish helpers, links, an account store, a lazy avatar, a feed screen with deep links, mocks and a read-only dev shim. Rename `myapp`/`My App` everywhere (they live in `src/config.ts` and the boot script).

Use current stable versions; the ranges below are a known-good floor. This starter was type-checked, tested and built as printed here (also on Vite 8 / Vitest 5 / React 19 / React Router 6). If a Vite release drops `manualChunks`, move the vendor split to `advancedChunks.groups`.

#### `package.json`

```json
{
  "name": "my-app",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc --noEmit && vite build",
    "test": "vitest run",
    "zip": "bash scripts/zip.sh"
  },
  "dependencies": {
    "dompurify": "^3.4.17",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "react-router-dom": "^7.0.0"
  },
  "devDependencies": {
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "@vitejs/plugin-react": "^4.3.0",
    "jsdom": "^25.0.0",
    "typescript": "^5.6.0",
    "vite": "^6.0.0",
    "vitest": "^3.0.0"
  }
}
```

#### `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2020", "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext", "moduleResolution": "Bundler", "jsx": "react-jsx",
    "strict": true, "noEmit": true, "skipLibCheck": true, "isolatedModules": true,
    "types": ["vite/client", "vitest/globals"]
  },
  "include": ["src"]
}
```

#### `vite.config.ts`

```ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

// Plain-browser dev only: forward Core API paths to a node, GET/HEAD only (see devShim.ts).
// Inside Hub Dev Mode, Core's proxy answers these paths itself.
const NODE = process.env.QORTAL_NODE ?? 'http://127.0.0.1:12391';
const CORE_PATHS = ['/arbitrary', '/names', '/addresses', '/transactions', '/groups', '/blocks',
  '/admin/info', '/crosschain', '/polls', '/assets/info', '/assets/balances'];
const readOnly = {
  target: NODE, changeOrigin: true,
  // Anything but GET/HEAD gets a 404 here and never reaches the node.
  bypass: (req: { method?: string }) => (req.method === 'GET' || req.method === 'HEAD' ? undefined : false),
};

export default defineConfig({
  plugins: [react()],
  base: '',                                // relative ./assets/… ; never '/'
  build: {
    target: 'es2020',                      // old Android WebViews in GO
    sourcemap: false,
    rollupOptions: {
      output: {
        manualChunks(id) {
          if (/node_modules[\\/](react|react-dom|react-router|react-router-dom|scheduler)[\\/]/.test(id)) return 'react-vendor';
        },
      },
    },
  },
  server: {
    host: '127.0.0.1', port: 5173, strictPort: true,
    proxy: Object.fromEntries(CORE_PATHS.map(p => [p, readOnly])),
  },
  test: { environment: 'jsdom', globals: true, setupFiles: ['./src/test/setup.ts'], include: ['src/**/*.test.{ts,tsx}'] },
});
```

#### `index.html`

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
    <title>My App</title>
    <script>
      (function () {
        var KEY = 'myapp-hub-mode';
        var q = new URLSearchParams(location.search).get('theme'), mode = null;
        try {
          if (q === 'light' || q === 'dark') { mode = q; localStorage.setItem(KEY, q); }
          else mode = localStorage.getItem(KEY);
        } catch (e) {}
        if (mode !== 'light' && mode !== 'dark')
          mode = window.matchMedia && matchMedia('(prefers-color-scheme: light)').matches ? 'light' : 'dark';
        var r = document.documentElement;
        r.setAttribute('data-theme', mode); r.style.colorScheme = mode;
        r.style.background = mode === 'light' ? '#f6f7f9' : '#14161a';
      })();
    </script>
    <link rel="icon" href="./favicon.ico" />
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

#### `src/index.css`

```css
:root[data-theme="dark"]  { --bg:#14161a; --surface:#1e2127; --text:#e8eaed; --muted:#a3a9b3; --border:#2e333b; --accent:#5aa9ff; --accent-text:#0b1220; --danger:#ff8a80; }
:root[data-theme="light"] { --bg:#f6f7f9; --surface:#ffffff; --text:#14161a; --muted:#545b66; --border:#d5dae0; --accent:#0b62d6; --accent-text:#ffffff; --danger:#b3261e; }
:root { --app-height: 100dvh; }
* { box-sizing: border-box; }
html, body { margin: 0; background: var(--bg); color: var(--text); font: 16px/1.45 system-ui, -apple-system, "Segoe UI", Roboto, sans-serif; }
#root { min-height: var(--app-height); overflow-x: clip; }
button { min-height: 44px; min-width: 44px; touch-action: manipulation; font: inherit; }
```

#### `src/config.ts`

```ts
export const APP_SLUG = 'myapp';                 // storage-key prefix; all Q-Apps share one origin
export const PUBLISHED_NAME = 'My App';          // registered name the app is published under
export const VERSION = '1.0.0';
export const ID = {                               // identifier prefixes (lowercase, distinctive)
  post: 'myapp_post_',
} as const;
export const storageKey = (k: string) => `${APP_SLUG}-${k}`;
```

#### `src/qortal/globals.d.ts`

```ts
// q-apps.js declares these as global consts (not window properties).
declare const qortalRequest: (req: Record<string, unknown>) => Promise<any>;
declare const qortalRequestWithTimeout: (req: Record<string, unknown>, ms: number) => Promise<any>;

interface Window {
  _qdnContext?: string; _qdnTheme?: string; _qdnLang?: string; _qdnService?: string;
  _qdnName?: string; _qdnIdentifier?: string; _qdnPath?: string; _qdnBase?: string; _qdnBaseWithPath?: string;
  __QORTAL_STUB__?: boolean; __QORTAL_DEV_SHIM__?: boolean;
}
```

#### `src/qortal/env.ts`

```ts
import { PUBLISHED_NAME } from '../config';

export function installHostStub() {
  if (typeof qortalRequest === 'function') return;
  const reject = () => Promise.reject(new Error('Open this app in Qortal Hub or Qortal GO.'));
  (window as any).qortalRequest = reject;
  (window as any).qortalRequestWithTimeout = reject;
  window.__QORTAL_STUB__ = true;
}
export const inHost = () => typeof qortalRequest === 'function' && !window.__QORTAL_STUB__ && !window.__QORTAL_DEV_SHIM__;
export const isDevMode = () => window._qdnContext === 'proxy';
export const isGateway = () => window._qdnContext === 'gateway';
export const isGo = () => /Android/i.test(navigator.userAgent);

/** The app's own name (decoded), with a fallback for Dev Mode where _qdnName is "". */
export function appName(): string {
  const n = window._qdnName;
  if (n && n.trim()) { try { return decodeURIComponent(n); } catch { return n; } }
  return PUBLISHED_NAME;
}
```

#### `src/qortal/request.ts`

```ts
export type QortalReq = { action: string } & Record<string, unknown>;

export function qortal<T = any>(req: QortalReq, timeoutMs?: number): Promise<T> {
  if (typeof qortalRequest !== 'function') return Promise.reject(new Error('Open this app in Qortal Hub or Qortal GO.'));
  if (timeoutMs && typeof qortalRequestWithTimeout === 'function') return qortalRequestWithTimeout(req, timeoutMs);
  return qortalRequest(req);
}

/** Core {error:number,message} · Hub {error:string,message} · strings · {error:{cancelled:true}} · Error */
export function errorMessage(e: unknown, fallback = 'Something went wrong'): string {
  if (typeof e === 'string') return e || fallback;
  const x = e as { error?: unknown; message?: unknown } | null;
  if (typeof x?.message === 'string' && x.message) return x.message;
  if (typeof x?.error === 'string' && x.error) return x.error;
  return fallback;
}
const DECLINE = /declin|cancel|rechaz|refus|ablehn|abgelehnt|отклон|拒否|拒绝|kieltäyt|keeldus|rifiut|recus|رفض/i;
export function isDecline(e: unknown): boolean {
  const x = e as { error?: { cancelled?: unknown } } | null;
  if (x && typeof x === 'object' && x.error && typeof x.error === 'object' && x.error.cancelled) return true;
  return DECLINE.test(errorMessage(e, ''));
}
export const isTimeout = (e: unknown) => /timed out/i.test(errorMessage(e, ''));   // Hub's timeout text is not translated
// Fallback only (Hub translates this refusal). Primary check: usingPublicNode() once at start-up.
const PUBLIC_NODE = /public node|öffentlichen Node|nodo público|nœud public|nodo pubblico|nó público|публичный узел|パブリックノード|公共节点|julkisen solmun|avaliku haarde|نود عام/i;
export const isPublicNodeRefusal = (e: unknown) => PUBLIC_NODE.test(errorMessage(e, ''));
/** No dialog; true when Hub/GO talks to a public node (lists, hosted data, foreign coins are refused there). */
export async function usingPublicNode(): Promise<boolean> {
  try { return (await qortal<boolean>({ action: 'IS_USING_PUBLIC_NODE' }, 5000)) === true; } catch { return false; }
}
export const HUB_DIALOG_GRACE_MS = 35_000;
export const sleep = (ms: number) => new Promise<void>(r => setTimeout(r, ms));

export type WriteResult = 'done' | 'declined' | 'unknown' | { failed: string };
/** A timeout is "unknown": wait out Hub's dialog, then look before anyone retries. */
export async function writeThenVerify(send: () => Promise<unknown>, landed: () => Promise<boolean>): Promise<WriteResult> {
  try { await send(); return 'done'; }
  catch (e) {
    if (isDecline(e)) return 'declined';
    if (isTimeout(e)) {
      await sleep(HUB_DIALOG_GRACE_MS);
      return (await landed().catch(() => false)) ? 'done' : 'unknown';
    }
    return { failed: errorMessage(e) };
  }
}
```

#### `src/qortal/search.ts`

```ts
export const PAGE = 20;
const TTL_MS = 120_000;

export interface SearchParams {
  service: string; identifier?: string; prefix?: boolean; name?: string; names?: string[];
  query?: string; title?: string; description?: string;
  limit?: number; offset?: number; reverse?: boolean; mode?: 'ALL' | 'LATEST';
  includemetadata?: boolean; excludeblocked?: boolean; followedonly?: boolean; before?: number; after?: number;
}
export interface ResourceRow {
  name: string; service: string; identifier: string; latestSignature?: string; size?: number; created?: number; updated?: number;
  metadata?: { title?: string; description?: string; category?: string; tags?: string[] };
}

export function searchUrl(p: SearchParams): string {
  if (p.prefix && (p.query || p.title || p.description))   // prefix=true makes EVERY term starts-with (6.3)
    throw new Error('Search text without prefix: send the identifier as a contains-match and re-check it in code');
  const q = new URLSearchParams();                          // encodes + / # & correctly
  q.set('service', p.service);
  q.set('mode', p.mode ?? 'ALL');                           // LATEST = one row per (name, service)
  if (p.identifier) q.set('identifier', p.identifier);
  if (p.prefix) q.set('prefix', 'true');
  const names = [...new Set([p.name, ...(p.names ?? [])].filter(Boolean) as string[])];
  names.forEach(n => q.append('name', n));
  if (names.length) q.set('exactmatchnames', 'true');
  if (p.query) q.set('query', p.query);
  if (p.title) q.set('title', p.title);
  if (p.description) q.set('description', p.description);
  if (p.before) q.set('before', String(p.before));
  if (p.after) q.set('after', String(p.after));
  q.set('limit', String(p.limit && p.limit > 0 ? Math.floor(p.limit) : PAGE)); // never 0 (= everything)
  q.set('offset', String(p.offset ?? 0));
  q.set('reverse', String(p.reverse ?? true));
  if (p.includemetadata) q.set('includemetadata', 'true');
  q.set('excludeblocked', String(p.excludeblocked ?? true));
  if (p.followedonly) q.set('followedonly', 'true');
  return `/arbitrary/resources/search?${q}`;
}

const inflight = new Map<string, Promise<ResourceRow[]>>();
const cache = new Map<string, { rows: ResourceRow[]; expires: number }>();

export async function searchQdn(p: SearchParams, opts: { fresh?: boolean } = {}): Promise<ResourceRow[]> {
  const url = searchUrl(p);
  if (!opts.fresh) {
    const hit = cache.get(url);
    if (hit && hit.expires > Date.now()) return hit.rows;
    const pending = inflight.get(url);
    if (pending) return pending;
  }
  const req = (async () => {
    const res = await fetch(url);
    if (!res.ok) throw new Error(`Search failed (${res.status})`);
    const data = await res.json();
    const rows: ResourceRow[] = (Array.isArray(data) ? data : []).map((r: any) => ({ ...r, identifier: r.identifier ?? 'default' }));
    cache.set(url, { rows, expires: Date.now() + TTL_MS });  // failures are never cached
    return rows;
  })();
  inflight.set(url, req);
  const forget = () => { if (inflight.get(url) === req) inflight.delete(url); };
  req.then(forget, forget);
  return req;
}

export function invalidateSearches(match: (url: string) => boolean = () => true) {
  for (const k of [...cache.keys()]) if (match(k)) cache.delete(k);
}

/** Exact lookup (search is "contains"). `since`: only versions published after this time (ms). */
export async function findExact(service: string, name: string, identifier: string, since?: number) {
  const rows = await searchQdn({ service, name, identifier, prefix: true, limit: 50, excludeblocked: false }, { fresh: true });
  return rows.find(r => r.identifier === identifier && r.name.toLowerCase() === name.toLowerCase() &&
    (!since || (r.updated ?? r.created ?? 0) >= since - 120_000));
}

/** Bounded "all": pages until a short page; reports whether it got everything. */
export async function searchAll(p: Omit<SearchParams, 'limit' | 'offset'>, pageSize = 100, maxPages = 5) {
  const rows: ResourceRow[] = [];
  for (let i = 0; i < maxPages; i++) {
    const chunk = await searchQdn({ ...p, limit: pageSize, offset: i * pageSize });
    rows.push(...chunk);
    if (chunk.length < pageSize) return { rows, complete: true };
  }
  return { rows, complete: false };
}

export async function mapLimit<T, R>(items: T[], n: number, task: (t: T, i: number) => Promise<R>): Promise<R[]> {
  const out = new Array<R>(items.length); let next = 0;
  await Promise.all(Array.from({ length: Math.min(n, items.length) }, async () => {
    while (next < items.length) { const i = next++; out[i] = await task(items[i], i); }
  }));
  return out;
}
```

#### `src/qortal/resource.ts`

```ts
import { errorMessage, sleep } from './request';

export const resourcePath = (service: string, name: string, identifier = 'default') =>
  `/arbitrary/${service}/${encodeURIComponent(name)}/${encodeURIComponent(identifier)}`;
export const avatarUrl = (name: string) => `/arbitrary/THUMBNAIL/${encodeURIComponent(name)}/qortal_avatar?async=true`;

// ---- UTF-8 safe base64 ----
export function bytesToBase64(bytes: Uint8Array): string {
  let bin = ''; const CH = 0x8000;
  for (let i = 0; i < bytes.length; i += CH) bin += String.fromCharCode(...bytes.subarray(i, i + CH));
  return btoa(bin);
}
export function base64ToBytes(b64: string): Uint8Array {
  const bin = atob(b64); const out = new Uint8Array(bin.length);
  for (let i = 0; i < bin.length; i++) out[i] = bin.charCodeAt(i);
  return out;
}
export const objectToBase64 = (o: unknown) => bytesToBase64(new TextEncoder().encode(JSON.stringify(o)));
export const base64ToObject = <T = unknown>(b64: string): T => JSON.parse(new TextDecoder().decode(base64ToBytes(b64)));

export class NotYetAvailable extends Error {}

/** One GET; parsed JSON or text. `async` = don't block the node: 404 means "not here yet". */
export async function fetchResource(service: string, name: string, identifier: string, opts: { async?: boolean; base64?: boolean } = {}) {
  const q = new URLSearchParams();
  if (opts.async) q.set('async', 'true');
  if (opts.base64) q.set('encoding', 'base64');
  const qs = q.toString();
  const res = await fetch(resourcePath(service, name, identifier) + (qs ? `?${qs}` : ''));
  const text = await res.text();
  let body: unknown = text;
  try { body = JSON.parse(text); } catch { /* text body */ }
  if (res.status === 404 || /data unavailable/i.test(text)) throw new NotYetAvailable(errorMessage(body, 'Not on this node yet'));
  if (!res.ok || (body && typeof body === 'object' && 'error' in (body as object))) throw new Error(errorMessage(body, `QDN ${res.status}`));
  if (!text) throw new Error('Empty response');
  return body;
}

export const RETRY_MS = [2000, 4000, 8000, 16000];
const whenVisible = () => document.visibilityState !== 'hidden' ? Promise.resolve() : new Promise<void>(r => {
  const h = () => { if (document.visibilityState !== 'hidden') { document.removeEventListener('visibilitychange', h); r(); } };
  document.addEventListener('visibilitychange', h);
});

/** Small resources (JSON/text): async fetch with backoff while the node fetches from peers. */
export async function fetchWithRetry(service: string, name: string, identifier: string,
  onWaiting?: (attempt: number) => void, signal?: AbortSignal) {
  for (let attempt = 0; ; attempt++) {
    try { return await fetchResource(service, name, identifier, { async: true }); }
    catch (e) {
      if (!(e instanceof NotYetAvailable) || attempt >= RETRY_MS.length || signal?.aborted) throw e;
      onWaiting?.(attempt);
      await sleep(RETRY_MS[attempt]); await whenVisible();
    }
  }
}

/** Tombstones: "D" (some tools "\n"), or {deleted:true}/{cleared:true}. */
export function isDeleted(body: unknown): boolean {
  if (typeof body === 'string') { const t = body.trim(); return t === 'D' || t === ''; }
  return !!body && typeof body === 'object' && ((body as any).deleted === true || (body as any).cleared === true);
}

export interface Status { status: string; percentLoaded?: number; localChunkCount?: number; totalChunkCount?: number }
export async function getStatus(service: string, name: string, identifier: string, build = false): Promise<Status> {
  const res = await fetch(`/arbitrary/resource/status/${service}/${encodeURIComponent(name)}/${encodeURIComponent(identifier)}${build ? '?build=true' : ''}`);
  const body = await res.json().catch(() => null);
  if (!body || typeof body !== 'object' || 'error' in body) throw new Error('Status unavailable');
  return body as Status;
}

const FAILED = new Set(['NOT_PUBLISHED', 'BLOCKED', 'UNSUPPORTED', 'BUILD_FAILED']);
/** Big files: poll status (visible tab only), build when DOWNLOADED, report progress. */
export async function waitUntilReady(ref: { service: string; name: string; identifier: string },
  onProgress: (pct: number, status: string) => void, signal?: AbortSignal): Promise<'ready' | 'failed'> {
  const started = Date.now(); let failures = 0, lastBuild = 0;
  while (!signal?.aborted) {
    await whenVisible();
    try {
      let s = await getStatus(ref.service, ref.name, ref.identifier);
      failures = 0;
      if (s.status === 'READY') return 'ready';
      if (s.status === 'NOT_PUBLISHED' && Date.now() - started < 60_000) { await sleep(3000); continue; } // may still be arriving
      if (FAILED.has(s.status)) return 'failed';
      if (s.status === 'DOWNLOADED' && Date.now() - lastBuild > 7500) {
        lastBuild = Date.now();
        s = await getStatus(ref.service, ref.name, ref.identifier, true).catch(() => s);
        if (s.status === 'READY') return 'ready';
      }
      onProgress(Math.round(s.percentLoaded ?? 0), s.status);
    } catch { if (++failures >= 6) return 'failed'; }
    await sleep(5000);
  }
  return 'failed';
}
```

#### `src/qortal/publish.ts`

```ts
import { qortal, isDecline, isTimeout, errorMessage, writeThenVerify, sleep, type WriteResult } from './request';
import { objectToBase64 } from './resource';
import { findExact, invalidateSearches } from './search';

export async function publishJson(o: { name: string; service: string; identifier: string; doc: unknown;
  title?: string; description?: string; tags?: string[]; filename?: string }): Promise<WriteResult> {
  const started = Date.now();
  const r = await writeThenVerify(
    () => qortal({
      action: 'PUBLISH_QDN_RESOURCE', name: o.name, service: o.service, identifier: o.identifier,
      data64: objectToBase64(o.doc), filename: o.filename ?? 'data.json',
      ...(o.title ? { title: o.title.slice(0, 80) } : {}),
      ...(o.description ? { description: o.description.slice(0, 240) } : {}),
      ...(o.tags ? { tags: o.tags.filter(t => t.length >= 1 && t.length <= 20).slice(0, 5) } : {}),
    }, 60 * 60_000),
    async () => !!(await findExact(o.service, o.name, o.identifier, started)),
  );
  if (r === 'done') invalidateSearches(u => u.includes(encodeURIComponent(o.service)));
  return r;
}

// ---- Many resources: batches under Core's 25-unconfirmed limit ----
const MAX_UNCONFIRMED = 25, PER_BATCH = 20, MS_PER_RESOURCE = 30 * 60_000;
type Res = { service: string; name: string; identifier: string; [k: string]: unknown };

export function splitBatches<T>(res: T[], isIndex: (r: T) => boolean, n = PER_BATCH): T[][] {
  const files = res.filter(r => !isIndex(r)), index = res.filter(isIndex), out: T[][] = [];
  for (let i = 0; i < files.length; i += n) out.push(files.slice(i, i + n));
  if (index.length) { if (!out.length || out[out.length - 1].length + index.length > n) out.push([]); out[out.length - 1].push(...index); }
  return out;
}
async function unconfirmedCount(publicKey: string): Promise<number | null> {
  try {
    const rows = await (await fetch(`/transactions/unconfirmed?creator=${encodeURIComponent(publicKey)}&limit=100`)).json();
    return Array.isArray(rows) ? rows.filter((t: any) => t?.type !== 'CHAT').length : null;
  } catch { return null; }
}
async function waitForRoom(publicKey: string, needed: number, signal?: AbortSignal) {
  const max = MAX_UNCONFIRMED - Math.min(needed, MAX_UNCONFIRMED);
  while (!signal?.aborted) {
    const n = await unconfirmedCount(publicKey);
    if (n === null || n <= max) return;
    await sleep(document.visibilityState === 'hidden' ? 30_000 : 10_000);   // a block is ~1 minute
  }
}

/** Publishes in batches; the index resource goes last. Never resends blindly. */
export async function publishAll(publicKey: string, resources: Res[], isIndex: (r: Res) => boolean, signal?: AbortSignal) {
  const done = new Set<string>();
  const batches = splitBatches(resources, isIndex);
  for (const batch of batches) {
    if (batches.length > 1) await waitForRoom(publicKey, batch.length, signal);
    try {
      await qortal({ action: 'PUBLISH_MULTIPLE_QDN_RESOURCES', resources: batch }, batch.length * MS_PER_RESOURCE);
      batch.forEach(r => done.add(r.identifier));
    } catch (e: any) {
      if (isDecline(e)) return { done, stopped: 'declined' as const };
      const failed: string[] = (e?.error?.unsuccessfulPublishes ?? []).map((x: any) => x?.identifier).filter(Boolean);
      if (failed.length) {                                   // the rest DID publish
        batch.filter(r => !failed.includes(r.identifier)).forEach(r => done.add(r.identifier));
        return { done, stopped: 'partial' as const, failed };
      }
      if (isTimeout(e)) {                                    // Hub may still be publishing: ask QDN
        for (const r of batch) if (await findExact(r.service, r.name, r.identifier)) done.add(r.identifier);
        return { done, stopped: 'unknown' as const };
      }
      return { done, stopped: 'failed' as const, message: errorMessage(e) };
    }
  }
  invalidateSearches();
  return { done, stopped: null };
}
// Retry = call publishAll again with only the resources NOT in `done`, keeping the same identifiers.
```

#### `src/qortal/image.ts` (photos: resize, thumbnail, publish together)

```ts
import { publishAll } from './publish';
import { objectToBase64 } from './resource';

/** Downscale a picked photo before publishing (10.3). null = could not decode or not small enough. */
export async function resizeImage(file: Blob, maxPx: number, quality: number, maxBytes: number): Promise<Blob | null> {
  let bmp: ImageBitmap | HTMLImageElement;
  try { bmp = await createImageBitmap(file, { imageOrientation: 'from-image' }); }      // applies EXIF rotation
  catch {
    const url = URL.createObjectURL(file);
    try { bmp = await new Promise<HTMLImageElement>((ok, no) => { const i = new Image(); i.onload = () => ok(i); i.onerror = no; i.src = url; }); }
    catch { return null; } finally { URL.revokeObjectURL(url); }
  }
  const w0 = bmp.width, h0 = bmp.height;
  if (!w0 || !h0 || w0 * h0 > 50e6) { if ('close' in bmp) bmp.close(); return null; }   // refuse > 50 MP (phones run out of memory)
  const scale = Math.min(1, maxPx / Math.max(w0, h0));
  const c = document.createElement('canvas');
  c.width = Math.max(1, Math.round(w0 * scale)); c.height = Math.max(1, Math.round(h0 * scale));
  c.getContext('2d')!.drawImage(bmp, 0, 0, c.width, c.height);
  if ('close' in bmp) bmp.close();
  for (const type of ['image/webp', 'image/jpeg']) {
    for (let q = quality; q >= 0.4; q -= 0.15) {
      const out = await new Promise<Blob | null>(r => c.toBlob(r, type, q));
      if (!out || out.type !== type) break;               // this WebView can't encode the type: try the next one
      if (out.size <= maxBytes) return out;
    }
  }
  return null;
}

/**
 * One dialog for photo + thumbnail + JSON. Sibling identifiers share one suffix, so a grid can
 * build the thumbnail URL straight from a search row (7.2). The JSON goes last: it is the index.
 */
export async function publishWithPhoto(acct: { publicKey: string; name: string }, photo: File, doc: Record<string, unknown>,
  ids = { json: 'myapp_recipe_', img: 'myapp_img_', thumb: 'myapp_thumb_' }) {
  const [img, thumb] = await Promise.all([
    resizeImage(photo, 1600, 0.82, 2_000_000),           // IMAGE (Core cap 10 MiB)
    resizeImage(photo, 400, 0.72, 450_000),              // THUMBNAIL (Core cap 500 KiB)
  ]);
  if (!img || !thumb) throw new Error('This photo could not be prepared. Try another one.');
  const s = `${Date.now().toString(36)}${Math.random().toString(36).slice(2, 8)}`;
  const ext = img.type === 'image/webp' ? 'webp' : 'jpg';
  return publishAll(acct.publicKey, [
    { service: 'IMAGE', name: acct.name, identifier: ids.img + s, file: img, filename: `photo.${ext}` },
    { service: 'THUMBNAIL', name: acct.name, identifier: ids.thumb + s, file: thumb, filename: `thumb.${ext}` },
    { service: 'JSON', name: acct.name, identifier: ids.json + s, filename: 'data.json',
      data64: objectToBase64({ ...doc, image: ids.img + s, thumbnail: ids.thumb + s }) },
  ], r => r.service === 'JSON');                         // costs 3 × 0.01 QORT: show that on the button
}
```

Use `<input type="file" accept="image/*">` **without** `capture`, so phones offer both the gallery and the camera.

#### `src/qortal/links.ts`

```ts
import { appName } from './env';
const appSegment = (name: string) => name.replace(/ /g, '%20');          // "+" stays literal: Hub never decodes it
export const appLink = (path: string, name = appName()) => `qortal://APP/${appSegment(name)}/${path.replace(/^\/+/, '')}`;
export const postPath = (publisher: string, id: string) => `/post/${encodeURIComponent(publisher)}/${encodeURIComponent(id)}`;
export const postLink = (publisher: string, id: string) => appLink(postPath(publisher, id));
// Only for raw strings (location.pathname, _qdnName). useParams() values are already decoded: don't decode them again.
export const decodeParam = (v?: string) => { if (!v) return ''; try { return decodeURIComponent(v); } catch { return v; } };
const HUB_KEYS = ['theme', 'lang', 'identifier', 'time', 'isManualNavigation', 'goWidget'];
export function cleanSearch(search: string) {
  const p = new URLSearchParams(search); HUB_KEYS.forEach(k => p.delete(k));
  const s = p.toString(); return s ? `?${s}` : '';
}
export async function copyText(text: string) {
  try { await navigator.clipboard.writeText(text); return true; } catch { return false; }
}
```

#### `src/hub/routerBase.ts`

```ts
const dec = (v: string) => { try { return decodeURIComponent(v).toLowerCase(); } catch { return v.toLowerCase(); } };
/**
 * Basename = Core's _qdnBase; tolerate Hub's spelling of the name ("%2B" vs "+") and non-ASCII
 * names (Core injects them raw, the browser keeps the path percent-encoded); "" in Dev Mode.
 */
export function routerBase(qdnBase = window._qdnBase): string {
  if (!qdnBase) return '';
  const n = qdnBase.split('/').length;
  const head = () => location.pathname.split('/').slice(0, n).join('/');
  let prefix = head();
  if (prefix === qdnBase || dec(prefix) !== dec(qdnBase)) return qdnBase;
  try { history.replaceState(history.state, '', qdnBase + location.pathname.slice(prefix.length) + location.search + location.hash); } catch { /* ignore */ }
  prefix = head();
  if (prefix !== qdnBase && dec(prefix) === dec(qdnBase)) {
    window._qdnBase = prefix;          // q-apps.js strips _qdnBase when it reports routes to Hub
    return prefix;
  }
  return qdnBase;
}
```

#### `src/hub/hostMessages.ts`

```ts
import { storageKey } from '../config';

export function fromHost(e: MessageEvent): boolean {
  let w: Window = window;
  while (w !== window.top) { w = w.parent; if (e.source === w) return true; }
  return false;
}
export type Mode = 'light' | 'dark';
export const currentMode = (): Mode => document.documentElement.getAttribute('data-theme') === 'light' ? 'light' : 'dark';

export function applyMode(mode: Mode) {
  try { localStorage.setItem(storageKey('hub-mode'), mode); } catch { /* storage blocked */ }
  window._qdnTheme = mode;
  const r = document.documentElement;
  r.setAttribute('data-theme', mode); r.style.colorScheme = mode; r.style.background = '';
}
export function bootLanguage(): string {
  const q = new URLSearchParams(location.search).get('lang');
  if (q && /^[\w-]{1,20}$/.test(q)) { try { localStorage.setItem(storageKey('hub-lang'), q); } catch {} return q; }
  try { const s = localStorage.getItem(storageKey('hub-lang')); if (s) return s; } catch {}
  return (navigator.language || 'en').split('-')[0];
}
export function listenToHost(h: { theme?: (m: Mode) => void; language?: (l: string) => void }) {
  const onMessage = (e: MessageEvent) => {
    if (!fromHost(e)) return;
    const d = e.data as any;
    if (d?.action === 'THEME_CHANGED' && (d.theme === 'light' || d.theme === 'dark')) { applyMode(d.theme); h.theme?.(d.theme); }
    if (d?.action === 'LANGUAGE_CHANGED' && typeof d.language === 'string' && /^[\w-]{1,20}$/.test(d.language)) {
      try { localStorage.setItem(storageKey('hub-lang'), d.language); } catch {}
      window._qdnLang = d.language; h.language?.(d.language);
    }
  };
  window.addEventListener('message', onMessage);
  return () => window.removeEventListener('message', onMessage);
}
```

#### `src/hub/navigation.tsx`

```tsx
import { useCallback, useEffect, useRef } from 'react';
import { useLocation, useNavigate, useNavigationType } from 'react-router-dom';
import { fromHost } from './hostMessages';

/** Answer Hub's Back / deep links at once, or Hub reloads the frame. */
export function useHubNavigation() {
  const navigate = useNavigate();
  useEffect(() => {
    const onMessage = (e: MessageEvent) => {
      if (!fromHost(e)) return;
      const d = e.data as { action?: string; path?: unknown };
      if (d?.action === 'NAVIGATE_TO_PATH' && typeof d.path === 'string') {
        navigate('/' + d.path.replace(/^\/+/, ''));
        window.parent.postMessage({ action: 'NAVIGATION_SUCCESS', path: d.path }, '*');
      }
    };
    window.addEventListener('message', onMessage);
    return () => window.removeEventListener('message', onMessage);
  }, [navigate]);
}

// ---- In-app back stack (window.history is shared by all Hub tabs) ----
const stack: string[] = [];
let goingBack = false;
const scrollPos = new Map<string, number>();

export function useTrackInAppHistory() {
  const loc = useLocation(); const type = useNavigationType();
  const last = useRef<string>('');
  useEffect(() => {
    const path = loc.pathname + loc.search;
    if (last.current) scrollPos.set(last.current, window.scrollY);
    last.current = path;
    if (goingBack) goingBack = false;
    else if (type === 'REPLACE' && stack.length) stack[stack.length - 1] = path;
    else if (stack.length > 1 && stack[stack.length - 2] === path) stack.pop();
    else if (stack[stack.length - 1] !== path) stack.push(path);
    // New page at top; Back restores the saved position.
    requestAnimationFrame(() => window.scrollTo(0, type === 'POP' ? (scrollPos.get(path) ?? 0) : 0));
  }, [loc, type]);
}

export function useSafeBack(fallback = '/') {
  const navigate = useNavigate();
  return useCallback(() => {
    if (stack.length > 1) { stack.pop(); goingBack = true; navigate(stack[stack.length - 1]); }
    else navigate(fallback, { replace: true });            // opened by deep link: go to a parent page
  }, [fallback, navigate]);
}
```

Note: the back button above does a PUSH navigation (not POP), so it scrolls to top; if you want restored scroll on in-app Back too, call `scrollPos.get(target)` there.

#### `src/hub/frame.ts`

```ts
export function watchFrameHeight() {
  const apply = () => {
    let framed = true; try { framed = window.parent !== window; } catch { /* cross-origin */ }
    document.documentElement.style.setProperty('--app-height',
      framed && window.innerHeight > 0 ? `${Math.round(window.innerHeight)}px` : '100dvh');
  };
  apply(); window.addEventListener('resize', apply);
  return () => window.removeEventListener('resize', apply);
}
export const PHONE_MEDIA = '(max-width: 599.95px), ((pointer: coarse) and (max-height: 500px))';
```

#### `src/lib/poll.ts`, `src/lib/lazyWithRetry.ts`, `src/lib/sanitize.ts`

Copy the code from 14.3 (`poll`), 14.6 (`lazyWithRetry` + `vite:preloadError`) and 13 (`sanitizeHtml`).

#### `src/state/account.ts`

```ts
import { useSyncExternalStore } from 'react';
import { qortal, isTimeout, errorMessage, HUB_DIALOG_GRACE_MS } from '../qortal/request';
import { storageKey } from '../config';
import { invalidateSearches } from '../qortal/search';
import { inHost } from '../qortal/env';

/** balance: QORT, null while unknown. */
export interface Account { address: string; publicKey: string; names: string[]; name: string; balance: number | null }
type State = { account: Account | null; busy: boolean; error: string };
let state: State = { account: null, busy: false, error: '' };
const subs = new Set<() => void>();
const set = (p: Partial<State>) => { state = { ...state, ...p }; subs.forEach(f => f()); };

export const useAccount = () => useSyncExternalStore(cb => { subs.add(cb); return () => subs.delete(cb); }, () => state);

export async function signIn(auto = false) {
  if (state.busy) return;
  set({ busy: true, error: '' });
  try {
    const acct = await qortal<{ address: string; publicKey: string }>({ action: 'GET_USER_ACCOUNT' });
    const namesRes = await fetch(`/names/address/${encodeURIComponent(acct.address)}?limit=100`).then(r => r.json()).catch(() => []);
    const names = (Array.isArray(namesRes) ? namesRes : []).map((n: any) => n?.name).filter((n: unknown): n is string => typeof n === 'string');
    let primary = '';
    try { primary = await qortal<string>({ action: 'GET_PRIMARY_NAME', address: acct.address }); }
    catch { primary = await fetch(`/names/primary/${encodeURIComponent(acct.address)}`).then(r => r.json()).then(r => r?.name ?? '').catch(() => ''); }
    let chosen = '';
    try { chosen = localStorage.getItem(storageKey(`name-${acct.address}`)) ?? ''; } catch {}
    const name = names.includes(chosen) ? chosen : (typeof primary === 'string' && primary) || names[0] || '';
    if (state.account?.address !== acct.address) invalidateSearches();   // account switch: drop derived data
    set({ account: { address: acct.address, publicKey: acct.publicKey, names, name, balance: null } });
    try { localStorage.setItem(storageKey('signed-in'), '1'); } catch {}
    void refreshBalance();
  } catch (e) {
    if (isTimeout(e)) setTimeout(() => { if (!state.account) void signIn(true); }, HUB_DIALOG_GRACE_MS);
    // Hub turns EVERY GET_USER_ACCOUNT failure, a decline included, into a translated "unable to fetch
    // user account": in Hub any other rejection is a quiet "not signed in". Outside Hub, say why.
    else if (!auto && !inHost()) set({ error: errorMessage(e) });
  } finally { set({ busy: false }); }
}
/** GET /addresses/balance/{address} answers a plain number. Check it before showing publish/pay UI (7.5). */
export async function refreshBalance() {
  const a = state.account; if (!a) return;
  const v = await fetch(`/addresses/balance/${encodeURIComponent(a.address)}`).then(r => r.json()).catch(() => null);
  if (state.account?.address === a.address) set({ account: { ...state.account, balance: typeof v === 'number' ? v : null } });
}
export function chooseName(name: string) {
  if (!state.account || !state.account.names.includes(name)) return;
  try { localStorage.setItem(storageKey(`name-${state.account.address}`), name); } catch {}
  set({ account: { ...state.account, name } });
}
```

#### `src/components/NameAvatar.tsx`

```tsx
import { useEffect, useRef, useState } from 'react';
import { avatarUrl } from '../qortal/resource';

const known = new Map<string, 'ok' | 'none'>();
const jobs = new Map<string, Promise<boolean>>();
const DELAYS = [2000, 4000, 8000, 15000];

function loadAvatar(name: string): Promise<boolean> {          // one shared job per name
  const cached = known.get(name);
  if (cached) return Promise.resolve(cached === 'ok');
  let job = jobs.get(name);
  if (job) return job;
  job = new Promise<boolean>(resolve => {
    let attempt = 0;
    const tryOnce = () => {
      const img = new Image();
      img.onload = () => { known.set(name, 'ok'); jobs.delete(name); resolve(true); };
      img.onerror = async () => {
        const s = await fetch(`/arbitrary/resource/status/THUMBNAIL/${encodeURIComponent(name)}/qortal_avatar`)
          .then(r => r.json()).catch(() => null);
        if (s?.status === 'NOT_PUBLISHED') { known.set(name, 'none'); jobs.delete(name); return resolve(false); }
        if (attempt >= DELAYS.length) { jobs.delete(name); return resolve(false); }   // not cached: try again later
        setTimeout(tryOnce, DELAYS[attempt++]);
      };
      img.src = avatarUrl(name);
    };
    tryOnce();
  });
  jobs.set(name, job);
  return job;
}

export function NameAvatar({ name, size = 36 }: { name: string; size?: number }) {
  const ref = useRef<HTMLSpanElement>(null);
  const [ok, setOk] = useState(known.get(name) === 'ok');
  useEffect(() => {
    if (ok || !ref.current) return;
    let alive = true;
    const io = new IntersectionObserver(([e]) => {
      if (!e.isIntersecting) return;
      io.disconnect();
      loadAvatar(name).then(v => { if (alive && v) setOk(true); });
    }, { rootMargin: '200px' });
    io.observe(ref.current);
    return () => { alive = false; io.disconnect(); };
  }, [name, ok]);
  return (
    <span ref={ref} aria-hidden style={{ width: size, height: size, borderRadius: '50%', overflow: 'hidden', flex: 'none',
      display: 'inline-grid', placeItems: 'center', background: 'var(--border)', fontWeight: 600 }}>
      {ok ? <img src={avatarUrl(name)} alt="" width={size} height={size} style={{ objectFit: 'cover' }} />
          : (Array.from(name)[0] ?? '?').toUpperCase()}
    </span>
  );
}
```

#### `src/data/post.ts` (shape + validation, pure and tested)

```ts
import type { ResourceRow } from '../qortal/search';
export interface PostDoc { v: 1; text: string }
export interface Post { name: string; identifier: string; created?: number; text: string }
export const MAX_TEXT = 2000;
export const newPostId = (prefix: string) => `${prefix}${Date.now().toString(36)}${Math.random().toString(36).slice(2, 8)}`;
export const buildPost = (text: string): PostDoc => ({ v: 1, text: text.trim().slice(0, MAX_TEXT) });
/** Author/date from the row; body validated; anything odd → null (skipped). */
export function readPost(row: ResourceRow, body: unknown): Post | null {
  if (!body || typeof body !== 'object' || Array.isArray(body)) return null;
  const b = body as Record<string, unknown>;
  if (b.v !== 1 || typeof b.text !== 'string' || !b.text.trim()) return null;
  return { name: row.name, identifier: row.identifier, created: row.created, text: b.text.slice(0, MAX_TEXT) };
}
```

#### `src/screens/Feed.tsx`

```tsx
import { useCallback, useEffect, useRef, useState } from 'react';
import { Link } from 'react-router-dom';
import { ID } from '../config';
import { searchQdn, PAGE, type ResourceRow } from '../qortal/search';
import { fetchWithRetry, isDeleted } from '../qortal/resource';
import { publishJson } from '../qortal/publish';
import { errorMessage } from '../qortal/request';
import { readPost, buildPost, newPostId, type Post } from '../data/post';
import { useAccount, signIn, refreshBalance } from '../state/account';
import { postPath } from '../qortal/links';
import { NameAvatar } from '../components/NameAvatar';
import { ErrorBoundary } from '../components/ErrorBoundary';

// Bodies load only near the viewport, at most 5 at a time (6.8).
let active = 0;
const waiting: Array<() => void> = [];
function takeSlot(): Promise<() => void> {
  return new Promise(resolve => {
    const start = () => {
      active++;
      let freed = false;
      resolve(() => { if (!freed) { freed = true; active--; waiting.shift()?.(); } });
    };
    if (active < 5) start(); else waiting.push(start);
  });
}

function PostCard({ row }: { row: ResourceRow }) {
  const ref = useRef<HTMLElement>(null);
  const [near, setNear] = useState(false);
  const [post, setPost] = useState<Post | null | 'loading' | 'waiting' | 'gone' | 'unavailable'>('loading');
  useEffect(() => {
    if (near || !ref.current) return;
    const io = new IntersectionObserver(([e]) => { if (e.isIntersecting) { setNear(true); io.disconnect(); } }, { rootMargin: '400px' });
    io.observe(ref.current);
    return () => io.disconnect();
  }, [near]);
  useEffect(() => {
    if (!near) return;
    const ctl = new AbortController();
    let release: (() => void) | undefined;
    void takeSlot().then(free => {
      release = free;
      if (ctl.signal.aborted) return free();
      return fetchWithRetry('JSON', row.name, row.identifier, () => setPost('waiting'), ctl.signal)
        .then(body => { if (!ctl.signal.aborted) setPost(isDeleted(body) ? 'gone' : readPost(row, body) ?? 'gone'); })
        .catch(() => { if (!ctl.signal.aborted) setPost('unavailable'); })
        .finally(free);
    });
    return () => { ctl.abort(); release?.(); };
  }, [row, near]);
  if (post === 'gone') return null;
  return (
    <article ref={ref} style={{ background: 'var(--surface)', border: '1px solid var(--border)', borderRadius: 12, padding: 12, margin: '10px 0' }}>
      <div style={{ display: 'flex', gap: 10, alignItems: 'center' }}>
        <NameAvatar name={row.name} />
        <div style={{ minWidth: 0 }}>
          <div style={{ fontWeight: 600, overflowWrap: 'anywhere' }}>{row.name}</div>
          <Link to={postPath(row.name, row.identifier)} style={{ color: 'var(--muted)', fontSize: 14 }}>
            {row.created ? new Date(row.created).toLocaleString() : 'Open'}
          </Link>
        </div>
      </div>
      <p style={{ whiteSpace: 'pre-wrap', overflowWrap: 'anywhere', color: typeof post === 'object' ? 'var(--text)' : 'var(--muted)' }}>
        {post === 'loading' ? 'Loading…' : post === 'waiting' ? 'Fetching from peers…'
          : post === 'unavailable' ? 'Not available on your node right now.' : post?.text}
      </p>
    </article>
  );
}

export default function Feed() {
  const { account, busy } = useAccount();
  const [rows, setRows] = useState<ResourceRow[]>([]);
  const [nextOffset, setNextOffset] = useState(0);         // raw search offset (not rows.length)
  const [done, setDone] = useState(false);
  const [error, setError] = useState('');
  const [text, setText] = useState('');
  const [posting, setPosting] = useState(false);

  const load = useCallback(async (offset: number, fresh = false) => {
    setError('');
    try {
      const page = await searchQdn({ service: 'JSON', identifier: ID.post, prefix: true, limit: PAGE, offset }, { fresh });
      const mine = page.filter(r => r.identifier.startsWith(ID.post));                  // re-check: search is case-insensitive
      const key = (r: ResourceRow) => r.name + '\n' + r.identifier;
      setRows(prev => {
        const base = offset ? prev : [];
        const seen = new Set(base.map(key));
        return [...base, ...mine.filter(r => !seen.has(key(r)))];
      });
      setNextOffset(offset + page.length);
      setDone(page.length < PAGE);
    } catch (e) { setError(errorMessage(e)); }
  }, []);
  useEffect(() => { void load(0); }, [load]);

  const lowBalance = account?.balance != null && account.balance < 0.01;   // 7.5: check before the dialog
  const submit = async () => {
    if (!account?.name || !text.trim() || lowBalance) return;
    setPosting(true);
    const identifier = newPostId(ID.post);
    const r = await publishJson({ name: account.name, service: 'JSON', identifier, doc: buildPost(text), title: 'Post', filename: 'post.json' });
    setPosting(false);
    if (r === 'done') {
      setText('');
      setRows(prev => [{ name: account.name, service: 'JSON', identifier, created: Date.now() }, ...prev]); // optimistic
      void refreshBalance();
    } else if (r === 'unknown') setError('Not confirmed yet. Refresh in a minute before posting again.');
    else if (r !== 'declined') setError(r.failed);
  };

  return (
    <main style={{ maxWidth: 720, margin: '0 auto', padding: 16 }}>
      {!account && <button onClick={() => signIn()} disabled={busy}>Sign in</button>}
      {account && !account.name && <p>To post, register a name in Qortal Hub (1.25 QORT).</p>}
      {account?.name && (
        <section>
          <textarea value={text} onChange={e => setText(e.target.value)} maxLength={2000}
            style={{ width: '100%', minHeight: 96, fontSize: 16 }} aria-label="New post" />
          <button onClick={submit} disabled={posting || !text.trim() || lowBalance}>{posting ? 'Waiting for Hub…' : 'Post (0.01 QORT)'}</button>
          {lowBalance && <p>You need at least 0.01 QORT to post.</p>}
        </section>
      )}
      {error && <p role="alert">{error} <button onClick={() => load(0, true)}>Retry</button></p>}
      {!error && rows.length === 0 && done && <p>No posts yet. Write the first one.</p>}
      {rows.map(r => (
        <ErrorBoundary key={r.name + '\n' + r.identifier} fallback={<p style={{ color: 'var(--muted)' }}>This post could not be shown.</p>}>
          <PostCard row={r} />
        </ErrorBoundary>
      ))}
      {!done && <button onClick={() => load(nextOffset)}>Load more</button>}
    </main>
  );
}
```

A brand-new post is inserted optimistically; a fresh search later contains it once and the `key` dedupe keeps it single. Each card fetches its body only when it comes near the viewport, through a shared 5-slot limiter, and sits in its own error boundary. For long feeds, add virtualisation and an IntersectionObserver sentinel instead of the button.

#### `src/components/ErrorBoundary.tsx`

```tsx
import { Component, type ReactNode } from 'react';

/** Catches render errors and failed lazy chunks, so one bad document or screen never blanks the app (12.10, 14.6). */
export class ErrorBoundary extends Component<{ children: ReactNode; resetKey?: string; fallback?: ReactNode }, { failed: boolean }> {
  state = { failed: false };
  static getDerivedStateFromError() { return { failed: true }; }
  componentDidCatch(e: unknown) { console.error(e); }
  componentDidUpdate(prev: { resetKey?: string }) {
    if (prev.resetKey !== this.props.resetKey && this.state.failed) this.setState({ failed: false });  // new route: try again
  }
  render() {
    if (!this.state.failed) return this.props.children;
    return this.props.fallback ?? (
      <div role="alert" style={{ padding: 16 }}>
        This screen could not load. <button onClick={() => location.reload()}>Reload</button>
      </div>
    );
  }
}
```

#### `src/App.tsx`

```tsx
import { Suspense, useEffect } from 'react';
import { Routes, Route, useLocation, useParams } from 'react-router-dom';
import { lazyWithRetry } from './lib/lazyWithRetry';
import { useHubNavigation, useTrackInAppHistory, useSafeBack } from './hub/navigation';
import { listenToHost } from './hub/hostMessages';
import { ErrorBoundary } from './components/ErrorBoundary';

const Feed = lazyWithRetry(() => import('./screens/Feed'));

function PostPage() {
  const { name = '', id = '' } = useParams();     // already decoded by the router: never decode again
  const back = useSafeBack('/');
  return (
    <main style={{ padding: 16 }}>
      <button onClick={back} aria-label="Back">← Back</button>
      <p>Post {id} by {name}</p>
    </main>
  );
}

export default function App() {
  const { pathname } = useLocation();
  useHubNavigation();
  useTrackInAppHistory();
  useEffect(() => listenToHost({ /* theme is applied to <html>; re-read tokens here if your UI kit needs it */ }), []);
  return (
    <ErrorBoundary resetKey={pathname}>
      <Suspense fallback={<div style={{ padding: 16 }}>Loading…</div>}>
        <Routes>
          <Route path="/" element={<Feed />} />
          <Route path="/post/:name/:id" element={<PostPage />} />
          <Route path="*" element={<Feed />} />
        </Routes>
      </Suspense>
    </ErrorBoundary>
  );
}
```

#### `src/boot.ts` and `src/main.tsx`

```ts
// src/boot.ts — imported FIRST by main.tsx
import './devShim';                       // dev-only read-only shim outside Hub (no-op in builds and in Hub)
import { installHostStub } from './qortal/env';
import { watchFrameHeight } from './hub/frame';
installHostStub();
watchFrameHeight();
```

```tsx
// src/main.tsx
import './boot';
import './index.css';
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';   // WEBSITE service: use HashRouter instead
import App from './App';
import { routerBase } from './hub/routerBase';
import { signIn } from './state/account';
import { storageKey } from './config';
import { inHost } from './qortal/env';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <BrowserRouter basename={routerBase()}>
      <App />
    </BrowserRouter>
  </StrictMode>,
);
try { if (inHost() && localStorage.getItem(storageKey('signed-in')) === '1') void signIn(true); } catch { /* ignore */ }
```

#### `src/devShim.ts` (read-only, plain-browser dev only)

```ts
// Bare `typeof qortalRequest` (declared in globals.d.ts): globalThis.qortalRequest is undefined even in Hub.
if (import.meta.env.DEV && typeof qortalRequest !== 'function' && !window._qdnContext) {
  const get = async (u: string) => {
    const r = await fetch(u); const t = await r.text();
    let body: any = t; try { body = JSON.parse(t); } catch { /* text */ }
    if (body && typeof body === 'object' && 'error' in body) throw body;          // like q-apps.js
    return body;
  };
  // Build URLs EXACTLY like q-apps.js: raw values (no encoding) and no defaults (mode LATEST, no limit),
  // so a raw qortalRequest search behaves in the dev preview as it will in Hub.
  const qappsSearchUrl = (d: any) => {
    const parts: string[] = [];
    const raw = (k: string, v: unknown) => { if (v != null) parts.push(`${k}=${v}`); };
    const bool = (k: string, v: unknown) => { if (v != null) parts.push(`${k}=${Boolean(v)}`); };
    raw('service', d.service); raw('query', d.query); raw('identifier', d.identifier); raw('name', d.name);
    (d.names ?? []).forEach((x: string) => raw('name', x)); (d.keywords ?? []).forEach((x: string) => raw('keywords', x));
    raw('title', d.title); raw('description', d.description); bool('prefix', d.prefix); bool('exactmatchnames', d.exactMatchNames);
    bool('default', d.default); raw('mode', d.mode); raw('minlevel', d.minLevel); bool('includestatus', d.includeStatus);
    bool('includemetadata', d.includeMetadata); raw('namefilter', d.nameListFilter); bool('followedonly', d.followedOnly);
    bool('excludeblocked', d.excludeBlocked); raw('before', d.before); raw('after', d.after);
    raw('limit', d.limit); raw('offset', d.offset); bool('reverse', d.reverse);
    return `/arbitrary/resources/search?&${parts.join('&')}`;
  };
  const shim = async (p: any) => {
    switch (p.action) {
      case 'SEARCH_QDN_RESOURCES':
        if (p.mode == null || p.limit == null)
          console.warn('SEARCH_QDN_RESOURCES without mode/limit: Hub uses mode=LATEST and returns EVERYTHING (guide 6.3)');
        return get(qappsSearchUrl(p));
      case 'FETCH_QDN_RESOURCE':
        return get(`/arbitrary/${p.service}/${p.name}${p.identifier != null ? '/' + p.identifier : ''}?${p.encoding ? `&encoding=${p.encoding}` : ''}`);
      case 'GET_NAME_DATA': return get(`/names/${p.name}`);
      case 'GET_ACCOUNT_NAMES': return get(`/names/address/${p.address}`);
      case 'GET_PRIMARY_NAME': return (await get(`/names/primary/${encodeURIComponent(p.address)}`))?.name ?? '';
      case 'GET_QDN_RESOURCE_STATUS':
        return get(`/arbitrary/resource/status/${p.service}/${p.name}${p.identifier != null ? '/' + p.identifier : ''}`);
      default:
        throw { error: `Dev preview is read-only (${p.action}). Use Hub Dev Mode to sign in or publish.` };
    }
  };
  (window as any).qortalRequest = shim;
  (window as any).qortalRequestWithTimeout = (p: any) => shim(p);
  window.__QORTAL_DEV_SHIM__ = true;
}
export {};
```

In Hub Dev Mode `qortalRequest` exists and `_qdnContext` is `"proxy"`, so the shim never replaces the real host. The shim copies q-apps.js's lack of encoding on purpose: if a name with `+` or `/` breaks here, it breaks in Hub too (6.2). Your own `fetch` code (Template B `search.ts`) encodes properly.

#### `src/test/setup.ts` (vitest + jsdom)

```ts
import { beforeEach, vi } from 'vitest';

type P = Record<string, unknown> & { action: string };
const handlers = new Map<string, (p: P) => unknown>();
export const calls: P[] = [];
export const mockAction = (action: string, answer: unknown) =>
  handlers.set(action, typeof answer === 'function' ? (answer as (p: P) => unknown) : () => answer);
const qr = vi.fn(async (p: P) => {
  calls.push(p);
  const h = handlers.get(p.action);
  if (!h) throw new Error(`no mock for ${p.action}`);
  return h(p);
});
Object.defineProperty(globalThis, 'qortalRequest', { value: qr, writable: true, configurable: true });
Object.defineProperty(globalThis, 'qortalRequestWithTimeout', { value: vi.fn((p: P) => qr(p)), writable: true, configurable: true });

const routes: Array<[RegExp, (u: URL) => unknown, number]> = [];
export const fetched: string[] = [];
export const mockFetch = (re: RegExp, answer: unknown, status = 200) =>
  routes.push([re, typeof answer === 'function' ? (answer as (u: URL) => unknown) : () => answer, status]);
Object.defineProperty(globalThis, 'fetch', { configurable: true, writable: true, value: vi.fn(async (input: any) => {
  const u = new URL(typeof input === 'string' ? input : input.url, 'http://localhost');
  fetched.push(u.pathname + u.search);
  const r = [...routes].reverse().find(([re]) => re.test(u.pathname + u.search));
  if (!r) throw new Error(`no fetch mock for ${u.pathname}${u.search}`);
  const body = await r[1](u);
  return new Response(typeof body === 'string' ? body : JSON.stringify(body), { status: r[2] });
}) });

// jsdom gaps
Object.defineProperty(window, 'matchMedia', { writable: true, value: (q: string) => ({
  matches: false, media: q, onchange: null, addListener() {}, removeListener() {},
  addEventListener() {}, removeEventListener() {}, dispatchEvent: () => false }) });
class IO { observe() {} unobserve() {} disconnect() {} takeRecords() { return []; } root = null; rootMargin = ''; thresholds = []; }
(globalThis as any).IntersectionObserver = IO;

beforeEach(() => { handlers.clear(); calls.length = 0; routes.length = 0; fetched.length = 0; });
```

#### `src/qortal/search.test.ts`

```ts
import { describe, it, expect } from 'vitest';
import { searchUrl, searchQdn, findExact } from './search';
import { mockFetch, fetched } from '../test/setup';

describe('search', () => {
  it('always sends mode=ALL and a limit, and encodes names', () => {
    const u = new URL(searchUrl({ service: 'JSON', identifier: 'myapp_post_', prefix: true, name: 'My App+' }), 'http://x');
    expect(u.searchParams.get('mode')).toBe('ALL');
    expect(u.searchParams.get('limit')).toBe('20');
    expect(u.searchParams.getAll('name')).toEqual(['My App+']);     // survives the round trip
    expect(u.search).toContain('My+App%2B');
  });
  it('merges identical in-flight searches', async () => {
    mockFetch(/resources\/search/, [{ name: 'A', service: 'JSON', identifier: 'myapp_post_1' }]);
    await Promise.all([searchQdn({ service: 'JSON', identifier: 'x_' }), searchQdn({ service: 'JSON', identifier: 'x_' })]);
    expect(fetched.filter(f => f.startsWith('/arbitrary/resources/search')).length).toBe(1);
  });
  it('findExact ignores contains-matches', async () => {
    mockFetch(/resources\/search/, [{ name: 'Alice', service: 'JSON', identifier: 'myapp_post_12' }]);
    expect(await findExact('JSON', 'Alice', 'myapp_post_1')).toBeUndefined();
  });
});
```

### 18C. Zip script (`scripts/zip.sh`)

```bash
#!/usr/bin/env bash
# Build and zip dist/ CONTENTS (index.html at the zip root), no source maps or precompressed files.
set -euo pipefail
cd "$(dirname "$0")/.."
npm run build
test -f dist/index.html || { echo "dist/index.html missing" >&2; exit 1; }
if grep -rqE 'src="/assets/|href="/assets/' dist/index.html; then
  echo "Absolute /assets/ paths found: set base: \"\" in vite.config" >&2; exit 1
fi
if grep -rqE 'https?://(cdn|unpkg|fonts\.googleapis|fonts\.gstatic)' dist; then
  echo "Warning: external URLs found in dist (blocked by the Q-App CSP)" >&2
fi
rm -f release.zip
(cd dist && zip -r -q -X ../release.zip . -x '*.map' -x '*.gz' -x '*.br' -x '.DS_Store')
unzip -l release.zip | head -20
du -h release.zip      # APP cap is 50 MB; smaller loads faster from peers
```

Windows (PowerShell): `Compress-Archive -Path dist\* -DestinationPath release.zip -Force` (the `\*` keeps `index.html` at the root; delete `*.map` from `dist` first).

---

## 19. Reference

### 19.1 `qortalRequest` action catalogue (Hub 3.0.3 / GO 3.0)

Key: **C** = answered by Core's q-apps.js inside the frame (no dialog, no Hub timeout, works on public nodes). **H** = answered by Hub/GO. **Dlg** = approval dialog; **Dlg\*** = can be skipped by an "always" tick or a session grant; **Fee** = on-chain fee shown; **✓** = needs the "I have read this request" tick **in GO 3.0** (desktop Hub 3.0.3 asks for the tick only on QORT `SEND_COIN` and `SESSION_PERMISSIONS`; GO also on `SELL_NAME`, `BUY_NAME` and `SIGN_FOREIGN_FEES`). "Tx" = transaction JSON (`signature`, `type`, `fee`, `timestamp`, …). Unknown actions are never answered (they time out). Check what a host supports with `SHOW_ACTIONS` (H, returns an array of names; use `qortalRequestWithTimeout(…, 3000)`).

**Identity and accounts**

| Action | Key params | Returns | By | Dialog / notes |
|---|---|---|---|---|
| `GET_USER_ACCOUNT` | – | `{address, publicKey}` | H | Dlg\* "authenticate", per-app "always" tick; grants silent wallet reads/lists for the tab (read them only when a feature needs them; never publish or store them). Every failure, a decline included, arrives as a translated "unable to fetch user account" |
| `GET_PRIMARY_NAME` | `address` | name or `''` | H | no dialog |
| `GET_ACCOUNT_DATA` | `address` | `/addresses/{address}` | C | |
| `GET_ACCOUNT_NAMES` | `address` | `[{name, owner,…}]` | C | ignores limit/offset |
| `GET_BALANCE` | `address` | number (QORT) | C | `assetId` is broken: use `/assets/balances` |
| `IS_USING_PUBLIC_NODE` | – | boolean | H | |
| `WHICH_UI` | – | `'HUB_ELECTRON'` / `'HUB_WEB'` | H | GO says `HUB_WEB` |
| `OPEN_USER_LOOKUP` | `user` (name/address) | `true` | H | opens Hub's user panel |
| `SESSION_PERMISSIONS` | `permissions: string[]` | `true` | H | Dlg ✓; effective for `PUBLISH_QDN_RESOURCE`, `PUBLISH_MULTIPLE_QDN_RESOURCES`, `JOIN_GROUP`, wallet reads, `GET_LIST_ITEMS`, `NOTIFICATION_PERMISSION`, `LOCK_TAB` and a few others; invalid names throw. With a publish grant Hub stops confirming each publish: publish only from a user tap, no automatic retries, cap publishes per action, show the running cost, never request it at sign-in (7.6) |
| `SHOW_ACTIONS` | – | `string[]` | H | feature detection |

**Names** (all H, Dlg + Fee, return Tx, except the two C reads)

| Action | Params |
|---|---|
| `REGISTER_NAME` | `name`, `description?` |
| `UPDATE_NAME` | `oldName`, `newName`, `description?` |
| `SELL_NAME` / `CANCEL_SELL_NAME` / `BUY_NAME` | `nameForSale`, `salePrice` (sell only) |
| `SEARCH_NAMES` (C) | `query`, `prefix?`, `limit?`, `offset?`, `reverse?` (encode `+` yourself) |
| `GET_NAME_DATA` (C) | `name` → `/names/{name}` |

**QDN read** (C)

| Action | Key params | Returns / notes |
|---|---|---|
| `SEARCH_QDN_RESOURCES` | `service`, `identifier`, `name`, `names[]`, `query`, `title`, `description`, `keywords[]`, `prefix`, `exactMatchNames`, `default`, `mode`, `minLevel`, `includeMetadata`, `includeStatus`, `nameListFilter`, `followedOnly`, `excludeBlocked`, `before`, `after`, `limit`, `offset`, `reverse` | row array; **values not URL-encoded**; mode defaults to LATEST; no limit = all |
| `LIST_QDN_RESOURCES` | `service`, `name`, `identifier`, `default`, `includeStatus`, `includeMetadata`, `nameListFilter`, `followedOnly`, `excludeBlocked`, `limit`, `offset`, `reverse` | ordered by name |
| `FETCH_QDN_RESOURCE` | `service`, `name`, `identifier?`, `filepath?`, `rebuild?`, `encoding?: 'base64'` | body; JSON auto-parsed; blocks while fetching from peers; 60 s |
| `GET_QDN_RESOURCE_STATUS` | `service`, `name`, `identifier?`, `build?` | `{status, localChunkCount, totalChunkCount, percentLoaded}`; with `build:true` use a long timeout |
| `GET_QDN_RESOURCE_PROPERTIES` | `service`, `name`, `identifier` | size, mime type, filename |
| `GET_QDN_RESOURCE_METADATA` | `service`, `name`, `identifier` | title, description, tags, files |
| `GET_QDN_RESOURCE_URL` | `service`, `name`, `identifier?`, `path?` | `/arbitrary/…` (raw name; extra status request); rejects if not published |

**QDN write and hosting** (H)

| Action | Key params | Returns | Dialog / notes |
|---|---|---|---|
| `PUBLISH_QDN_RESOURCE` | `service`; one of `data64`/`base64`/`file`/`blob`; `name?`, `identifier?` (`default`), `filename?`, `title?`, `description?`, `category?`, `tags[]`/`tag1..5`, `encrypt?`+`publicKeys[]`, `isMultiFileZip?`, `appFee?`+`appFeeRecipient?` | Tx | Dlg\* Fee; 1 h; `PUBLISH_STATUS` progress; does **not** check that `encrypt` matches a `_PRIVATE` service (7.11) |
| `PUBLISH_MULTIPLE_QDN_RESOURCES` | `resources[]` (each as above, plus `disableEncrypt?`), `encrypt?`, `publicKeys?`, `appFee?` | Tx[] | one Dlg\* Fee × n; Hub n × 30 min; partial failure → `error.unsuccessfulPublishes`; refuses `encrypt` on a public service and plain data on a `_PRIVATE` one; GO only: Cancel → `error.cancelled` |
| `GET_HOSTED_DATA` | `limit?`, `offset?`, `query?` | hosted list | Dlg; local node only |
| `DELETE_HOSTED_DATA` | `hostedData: [{service,name,identifier}]` | `{deletedCount, failedCount, failures}` | Dlg; local node only; removes **this node's copy** only |

**Encryption** (H, no dialog, base64 results)

| Action | Key params | Notes |
|---|---|---|
| `ENCRYPT_DATA` | `data64`/`base64`/`file`/`blob`, `publicKeys[]` | sender can decrypt too |
| `DECRYPT_DATA` | `encryptedData`, `publicKey?` | `publicKey` for the old single-recipient format; doesn't decrypt chat |
| `ENCRYPT_QORTAL_GROUP_DATA` / `DECRYPT_QORTAL_GROUP_DATA` | `groupId`, `data64`/`base64`, `isAdmins?`, `refreshCache?` | group's shared key |
| `ENCRYPT_DATA_WITH_SHARING_KEY` / `DECRYPT_DATA_WITH_SHARING_KEY` | data + `publicKeys[]` / `encryptedData`, `key` | shareable content key |
| `DECRYPT_AESGCM` | `encryptedData`, `iv`, `senderPublicKey` | |

**Payments, wallets, trades** (H unless noted)

| Action | Key params | Returns | Dialog / notes |
|---|---|---|---|
| `SEND_COIN` | `coin`, `amount` (decimal string), `destinationAddress`/`recipient`, `fee?` (foreign) | Tx (QORT) | Dlg ✓ Fee; non-QORT refused on public node |
| `TRANSFER_ASSET` | `assetId`, `amount`, `recipient` | Tx | Dlg ✓ |
| `MULTI_ASSET_PAYMENT_WITH_PRIVATE_DATA` | `assetId`, `payments[{recipient, amount, arbitraryTxs?}]`, `additionalArbitraryTxsWithoutPayment?` | signatures | Dlg ✓; 30 s Hub limit |
| `GET_USER_WALLET` | `coin` | `{address, publickey}` (lower-case k) | Dlg\* |
| `GET_WALLET_BALANCE` | `coin` | QORT number; foreign string | Dlg\* |
| `GET_USER_WALLET_INFO`, `GET_USER_WALLET_TRANSACTIONS` | `coin` | | Dlg\* |
| `GET_CROSSCHAIN_SERVER_INFO`, `GET_FOREIGN_FEE` (`type`), `GET_TX_ACTIVITY_SUMMARY`, `GET_ARRR_SYNC_STATUS` | `coin` | | |
| `UPDATE_FOREIGN_FEE`, `SET_CURRENT_FOREIGN_SERVER`, `ADD_FOREIGN_SERVER`, `REMOVE_FOREIGN_SERVER`, `START_CROSSCHAIN_SERVER`, `SIGN_FOREIGN_FEES`, `GET_SERVER_CONNECTION_HISTORY` | `coin`, … | | local node only (most) |
| `GET_PRICE` (C) | `blockchain`, `maxtrades?`, `inverse?` | number | |
| `CREATE_TRADE_BUY_ORDER` | `foreignBlockchain`, `crosschainAtInfo[]` | | Dlg ✓ |
| `CREATE_TRADE_SELL_ORDER` | `qortAmount`, `foreignBlockchain`, `foreignAmount` | | Dlg ✓ |
| `CANCEL_TRADE_SELL_ORDER` | `atAddress` | | Dlg |

**Groups, polls, chat** (H, Dlg + Fee, Tx unless noted)

| Action | Key params |
|---|---|
| `JOIN_GROUP` (Dlg\*) / `LEAVE_GROUP` | `groupId` |
| `CREATE_GROUP` | `groupName`, `type`, `approvalThreshold`, `minBlock`, `maxBlock`, `description?` |
| `UPDATE_GROUP` | `groupId`, `newOwner`, `type`, `approvalThreshold`, `minBlock`, `maxBlock`, `description?` (missing = empty), `txGroupId?` |
| `INVITE_TO_GROUP` | `groupId`, `inviteeAddress`, `inviteTime` (s, non-zero) |
| `CANCEL_GROUP_INVITE`, `KICK_FROM_GROUP`, `BAN_FROM_GROUP`, `CANCEL_GROUP_BAN`, `ADD_GROUP_ADMIN`, `REMOVE_GROUP_ADMIN` | `groupId`, `qortalAddress` (+ `reason?`, `banTime?`) |
| `REENCRYPT_GROUP_KEYS` | `groupId` (owner only) → `true` |
| `LIST_GROUPS` (C) | `limit?`, `offset?`, `reverse?` |
| `CREATE_POLL` | `pollName`, `pollDescription`, `pollOptions` (**one string "A, B, C" in an array**), `pollOwnerAddress` |
| `VOTE_ON_POLL` | `pollName`, `optionIndex` |
| `SEND_CHAT_MESSAGE` | `message`/`fullMessageObject`, `recipient`/`destinationAddress` or `groupId`, `chatReference?` (Dlg\*, proof-of-work instead of a normal fee, needs ≥ 4 QORT; can resolve `false`) |
| `SEARCH_CHAT_MESSAGES` (C) | `txGroupId` or `involving[2]`, `before`, `after`, `reference`, `chatReference`, `hasChatReference`, `encoding`, `limit`, `offset`, `reverse` |

**Lists, notifications** (H)

| Action | Key params | Notes |
|---|---|---|
| `GET_LIST_ITEMS` | `list_name` | Dlg\*; refused on public node |
| `ADD_LIST_ITEMS` | `list_name`, `items[]` | Dlg; refused on public node |
| `DELETE_LIST_ITEM` | `list_name`, `item` | Dlg; refused on public node |
| `NOTIFICATION_PERMISSION` | – | call each tab session before ADD |
| `NOTIFICATION_HAS_PERMISSION` | – | boolean |
| `NOTIFICATION_ADD` | `notifications[{notificationId, link, filters, message:{en,…}, image?}]` | |
| `NOTIFICATION_GET` / `NOTIFICATION_REMOVE` (`notificationIds?`) / `NOTIFICATION_MARK_SEEN` (`notificationIds`) | | |

**Navigation, tabs, files, media** (H unless noted)

| Action | Key params | Notes |
|---|---|---|
| `LINK_TO_QDN_RESOURCE` (C → H) | `service` (default WEBSITE), `name`, `identifier?`, `path?` | same app: frame reload; other app: new tab |
| `OPEN_NEW_TAB` | `qortalLink`, `navigateIfAlreadyOpen?` | qortal:// only |
| `LOCK_TAB` / `UNLOCK_TAB` | `lockMessage` (`LOCK_TAB` only, required) | stop the user closing the tab during long work |
| `CREATE_AND_COPY_EMBED_LINK` | `type: 'POLL'\|'IMAGE'\|'ATTACHMENT'`, `name`, … | copies `qortal://use-embed/…` |
| `SET_FULL_SCREEN` | `enabled` | **GO only**; unanswered in desktop Hub |
| `SAVE_FILE` | `filename` + (`blob`, `mimeType`) or `location{service,name,identifier}` | Dlg every time; decline = string |
| `SHOW_PDF_READER` | `blob` (type `application/pdf`) | |
| `PLAY_ENCRYPTED_MEDIA` | `mediaId`, `key`, `iv`, `location`, `totalSize?`, `mimeType?` | Electron Hub only |

**Node, blocks, transactions**

| Action | Key params | By |
|---|---|---|
| `GET_NODE_INFO` / `GET_NODE_STATUS` / `GET_DAY_SUMMARY` | – | H |
| `FETCH_BLOCK` | `signature` or `height`, `includeOnlineSignatures?` | C |
| `FETCH_BLOCK_RANGE` | `height`, `count?`, `reverse?`, `includeOnlineSignatures?` | C |
| `SEARCH_TRANSACTIONS` | `txType[]`, `address`, `confirmationStatus` (**required**), `startBlock`, `blockLimit`, `txGroupId`, `limit`, `offset`, `reverse` | C |
| `GET_AT` / `GET_AT_DATA` | `atAddress` | C |
| `LIST_ATS` | `codeHash58`, `isExecutable?`, `limit`, `offset`, `reverse` | C |
| `DEPLOY_AT` | `name`, `description`, `tags`, `creationBytes`, `amount`, `assetId`, `type` | H, Dlg |
| `SIGN_TRANSACTION` | `unsignedBytes` (base58), `process?` | H, Dlg ✓ (8.9) |
| `ADMIN_ACTION` | `type`, `value?` | H, Dlg; local node only; avoid in public apps |

Not available as actions: group approval, asset issue/orders, reward shares, plain multi-payment, decrypting chat DMs, "sign a message to prove identity", opening web URLs, deleting QDN data from the network.

Hub `develop` (unreleased as of 2026-10-29) adds e.g. `FILE_SAVE_OPEN/WRITE/FINISH/ABORT`, `SCREEN_CAPTURE_SELECT`, `PRIVATE_CHANNEL_*`, `RNS_*`, `MOQ_*`, `SIGN_QAPP_IDENTITY`. Parameters UNVERIFIED: use only behind `SHOW_ACTIONS` detection.

### 19.2 REST cheat sheet (same origin, GET unless noted)

| Endpoint | Returns / notes |
|---|---|
| `/arbitrary/resources/search?…` | rows (6.3) |
| `/arbitrary/resources/searchsimple?…` | one row per resource (like `mode=ALL`); exact, case-sensitive unless `prefix`/`caseInsensitive` |
| `/arbitrary/resources?service=&name=&identifier=` | exact list, by name |
| `/arbitrary/{service}/{name}/{identifier}[?async=true][&encoding=base64][&filepath=]` | data; blocks while fetching unless `async` |
| `/arbitrary/resource/status/{service}/{name}/{identifier}[?build=true]` | `{status, localChunkCount, totalChunkCount, percentLoaded}` |
| `/arbitrary/resource/properties/{service}/{name}/{identifier}` | size, mime, filename |
| `/arbitrary/metadata/{service}/{name}/{identifier}` | title, description, tags, category, `files` |
| `/arbitrary/THUMBNAIL/{name}/qortal_avatar?async=true` | avatar image |
| `/arbitrary/categories` | category enum |
| `/names/{name}` | `{name, owner, data, …}` (case-insensitive) |
| `/names/address/{address}?limit=&offset=` | `[{name, owner}]` |
| `/names/primary/{address}` | `{name?, owner}` |
| `/names/search?query=&prefix=&limit=` | name rows (character order) |
| `/addresses/{address}` | `{address, publicKey, level, blocksMinted, …}` |
| `/addresses/publickey/{address}` | **plain text** base58 |
| `/addresses/balance/{address}` | number |
| `/addresses/validate/{address}` | boolean |
| `/addresses/lastreference/{address}` | base58 |
| `/assets/info?assetId=` | `{assetId, name, isDivisible, …}` |
| `/assets/balances?address=&assetid=` | balances |
| `/transactions/search?txType=&address=&confirmationStatus=BOTH&limit=&offset=&reverse=` | confirmationStatus required |
| `/transactions/signature/{signature}` | one transaction (`blockHeight` once confirmed) |
| `/transactions/unconfirmed?creator=<publicKey>&limit=` | your pending txs |
| `/transactions/unitfee?txType=ARBITRARY` | fee in 1e-8 QORT (`1000000` = 0.01) |
| POST `/transactions/decode?ignoreValidityChecks=true` | decode unsigned bytes (read-only) |
| `/blocks/height`, `/blocks/byheight/{h}`, `/blocks/timestamp/{ms}` | block info |
| `/groups?limit=`, `/groups/{id}` | groups |
| `/groups/members/{id}?onlyAdmins=true` / `?limit=&offset=` | admins / members (ignore paged `isAdmin`) |
| `/groups/member/{address}` | an account's groups |
| `/polls/{name}`, `/polls/votes/{name}?onlyCounts=true` | polls |
| `/crosschain/price/{BLOCKCHAIN}?inverse=` | integer × 1e-8 |
| `/chat/messages?txGroupId=` or `?involving=a&involving=b` | chat (recent only) |
| `/admin/info` | `{buildVersion, isTestNet, …}` |
| `/apps/q-apps.js` | the injected script (read it to confirm behaviour) |
| websockets `/websockets/blocks`, `/websockets/chat/messages` | live updates |

Errors: JSON `{error: <code>, message}` with 400/404; bad enum values → plain text 400; missing required params can give an HTML 500.

### 19.3 Services (Core 6.1.9)

| Service | Id | Max size | Single file | Private | Notes |
|---|---|---|---|---|---|
| `ARBITRARY_DATA` | 100 | – | n | n | |
| `QCHAT_ATTACHMENT` / `_PRIVATE` | 120 / 121 | 1 MiB | y | n / y | limited extensions |
| `ATTACHMENT` / `_PRIVATE` | 130 / 131 | 50 MiB | y | n / y | |
| `FILE` / `_PRIVATE` | 140 / 141 | – | n (y before 6.1.9) | n / y | publish one file if older nodes must read it |
| `FILES` | 150 | – | n | n | |
| `CHAIN_DATA` | 160 | 239 B | y | n | |
| `WEBSITE` | 200 | – | n | n | needs root index file |
| `GIT_REPOSITORY` | 300 | – | n | n | |
| `IMAGE` / `_PRIVATE` | 400 / 401 | 10 MiB | n (y before 6.1.9) | n / y | |
| `THUMBNAIL` | 410 | 500 KiB | y | n | avatars: `qortal_avatar` |
| `QCHAT_IMAGE` | 420 | 500 KiB | y | n | |
| `VIDEO` / `_PRIVATE` | 500 / 501 | – | y | n / y | |
| `AUDIO` / `_PRIVATE` | 600 / 601 | – | y | n / y | |
| `QCHAT_AUDIO`, `QCHAT_VOICE`, `VOICE` / `VOICE_PRIVATE` | 610, 620, 630 / 631 | 10 MiB | y | n (631 y) | |
| `PODCAST` | 640 | – | y | n | |
| `BLOG` / `BLOG_POST` / `BLOG_COMMENT` | 700 / 777 / 778 | – / – / 500 KiB | n / y / y | n | |
| `DOCUMENT` / `_PRIVATE` | 800 / 801 | – | n (y before 6.1.9) | n / y | general-purpose JSON/text |
| `LIST`, `PLAYLIST` | 900, 910 | – | y | n | |
| **`APP`** | **1000** | **50 MiB** | n | n | Q-Apps |
| `METADATA` | 1100 | – | y | n | |
| `JSON` | 1110 | **25 KiB**, valid JSON | y | n | small public JSON |
| `GIF_REPOSITORY` | 1200 | 25 MiB | n | n | .gif only |
| `STORE`, `PRODUCT`, `OFFER`, `COUPON`, `CODE`, `PLUGIN`, `EXTENSION` | 1300, 1310, 1330, 1340, 1400, 1410, 1420 | – | y | n | |
| `GAME` / `ITEM` | 1500 / 1510 | – | n / y | n | |
| `NFT` | 1600 | – | y | n | |
| `DATABASE`, `SNAPSHOT` | 1700, 1710 | – | n | n | |
| `COMMENT` / `CHAIN_COMMENT` | 1800 / 1810 | 500 KiB / 239 B | y | n | |
| `MAIL` / `MAIL_PRIVATE` | 1900 / 1901 | 1 MiB / 5 MiB | y | n / y | |
| `MESSAGE` / `MESSAGE_PRIVATE` | 1910 / 1911 | 1 MiB | y | n / y | |

- Single-file services are always validated. Private single-file services must hold encrypted data; public ones must not.
- Core's encryption check (`DATA_NOT_ENCRYPTED` / `DATA_ENCRYPTED`) runs only for services with "Single file = y". `FILE`, `IMAGE`, `DOCUMENT` and their `_PRIVATE` forms are not checked on 6.1.9: encrypt before publishing to them yourself (7.11).
- Data ≤ 256 bytes (after encryption) is stored on chain (`RAW_DATA`).
- Transaction JSON shows the service **id**; searches show the **name**. Service names are case-sensitive in queries.

**Which service should I use?**

| Data | Service |
|---|---|
| Small public JSON records (≤ 25 KB) | `JSON` |
| Larger public JSON/text/HTML | `DOCUMENT` |
| Private records | `DOCUMENT_PRIVATE` (or `MAIL_PRIVATE` for mail-like) with encryption |
| Images / thumbnails | `IMAGE` / `THUMBNAIL` |
| Video / audio | `VIDEO` / `AUDIO` |
| Arbitrary files | `FILE` (or `ATTACHMENT` ≤ 50 MB) |
| Comments | `COMMENT` / `BLOG_COMMENT` (≤ 500 KB) |
| Your app | `APP` |
| A static site | `WEBSITE` |

### 19.4 Search parameters (`/arbitrary/resources/search`), compact

| Param | Type | Default | Gotcha |
|---|---|---|---|
| `service` | enum | – | always send; case-sensitive |
| `mode` | `ALL`/`LATEST` | `LATEST` | LATEST = 1 per (name, service) |
| `identifier` | string | – | contains, case-insensitive |
| `prefix` | bool | false | starts-with for identifier/name/title/description/query |
| `name` (repeat) | string | – | contains unless `exactmatchnames=true` |
| `exactmatchnames` | bool | false | exact, case-insensitive |
| `query` | string | – | contains in name/identifier/title/description |
| `title`, `description` | string | – | needs metadata on node |
| `keywords` (repeat) | string | – | in description |
| `default` | bool | – | not with `identifier` |
| `minlevel` | int | – | publisher level |
| `namefilter` | list name | – | node list; empty → `[]` |
| `followedonly`, `excludeblocked` | bool | false | node lists |
| `includemetadata`, `includestatus` | bool | false | status may be stale |
| `before`, `after` | ms | – | on `created`, strict |
| `limit` | int | **all** | always send |
| `offset` | int | 0 | |
| `reverse` | bool | false (oldest first) | `true` for newest first |

`SEARCH_QDN_RESOURCES` uses camelCase for some: `exactMatchNames`, `includeMetadata`, `includeStatus`, `minLevel`, `nameListFilter`, `followedOnly`, `excludeBlocked`, plus `names[]`.

### 19.5 Categories (`category` metadata)

ART, AUTOMOTIVE, BEAUTY, BOOKS, BUSINESS, COMMUNICATIONS, CRYPTOCURRENCY, CULTURE, DATING, DESIGN, ENTERTAINMENT, EVENTS, FAITH, FASHION, FINANCE, FOOD, GAMING, GEOGRAPHY, HEALTH, HISTORY, HOME, KNOWLEDGE, LANGUAGE, LIFESTYLE, MANUFACTURING, MAPS, MUSIC, NEWS, OTHER, PETS, PHILOSOPHY, PHOTOGRAPHY, POLITICS, PRODUCE, PRODUCTIVITY, PSYCHOLOGY, QORTAL, SCIENCE, SELF_CARE, SELF_SUFFICIENCY, SHOPPING, SOCIAL, SOFTWARE, SPIRITUALITY, SPORTS, STORYTELLING, TECHNOLOGY, TOOLS, TRAVEL, UNCATEGORIZED, VIDEO, WEATHER.

### 19.6 Key constants

| Fact | Value |
|---|---|
| Identifier max | 64 bytes UTF-8 |
| Name | 3-40 bytes UTF-8, normalised |
| Title / description | 80 / 240 bytes (truncated) |
| Tags | ≤ 5, each 1-20 chars (longer dropped) |
| Publish fee | 0.01 QORT per resource |
| Name registration | 1.25 QORT |
| Unconfirmed txs per account | 25 (26th: `TOO_MANY_UNCONFIRMED`) |
| Tx timestamp window | 24 h back, 30 min ahead |
| Block target | ~60 s |
| Hub request timeout | 30 s (publish 1 h; multi n × 30 min; SAVE_FILE none) |
| Hub dialog lifetime | 60 s |
| `NAVIGATE_TO_PATH` reply window | ~200 ms |
| q-apps.js concurrency | 30 in flight; no request dedupe |
| Hub publish cap | 2 GB; 500 MB on public node |
| Core API port | 12391 (mainnet); dev proxy 12393 |
| Chunk size | 512 KiB |

---

## 20. Compact system prompt

Copy this block into the system prompt (or the top of the conversation) of any LLM that will write Qortal Q-Apps.

```text
You build Qortal Q-Apps (service APP) and QDN websites (service WEBSITE) that must work first time in
Qortal Hub (desktop, Electron) and Qortal GO (Android WebView, usually on a public node).

Before coding, ask (with defaults): the registered name to publish under; single HTML file or Vite+React+TS;
features; what data is stored on QDN, its shape and who writes it; private data?; money?; must it share data
with an existing app? Then state services, identifier prefixes, JSON shapes, screens and costs, then code.

Hard rules:
1. Bundle everything; CSP allows only the node ('self'), no CDNs/remote fonts/images/APIs. No service workers.
2. Vite base "" (relative ./assets). Never base "/". Don't add <base>. index.html at the zip root. Publish under
   the default identifier. No manifest.json needed.
3. Host check: `typeof qortalRequest === 'function'` (global const, not window.qortalRequest). Outside Hub install a
   rejecting stub; never crash.
4. Inline boot script in <head>: theme from ?theme= → remembered (app-prefixed localStorage) → OS; a reload without
   ?theme= gets _qdnTheme "light". Listen for postMessage THEME_CHANGED / LANGUAGE_CHANGED from ancestor frames.
5. Height from window.innerHeight (CSS var), never 100vh. Phone layout < 600 px wide. overflow-x: clip on #root only.
   44 px targets, nothing hover-only, safe-area insets, keyboard-aware dialogs.
6. Router basename = _qdnBase ("" in Dev Mode), tolerant of "%2B" vs "+" and of non-ASCII names (Template B
   routerBase.ts). APP: path routes. WEBSITE: hash routes.
7. On message {action:'NAVIGATE_TO_PATH', path}: navigate and immediately
   window.parent.postMessage({action:'NAVIGATION_SUCCESS', path}, '*'). Never window.history/navigate(-1) for Back;
   keep an in-app back stack.
8. Reads: fetch('/arbitrary/resources/search?…') built with URLSearchParams: always service, mode=ALL, limit (≈20),
   offset, reverse=true; identifier+prefix=true; names with exactmatchnames=true; then compare identifier and name
   exactly in code (matching is contains, case-insensitive). prefix=true makes query/title/description starts-with
   too: for text search send the identifier without prefix plus query, and filter in code. Merge identical
   in-flight requests; cache 1-3 min; never cache failures. Never limit 0 or missing.
9. q-apps.js does not URL-encode parameters: for user-supplied values call REST yourself with encodeURIComponent.
10. Bodies: /arbitrary/<service>/<encoded name>/<encoded id>?async=true, ≤5 in parallel, only near the viewport;
    404 = still fetching → "Fetching from peers…", retry 2/4/8/16 s, then "Not available right now" + Retry.
    Big files: status endpoint with build=true. Tombstones ("D" or {deleted:true}) are hidden.
11. Trust the search row (name, identifier, created), never the body; validate every field; error boundaries.
12. Identifiers: app_type_<base36 time><random>, [a-z0-9_-], ≤ 64 bytes. New data kinds get new prefixes.
13. UTF-8 safe base64 (TextEncoder), never btoa(JSON.stringify()). No top-level "error" key in documents.
14. Publishing: needs a registered name (check first), 0.01 QORT per resource (show it), explicit user action only,
    never automatic. Title ≤ 80 B, description ≤ 240 B, ≤ 5 tags ≤ 20 chars. Metadata is public.
15. > 20 resources: PUBLISH_MULTIPLE in batches ≤ 20, wait for /transactions/unconfirmed room (25 max), index last,
    qortalRequestWithTimeout(req, n*30min); retry only error.unsuccessfulPublishes with the same identifiers.
16. Errors: normalise ({error:number|string, message} | string | {error:{cancelled:true}}); a decline (12 languages:
    /declin|cancel|rechaz|refus|ablehn|abgelehnt|отклон|拒否|拒绝|kieltäyt|keeldus|rifiut|recus|رفض/i) is a quiet cancel.
    Hub reports every GET_USER_ACCOUNT failure, a decline included, as "unable to fetch user account" (translated):
    any non-timeout rejection there means "not signed in", quietly.
17. Timeouts: Hub answers dialog actions after 30 s but the dialog lives 60 s and a late Accept still runs.
    After a timeout wait ~35 s and look on QDN/chain before offering Retry. Never publish or pay twice.
18. Money: only on an explicit tap showing amount and recipient; validate balance/assets/unconfirmed first; save
    pending state before the request; verify payments on chain; foreign coins need a local node.
19. *_PRIVATE services need encryption (encrypt:true + publicKeys, or ENCRYPT_DATA); public services must not be
    encrypted. Core doesn't check FILE/IMAGE/DOCUMENT(_PRIVATE), so always encrypt before a _PRIVATE publish.
    Decrypt with DECRYPT_DATA {encryptedData}. Cache decrypted data per viewer address.
20. Deadlines use block time, not created (publisher-signed, up to 24 h old).
21. Public nodes refuse lists, hosted data, foreign coins: ask IS_USING_PUBLIC_NODE once and degrade quietly (the
    refusal text is translated). localStorage is shared by all Q-Apps on a node, and is per node: prefix keys, no
    secrets. Sanitise HTML with DOMPurify and DOM post-processing; prefer textContent.
22. http(s) links don't open (blocked, no popups): copy to clipboard. qortal:// links: app name raw ("+" literal,
    spaces %20), each path segment encodeURIComponent. Open other apps with OPEN_NEW_TAB.
23. Avatars/images: build /arbitrary/THUMBNAIL/<encoded name>/qortal_avatar?async=true, lazy via
    IntersectionObserver, retry with backoff; NOT_PUBLISHED status = no avatar. Never <img src="qortal://">.
24. PDFs: SHOW_PDF_READER {blob type application/pdf}; files: SAVE_FILE {filename, blob, mimeType} or {location}
    in GO/large files; decline = quiet.
25. Poll only while visible, with backoff; lazy-load heavy screens/libraries; reload once on chunk-load errors.
26. Feature-detect rare actions with SHOW_ACTIONS (short timeout); unknown actions never answer.

Deliver: the code, the data contract (services, identifiers, JSON shapes, costs), and the steps to test in Hub
Dev Mode (Server with `vite dev`, reload at "/"; or Zip) and to publish (zip with index.html at root, service APP).
```

---

## Appendix: open points (UNVERIFIED)

- How long a publish takes to become visible on other nodes in practice (plan for 1-2 blocks; metadata later).
- Whether a hidden Hub tab reports `document.visibilityState === 'hidden'`.
- `blob:` Web Workers and WebAssembly inside workers under the CSP.
- Hub `develop`'s isolated `<webview>` app model (unreleased): message routing and `File`/`Blob` transfer may change; `fromHost()` assumes frames.
- Parameters of the unreleased `develop` actions (19.1).
- The 60 s dialog lifetime is in source; the "late Accept still applies" behaviour is from source reading plus field observation.
- GET_USER_ACCOUNT hanging while Hub is locked (reported, not traced).
- How long a cold `/arbitrary/...` read blocks before "Data unavailable" (5 attempts by default; one live cold fetch succeeded in 5.7 s).
- The in-app Back history effect in 9.4 is traced in q-apps.js and Hub source, not yet observed in a running Hub.
- The dev-proxy `/assets/*` 404 was observed on 6.1.9; the mechanism (Core's `/assets` API route registered alongside the proxy) is from source; other API-root first segments likely behave the same.
- Range requests with an over-long end and missing `Content-Range` exposure (observed).
- The pdf.js lazily loaded worker failure in Hub (observed).
- Whether Core unwraps a zip whose files sit in a top-level folder: always put `index.html` at the root.
- Path case-sensitivity inside the zip (likely case-sensitive: treat it as such).
- The `pollOptions` behaviour is from Hub 3.0.3/GO source; re-check on newer Hub.
