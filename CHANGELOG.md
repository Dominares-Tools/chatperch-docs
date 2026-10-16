# Changelog

## 1.0.1

Fixes from review.

- `fetch_page`: the 8-second cap now covers the response headers and the connection, not only the body. A watchdog cuts the connection at the deadline, so a server that sends its headers one line at a time can no longer hold a check open. Every address the name resolved to is now tried in turn (it used to dial only the first).
- The button now draws a close control over the full-screen chat on a phone when the chat page does not run the frame client (it used to leave a phone with no way out). The frame client's `ready` message says `closes: true` when it drew its own, so nothing is drawn twice. `bubble_config` takes `close_label`.
- `check_install`: only a script that runs counts as the line. The address in a link, in text, in `<noscript>`, `<textarea>`, `<template>`, a non-JavaScript `type`, or as a longer file name (`.jsx`, `.js.map`) no longer counts as installed; `http`/`https` and protocol-relative spellings of the same address now match.
- `split_frame_url`: a default port (`:443` on https, `:80` on http) is dropped from the origin, the way a browser writes it, so the button's origin checks match. Behaviour change: the origin of `https://host:443/...` is now `https://host`.
- `frame_headers`: `Strict-Transport-Security` is now opt-in (`hsts=True`, `--hsts`). Behaviour change: 1.0.0 always sent it.
- `chatperch demo`: the second line is labelled as a simulated ping, because the demo sends the ping itself. `chatperch walk` accepts `--site-port`; the walk checks the phone close control and no longer expects Escape to work from inside a chat page without the frame client.
- `csp`: `data-src=` no longer hides an inline script from the policy; `inline_script_hashes` is the new name of `help_script_src_hashes` (which still works).
- README: states the full sandbox, the ping store's 1000-site limit, and what the check does and does not match.

## 1.0.0

First release.

- The chat button script (`bubble.js`): one pasted line, shadow root, CSSOM-only styles, sandboxed frame of any chat URL you give it, a three-word postMessage handshake checked by window and origin, one button per page, phone and laptop layouts, optional load ping.
- The framed page's half of the handshake (`frame_client.js`): ready and close to the embedding origin, focus on cue, optional close button.
- `build_frame_csp` and `frame_headers`: a hash-based Content-Security-Policy for the framed page, with validated source lists.
- `check_install`: working, found, missing, different_address, unreachable, refused or invalid, from the page's HTML and the optional load ping.
- `fetch_page`: bounded in redirects, bytes and total time; public-address guard on every hop with the connection pinned to the checked address; gzip handled within the byte cap.
- `SeenStore`, `parse_beacon` and `make_app` / `chatperch serve`: the optional load-ping adapter.
- `chatperch walk`: a Playwright walk in Chromium and WebKit at phone and laptop sizes.
- `chatperch demo`: the whole loop on your machine.
