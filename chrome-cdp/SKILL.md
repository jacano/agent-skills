---
name: chrome-cdp
description: Drive the person's own Chrome — already signed in — over the DevTools protocol, and keep that one connection alive. Use when a task needs a page that only exists logged in in their browser, a tab they already have open, or a site that blocks a fresh automated browser.
version: 1.1.0
---

# chrome-cdp

Three files: this one, [`cdp-daemon.mjs`](cdp-daemon.mjs) and [`cdp.mjs`](cdp.mjs). No
dependencies; Node 22 or later for the global `WebSocket`. Nothing to install.

They drive the person's *own* Chrome — the one with their session, cookies and tabs — so a
page that needs a login can be read, and a site does not challenge it the way it challenges a
fresh automated browser.

## The one rule

**Chrome grants remote debugging to one external client per armed session.** The grant is
spent by the connect, and closing the socket does not return it: the next attempt hangs with
no error until a person re-arms the box. So one long-lived process owns the socket and
nothing else opens one. Never reconnect in a loop.

(A Chrome launched with `--remote-debugging-port` on a throwaway profile accepts several
clients. That is the test profile, not the person's.)

## A new machine, from nothing

**1. Node 22 or newer.**

```bash
node --version                             # v22.x or later; written on v24
node -e "console.log(typeof WebSocket)"    # must print: function
```

`WebSocket` is the whole transport. The daemon stops on its first line without it, because a
missing global and a missing Chrome otherwise both surface as `refused`.

**2. Chrome, running, with remote debugging allowed — once.** Open `chrome://inspect` →
**Remote debugging** → check **Allow remote debugging for this browser instance**. That page
prints `Server running at: 127.0.0.1:<port>`: the port is random and the daemon finds it, but
that is where to read it by eye, and it is the only place the grant is given or taken back.

**3. This directory.** A clone is enough; state lives in the system temp directory.

```bash
git clone https://github.com/jacano/agent-skills
cd agent-skills
```

Check the machine before blaming the skill:

```bash
node chrome-cdp/cdp-daemon.mjs --probe     # the ports found, and what each one answers
```

`open` is the answer you want. Then:

```bash
node chrome-cdp/cdp.mjs start
node chrome-cdp/cdp.mjs eval "location.href"
```

That must print a URL. If it does, the machine is ready.

### When it will not cooperate

| Symptom | Do |
| --- | --- |
| `node: command not found`, or `typeof WebSocket` is `undefined` | Install Node 22 or newer. |
| `{ "ports": [] }` | No Chrome listening. On a Linux without `lsof`, read the port from `chrome://inspect` and pass `CDP_PORT=<port>`. |
| The box cannot be checked | The browser is managed and a policy forbids remote debugging (`RemoteDebuggingAllowed`). Only a separate Chrome with `--remote-debugging-port` and its own `--user-data-dir` gets round it, and it will not carry their session. |
| No **Remote debugging** page at all | An older Chrome. `--remote-debugging-port=9222` means closing it first and losing the running session. |
| They browse in Edge, Brave or Chromium | Read the port from their own `edge://inspect` and pass `CDP_PORT=<port>`. |

## The two failures, and they are not the same

| The daemon says | Means | Do |
| --- | --- | --- |
| `refused` | Nothing on those ports speaks DevTools: Chrome is closed, or the box is unchecked. | Open `chrome://inspect/#remote-debugging`, check the box, `start` again. |
| `hung` | The server is there and said nothing: the one slot is taken, by a dead daemon or a spent grant. | **Uncheck, wait a second, re-check** the box. Then `start` once. |

`refused` answers in milliseconds, `hung` costs the full 6-second probe: that difference is
the diagnosis, so never read a hang as "wrong port".

## What not to reach for

`--remote-debugging-port` on the default profile (Chrome no longer offers it),
`chromium.connectOverCDP` (it calls HTTP `/json/version` first, and the inspect endpoint
answers 404 to every HTTP path), and copying the profile's `Cookies` while Chrome runs (SQLite
holds an exclusive lock). The endpoint takes `ws://127.0.0.1:<port>/devtools/browser`, and
`CDP_PORT` overrides the port discovery.

## Commands

| Command | Returns |
| --- | --- |
| `start` / `stop` | attach (idempotent) / close the daemon |
| `ping` | `{port, product, protocol, pages}` |
| `targets` | every page target: `{id, title, url}` |
| `open <url> [--foreground]` | `{targetId}`; background by default, focus untouched |
| `eval [--url substr] [--target id] [--wait] [--timeout ms] <expr>` | the expression's value |
| `close <targetId>` | `{closed}` |
| `activate <targetId>` | bring a tab to the front (their screen) |
| `shot <file> [--url substr]` | `{path, bytes}`; brings the tab forward to render it |

`eval` is the whole DOM API and returns objects as JSON. `--wait` polls every 300 ms until the
value is truthy, which is how a page is allowed to settle. Every command fails in about 40 ms
when no daemon is attached, so a wrong assumption costs a line, not a timeout.

## Recipes

```bash
# read a page they are already signed in to
node chrome-cdp/cdp.mjs open "https://example.com/dashboard"
node chrome-cdp/cdp.mjs eval --url example.com --wait "document.querySelector('.row')?.innerText"

# set a field the way a framework notices, click, then wait for the result
node chrome-cdp/cdp.mjs eval --url example.com "(el => { el.value = 'x'; el.dispatchEvent(new Event('input', {bubbles:true})); })(document.querySelector('#name'))"
node chrome-cdp/cdp.mjs eval --url example.com "(el => { el.scrollIntoView(); el.click(); })(document.querySelector('button[type=submit]'))"
node chrome-cdp/cdp.mjs eval --url example.com --wait --timeout 60000 "!!document.querySelector('.confirmation')"

# close what you opened
node chrome-cdp/cdp.mjs close <targetId>
```

## Rules of engagement

- **Do not tidy up the socket.** `stop` is for a deliberate restart; re-attaching costs a
  person's click.
- **Do not kill Chrome**, and do not launch a second Chrome on their profile: that destroys
  the session you came for.
- **Background by default**, and no `activate` unless they have to look. `shot` does bring a
  tab forward; say so.
- **Their session is their identity.** Reading is yours; pressing a button that spends money,
  sends a message or deletes something is theirs to approve.
- **Close what you open, keep what was theirs.**

The "Chrome is being controlled by automated test software" banner is expected while the
daemon is attached. Nothing needs turning off.

## What was tested

Windows, Chrome 157, Node 24: the attach, `--probe` on both shapes, every command above, the
missing-`WebSocket` guard, and `shot`. Not tested: macOS or Linux, an older Chrome, a managed
browser, and Edge or Brave through `CDP_PORT`.
