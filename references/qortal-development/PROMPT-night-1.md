Local overnight session for Q-Shop+, in ultracode: use workflows and parallel agents where they help, following "Working with parallel agents" in the qplus-app skill. Simon is asleep: don't stop to ask questions. Make careful, sensible calls and list open questions in the brief's Follow-ups.

Goal: take Q-Shop+ from its round-1 checkpoint (29 Sep) to the Q-Share+ standard: rounds 1–3 of the qplus-app skill tonight, plus as much of round 4 (the read-only Hub check) as time allows. Q-Shop is a marketplace where people pay real coins, so the money code gets the most care of all. Q-Shop should end up a world-class Qortal marketplace combining premium visual design, striking product imagery, refined typography, effortless navigation, comprehensive search, smart filters, beautiful storefronts, seamless checkout, trusted reviews, and intuitive buyer and seller dashboards.

WHERE
- Work ONLY in /home/simon-james/Desktop/Q-Apps+-QShop. It is a git worktree on branch q-shop-plus/1.0.0: the old pass-1 work with current main merged in and the theme kit synced. Build passes and 24 tests pass.
- Never touch the other Q-Apps+ folders; other chats are working in them.
- Start with: git fetch origin main && git merge origin/main, then cd "apps/Q-Shop+" && npm ci, then a green build and tests. There is no lint script yet; add ESLint 9 with the Q-Share+ config early, and fix what it finds.

READ FIRST
- CLAUDE.md and the qplus-app skill: its "Done means" checklist, rounds 1–4, the parts to reuse, and the brief template.
- docs/apps/Q-Shop+.md: the partial audit, the plan, and the open questions.
- docs/DESIGN.md (Mobile), docs/QORTAL.md (Hub & GO pitfalls; check every one), and docs/RELEASE.md.
- The worked examples:
  - docs/apps/Q-Share+.md;
  - docs/apps/Q-Mail+.md, which also covers paying fees and Hub dialogs;
  - docs/apps/Q-Mintership+.md, for status-tailored screens and optimistic UI.
  Reuse their parts (phone shell, NameSwitcher/NameAvatar, qdnSearch, hubErrors, qortalLinks, settings sync, publish batching in Q-Share+'s publishBatches.ts) by copying and adapting. Never import across apps/.

MONEY AND SAFETY (non-negotiable)
- Never send a payment or publish anything on mainnet: no SEND_COIN, PUBLISH_QDN_RESOURCE, PUBLISH_MULTIPLE or list writes from any Hub, test or script. Tests mock every write.
- Checkout, SEND_COIN (QORT, ARRR and any other coin), order encryption (DOCUMENT_PRIVATE / DECRYPT_DATA) and price conversion are money code.
  - Before changing any of it, write tests that pin today's behaviour: amounts, recipients, coin, the order JSON, encryption keys, and identifiers.
  - Keep the flow identical unless a change is clearly a fix. Record every money-code change in the brief with its reason.
  - Never round or convert differently without a test that proves it's right.
- Data compatibility (CLAUDE.md rule 1): stores, products, catalogues, orders and reviews made in Q-Shop+ must open in the original Q-Shop, and the reverse. Finish the data contract in the brief before touching any data code.
- Q-Apps run sandboxed: no web2 calls, no CDNs. Coin prices come only from the node (/crosschain/price…).
- Core allows at most 25 unconfirmed transactions per account. Bulk publishing (many products) must go in batches that wait for a block; Q-Share+'s publishBatches.ts is the pattern.

TONIGHT'S SCOPE
Round 1, finish it:
- the full audit in the brief: architecture, data contract, Qortal call inventory with the first-load count, performance, ranked UX problems, bugs, and missing features;
- the plan;
- one module for every + specific (docs/RELEASE.md);
- the efficiency fixes: no limit:0, page every list, deduped and cached searches, one shared price fetch, identifier+prefix instead of query=, lazy images, code-splitting.

Round 2, features and phone:
- replace the ~100 hard-coded colours and ~80 mode ternaries with theme values, so all four themes look right;
- the phone shell: a bottom nav (e.g. Shops, Orders, Cart, Settings), sheets, full-screen dialogs, a checkout that stays usable above the keyboard, 44 px targets, the frame height and a compact landscape layout;
- loading, empty and error states everywhere;
- the obvious missing features, most valuable first:
  - store and product search with filters and sort (price, newest, rating);
  - a cart that survives a reload;
  - clear order status for buyers and sellers, with counts;
  - a seller dashboard (orders to handle, products, reviews);
  - prices shown in the buyer's chosen coin;
  - product image galleries;
  - store and product share links (qortal://APP/Q-Shop+/…, literal +).
- Bulk product publishing goes in batches.
- e2e/screens.config.mjs with fixtures for stores, products, cart, checkout, orders (buyer and seller) and empty states.
- support for taking payment and paying in other q-assets (https://gitea.qortal.link/crowetic/Q-Assets)
- adding various sorting features like by activity, biggest and so on
- add search bar and browse capability
- overall shopping cart

Round 3, quality:
- 0 axe violations and 4.5:1 contrast in all four themes (Hub 2.0 = "Q-Shop Classic");
- Settings sync;
- the changelog dialog;
- the README + section;
- a review workflow over the whole diff: correctness, money-code safety, data compatibility, efficiency, mobile/UX, accessibility and security (user HTML in product descriptions: DOMPurify 3.4, links built on the DOM). Adversarially verify each finding, then fix the confirmed ones, one commit each.

Round 4, if time allows, read-only:
- Use your own test Hub: --remote-debugging-port=9223, dev server on 5174, HUB_CDP_PORT=9223 (docs/HUB-TESTING.md).
- Walk real stores and products at 1440, 700, 390x844 touch, 360x740 touch and 844x390 touch, in four themes and Hub light/dark.
- Count the calls, and compare with the original Q-Shop.
- Never press checkout, pay, publish or review.
- Close the test Hub after.

DECISIONS ALREADY MADE
- Version: plain 1.0.0 (unpublished). No -plus suffix; replace the old 1.0.0-plus.1.
- The app's own share links point at Q-Shop+.
- Keep the shared blocked-names list (blockedNames_q-blog) so the original app keeps working.

PARALLEL WORK
- Do the audit, the money-code tests and the data layer yourself first.
- Then fan out to agents on disjoint files, in worktrees on local branches q-shop-plus/feat-<slug>. Integrate one at a time with build + lint + tests, then push.
- Only one agent drives the test Hub.

GIT
- Only q-shop-plus/1.0.0 is pushed, after every commit.
- One logical change per commit ("Q-Shop+: …"; "Repo: …" for shared/).
- No Claude attribution lines.
- Rebuild release/Q-Shop+.zip (scripts/build-zip.sh Q-Shop+) as soon as a change passes its checks.
- Open the PR early: gh pr create --base main --title "Q-Shop+ 1.0.0: React 19.3 / MUI 9.4 + redesign". It supersedes #8; say so in the body. Keep its description current.

USAGE GUARD
Check usage (get_usage) between workflows. If weekly all-models passes 85%, stop starting workflows and finish alone.

FINISH
- Fill in the brief using the skill's template, with Done per round (numbers, before/after tables, the screenshot matrix) and Follow-ups (numbered questions for Simon; what round 4 still needs; what only a phone can check).
- Add the changelog entry.
- Run scripts/build-zip.sh Q-Shop+.
- Put a short morning summary at the top of the PR description.
