# chatperch — A chat button for any site, from one line

**Paid tool: $99 once (plus sales tax where it applies), then a private GitHub repository you're invited to, with every update. Buy: https://dominares.org/tools/chatperch**

Commercial license v1.3 — see https://dominares.org/license · Support: support@dominares.org

chatperch draws a chat button on someone else's website from one pasted line, opens your chat page in a sandboxed frame, serves the headers that page needs, and checks that the line is installed and working.

## Get it

Buy chatperch at https://dominares.org/tools/chatperch: one payment of $99, plus sales tax where it applies, and every future update. After checkout you are invited to a private GitHub repository with the source and every version published to it. The repository includes the Python package and the chatperch command, the button script and the frame client, a Playwright walk, and the full test suite.

Requirements: Python 3.10 or newer, no runtime dependencies. Node and Playwright only for the optional browser walk.

This page documents version 1.0.1. Install commands and file paths in the documentation below run inside the private repository you are invited to; the source is not in this repository.

## Documentation

### What it does

You run a chat page. A site owner (you, or your customer) pastes one line into their own website:

```html
<script src="https://chat.example.com/chatperch.js" async></script>
```

That draws a button in the corner of their page. A click opens your chat page in a sandboxed frame: a panel above the button on a laptop, the whole screen on a phone. It runs on someone else's page, so it follows rules written from that:

- **It never breaks the host page.** ES5, one function, every step guarded.
- **It needs almost nothing from the site's security policy.** Styles go through the CSSOM inside a shadow root, never a `<style>` element or style attribute, so a site with a strict Content-Security-Policy needs only `script-src` and `frame-src` for your chat's address (plus `connect-src` if you turn on the load ping).
- **One button per page.** A second copy of the line does nothing. By default the line also does nothing on the chat's own origin.
- **Nothing about the conversation crosses.** The page and the frame exchange three words (`ready`, `close`, `focus`) as `{chatperch: "chat", type}` messages, and each side checks the other's window and origin.

The pieces, all standard library, all usable from Python or the command line:

| Command | Python | What it does |
|---|---|---|
| `chatperch script` | `bubble_config`, `render_bubble_script` | Writes the button script with your settings in it. Host the file anywhere that serves JavaScript. |
| `chatperch snippet` | `snippet_for` | Prints the one line to paste. |
| `chatperch frame-client` | `frame_client_script` | The script your chat page runs so it can close itself (Escape, a close button) and take focus when the button opens it. Recommended, not required: without it the chat still opens and, on a phone, the button draws its own close control over the full-screen chat, but Escape works only while keyboard focus is on the embedding page, and focus does not move into your input on cue. |
| `chatperch frame-headers` | `frame_headers`, `build_frame_csp` | The response headers for your chat page: an enforced CSP that allows each inline script by its sha256 hash, `frame-ancestors` open (or narrowed to the sites you name), no `X-Frame-Options`, never cached. `Strict-Transport-Security` is sent only with `--hsts` / `hsts=True`, because it binds your whole domain to HTTPS for a year. |
| `chatperch check` | `check_install` | Opens a site and says whether the line is there and working. |
| `chatperch serve` | `make_app`, `SeenStore` | Optional adapter: serves the script and receives the button's load ping. |
| `chatperch walk` | | Drives a real browser through a page that carries the line. |
| `chatperch demo` | `run_demo` | The whole loop on your machine. |

#### Setting it up

```bash
chatperch script --frame-url https://chat.example.com/widget --org "Rivera Plumbing" --color "#0f766e" -o chatperch.js
chatperch snippet --script-url https://chat.example.com/chatperch.js
```

`--frame-url` is any chat page you can frame. To let it close itself (Escape from inside the chat, its own close button) and take focus on cue, include `chatperch frame-client` in that page; without it, a phone still gets a close control from the button. The button adds `#parent=<the embedding site's origin>` to the address it frames; the client reads it and talks only to that origin.

The tag accepts `data-position="left"` (default is the right corner; `--position left` sets it for every site).

#### Checking an install

```bash
chatperch check www.example.com --script-url https://chat.example.com/chatperch.js
```

It prints one plain sentence and exits 0 for `working` or `found`, 1 otherwise. The answers:

| Result | Meaning |
|---|---|
| `working` | The line is on the page (or the button pinged recently, which also covers a tag manager) and the button loaded there in the last 24 hours. |
| `found` | The line is in the page's HTML, but no recent load was seen. |
| `missing` | The page opened and the line is not in it. |
| `different_address` | The page has a button script with the same file name at another address. |
| `unreachable` | The page did not open. |
| `refused` | The address is not a public website. |
| `invalid` | Not a web address. |

The line is matched by address, not by spelling: attribute order and case, a query string, a relative or protocol-relative `src`, and an explicit default port all match, and `http` versus `https` is not told apart. Only a script that runs counts: a `<script src>` that is a different file name (`chatperch.jsx`), is inside an HTML comment, `<noscript>`, `<textarea>` or `<template>`, or has a non-JavaScript `type` does not, and neither does the address shown as text or in a link. An inline `<script>` whose text names the script's address as a whole address counts as found (a mention in a JavaScript comment counts too).

Two kinds of evidence, because either alone misleads. The HTML scan proves the line was pasted. The button's own ping proves it ran, which also covers scripts a tag manager adds later. The ping is optional: run `chatperch serve --frame-url ... --seen-file seen.json`, pass the same file to `chatperch check --seen-file`, or use `SeenStore(path).last_seen` as `seen_at=` in Python.

Every page fetch is bounded: 5 redirects, 2 MB, 8 seconds in all. A watchdog cuts the connection at the deadline, so a server that trickles its response headers or its body cannot hold the check open; the name lookup before the connection is not under that cap. Each address the name resolved to is tried in turn within the same 8 seconds. Every hop is checked against a public-address guard, and the connection goes to the address that was just checked, so a name cannot resolve one way for the check and another for the connection. Loopback, private, link-local, carrier-grade NAT, multicast and reserved ranges, and IPv4-mapped forms of them, are refused. `--allow-private` turns the guard off for a site on your own machine.

#### Walking it in a real browser

```bash
npm install playwright && npx playwright install chromium webkit
chatperch walk --script-url https://chat.example.com/chatperch.js
```

It serves a stand-in customer site on `127.0.0.1` that carries your line twice under a strict Content-Security-Policy that allows only your chat's address, then checks in Chromium and WebKit, at phone and laptop sizes: one button, an accessible name, its label, its place in the corner and a 44px height, the frame opening your chat page, the frame filling a phone and sitting above the button on a laptop, no policy violation on the page, close by the chat's own button (when your page runs the frame client) and by Escape, and focus back on the button. Run it from a folder where `npm install playwright` was done. It exits 1 if any check fails, 2 on bad usage.

### Options worth knowing

| Option | Where | Default |
|---|---|---|
| `--label` | `script` | `Ask us` |
| `--color` | `script` | `#1f2937` (readable text colour is picked for you; a value that is not `#rgb` or `#rrggbb` falls back to the default) |
| `--position left\|right` | `script`, `snippet` | `right` |
| `--seen-url` | `script`, `serve` | none (no ping); `serve --seen-file` derives it from the chat's origin |
| `--allow-own-origin` | `script`, `serve` | off |
| `--frame-ancestors` | `frame-headers` | `*` |
| `--hsts` | `frame-headers` | off |
| `--site-port` | `walk`, `demo` | any free port |
| `--allow-private` | `check` | off |

### Limits

- **The load ping is evidence, not proof.** Anyone can POST to a public endpoint. A request whose `Origin` header differs from the page it names is ignored, but a script outside a browser can claim any site. A site whose CSP has no `connect-src` for your address blocks the ping; there the HTML scan is the evidence and the answer stays `found`.
- **A page that adds the line only in the browser looks `missing`** to the HTML scan (a tag manager, a single-page app that injects scripts), unless the ping has been seen. The check opens the page's HTML; it does not run it.
- **The ping store keeps the 1000 most recent sites.** There is no login on the ping, so someone who posts 1000 made-up origins pushes older real ones out (the next real page load puts a site back). If that matters, put your own filter in front of the ping endpoint.
- **The check reads one page.** The line on another page of the site is not found.
- **The sandbox is `allow-scripts allow-same-origin allow-forms allow-popups allow-popups-to-escape-sandbox`.** Popups the chat opens (a link, a policy page) are not sandboxed. Your chat page must be served from a different origin than the sites that embed it; `allow-same-origin` plus `allow-scripts` on the same origin as the embedding page would remove the sandbox. chatperch cannot enforce where you host.
- **The chat page itself is yours.** chatperch frames any URL and does not provide a chat, a model, or storage. `chatperch demo` includes a stand-in page with canned replies so you can see the button work.
- **`walk` is run by hand.** It needs node and Playwright with browsers installed, and it is not part of `pip install`. The test suite runs it only when Playwright is available (set `CHATPERCH_PLAYWRIGHT_DIR` to a folder where `require('playwright')` works).
- **Fetching is by name through the system resolver.** With a custom `is_safe` function passed to `fetch_page`, the connection is made by name and the address-pinning protection does not apply.
- Python 3.10 or newer. No runtime dependencies.

### Running the tests

```bash
pytest
```

The tests run the button script under node against a stub page, so node must be installed for those (they are skipped without it). Set `CHATPERCH_TEST_PORTS=8820-8829` to make the tests' local servers use a port range instead of any free port.

---

Docs mirror of the private repository Dominares-Tools/chatperch; updated with each release.
