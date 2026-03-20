---
name: browse
description: "Command reference and usage patterns for headless browser interaction. Covers navigation, element interaction, snapshot diffing, responsive testing, form filling, dialog handling, cookie management, and visual evidence gathering. Use when you need to test a feature, verify a deployment, dogfood a user flow, take a screenshot, assert element states, diff before/after actions, test responsive layouts, or file a bug with visual evidence. Also use when asked to 'open in browser', 'test the site', 'take a screenshot', 'check the page', 'dogfood this', 'browse to', 'navigate to', 'check if the page works', or 'verify the UI'. This skill is a methodology and command reference; the browse binary must be installed separately. Do NOT use for API testing, backend-only verification, or curl/wget HTTP requests."
metadata:
  strawpot:
    tools:
      browse:
        description: Headless Chromium CLI for navigation, snapshots, and browser interaction
        install:
          macos: "See Binary Setup section in skill body (custom binary, not in package managers)"
          linux: "See Binary Setup section in skill body (custom binary, not in package managers)"
---

# Browse: Headless Browser Interaction

Persistent headless Chromium. First call auto-starts (~3s), then ~100ms per command.
State persists between calls (cookies, tabs, login sessions).

## Binary Setup

This skill documents how to use the browse binary. The binary itself must be installed
separately. Throughout this document, `$BROWSE` refers to the path to the browse binary.

**Before first use, verify the binary is available:**

```bash
if command -v browse &>/dev/null; then
  BROWSE=browse
  echo "READY: $BROWSE"
else
  echo "ERROR: browse binary not found in PATH"
  echo "See installation options below."
  exit 1
fi
```

### Installation Options

**Option 1 — Build from gstack source (recommended):**

```bash
# Requires Bun (https://bun.sh)
cd /path/to/gstack
bun build browse.ts --compile --outfile browse
# Copy the binary to somewhere in your PATH
cp browse /usr/local/bin/browse
```

**Option 2 — Download from gstack releases:**

Check the [gstack releases page](https://github.com/garrytan/gstack/releases) for pre-built
binaries. Download the binary for your platform and place it in your PATH.

**Option 3 — Add gstack bin to PATH:**

If you already have a gstack installation with the compiled binary, add its directory to PATH:

```bash
export PATH="/path/to/gstack:$PATH"
```

Add this to your shell profile (`~/.bashrc`, `~/.zshrc`) to persist across sessions.

**Option 4 — Use MCP browser tools:**

Connect to an MCP server that exposes browser capabilities as an alternative to the native binary.

### Verifying Installation

After installation, confirm it works:

```bash
browse status   # should report server health
browse goto https://example.com
browse text     # should print page content
```

Once installed, set `$BROWSE` to the binary path and all commands below will work.

## Core QA Patterns

### 1. Verify a page loads correctly
Start broad (content → console → network → specific element) to catch issues at every layer before drilling in.
```bash
$BROWSE goto https://yourapp.com
$BROWSE text                          # content loads?
$BROWSE console                       # JS errors?
$BROWSE network                       # failed requests?
$BROWSE is visible ".main-content"    # key elements present?
```

### 2. Test a user flow
Snapshot interactive elements first to discover refs, fill and click using those refs, then diff to confirm the flow progressed.
```bash
$BROWSE goto https://app.com/login
$BROWSE snapshot -i                   # see all interactive elements
$BROWSE fill @e3 "user@test.com"
$BROWSE fill @e4 "password"
$BROWSE click @e5                     # submit
$BROWSE snapshot -D                   # diff: what changed after submit?
$BROWSE is visible ".dashboard"       # success state present?
```

### 3. Verify an action worked
Take a baseline snapshot before the action, then diff after. The unified diff shows exactly what changed, no guessing.
```bash
$BROWSE snapshot                      # baseline
$BROWSE click @e3                     # do something
$BROWSE snapshot -D                   # unified diff shows exactly what changed
```

### 4. Visual evidence for bug reports
Annotated screenshots with ref labels make bug reports unambiguous. Combine with console errors for full context.
```bash
$BROWSE snapshot -i -a -o /tmp/annotated.png   # labeled screenshot
$BROWSE screenshot /tmp/bug.png                # plain screenshot
$BROWSE console                                # error log
```

### 5. Find all clickable elements (including non-ARIA)
Use `-C` when standard `-i` misses elements, since custom divs with `cursor:pointer` or `onclick` won't appear in the ARIA tree.
```bash
$BROWSE snapshot -C                   # finds divs with cursor:pointer, onclick, tabindex
$BROWSE click @c1                     # interact with them
```

### 6. Assert element states
Use `is` for declarative checks that read like assertions. Fall back to `js` for custom conditions not covered by built-in states.
```bash
$BROWSE is visible ".modal"
$BROWSE is enabled "#submit-btn"
$BROWSE is disabled "#submit-btn"
$BROWSE is checked "#agree-checkbox"
$BROWSE is editable "#name-field"
$BROWSE is focused "#search-input"
$BROWSE js "document.body.textContent.includes('Success')"
```

### 7. Test responsive layouts
`responsive` captures all three breakpoints in one command. Use `viewport` + `screenshot` for custom sizes.
```bash
$BROWSE responsive /tmp/layout        # mobile + tablet + desktop screenshots
$BROWSE viewport 375x812              # or set specific viewport
$BROWSE screenshot /tmp/mobile.png
```

### 8. Test file uploads
Target the file input by selector, then verify the success state appeared. No need to interact with OS file dialogs.
```bash
$BROWSE upload "#file-input" /path/to/file.pdf
$BROWSE is visible ".upload-success"
```

### 9. Test dialogs
Set up the handler *before* triggering the dialog, because dialogs block execution, so the accept/dismiss must be pre-registered.
```bash
$BROWSE dialog-accept "yes"           # set up handler
$BROWSE click "#delete-button"        # trigger dialog
$BROWSE dialog                        # see what appeared
$BROWSE snapshot -D                   # verify deletion happened
```

### 10. Compare environments
Quick way to spot content drift between staging and production without manually switching tabs.
```bash
$BROWSE diff https://staging.app.com https://prod.app.com
```

### 11. Show screenshots to the user
Screenshots save to disk but are invisible to the user unless you explicitly read the file. This step is easy to forget.
After `$BROWSE screenshot`, `$BROWSE snapshot -a -o`, or `$BROWSE responsive`, always use the Read tool on the output PNG(s) so the user can see them. Without this step, screenshots are invisible to the user.

## User Handoff

When you hit something you cannot handle in headless mode (CAPTCHA, complex auth, multi-factor
login), hand off to the user:

```bash
# 1. Open a visible Chrome at the current page
$BROWSE handoff "Stuck on CAPTCHA at login page"

# 2. Tell the user what happened
#    "I've opened Chrome at the login page. Please solve the CAPTCHA
#     and let me know when you're done."

# 3. When user says "done", re-snapshot and continue
$BROWSE resume
```

**When to use handoff:**
- CAPTCHAs or bot detection
- Multi-factor authentication (SMS, authenticator app)
- OAuth flows that require user interaction
- Complex interactions the AI cannot handle after 3 attempts

The browser preserves all state (cookies, localStorage, tabs) across the handoff.
After `resume`, you get a fresh snapshot of wherever the user left off.

## Snapshot Flags

The snapshot is your primary tool for understanding and interacting with pages.

```
-i        --interactive           Interactive elements only (buttons, links, inputs) with @e refs
-c        --compact               Compact (no empty structural nodes)
-d <N>    --depth                 Limit tree depth (0 = root only, default: unlimited)
-s <sel>  --selector              Scope to CSS selector
-D        --diff                  Unified diff against previous snapshot (first call stores baseline)
-a        --annotate              Annotated screenshot with red overlay boxes and ref labels
-o <path> --output                Output path for annotated screenshot (default: /tmp/browse-annotated.png)
-C        --cursor-interactive    Cursor-interactive elements (@c refs: divs with pointer, onclick)
```

All flags combine freely. `-o` only applies when `-a` is also used.
Example: `$BROWSE snapshot -i -a -C -o /tmp/annotated.png`

**Ref numbering:** @e refs are assigned sequentially (@e1, @e2, ...) in tree order.
@c refs from `-C` are numbered separately (@c1, @c2, ...).

After snapshot, use @refs as selectors in any command:
```bash
$BROWSE click @e3       $BROWSE fill @e4 "value"     $BROWSE hover @e1
$BROWSE html @e2        $BROWSE css @e5 "color"       $BROWSE attrs @e6
$BROWSE click @c1       # cursor-interactive ref (from -C)
```

**Output format:** indented accessibility tree with @ref IDs, one element per line.
```
  @e1 [heading] "Welcome" [level=1]
  @e2 [textbox] "Email"
  @e3 [button] "Submit"
```

Refs are invalidated on navigation, so run `snapshot` again after `goto`.

## Full Command List

### Navigation
| Command | Description |
|---------|-------------|
| `back` | History back |
| `forward` | History forward |
| `goto <url>` | Navigate to URL |
| `reload` | Reload page |
| `url` | Print current URL |

### Reading
| Command | Description |
|---------|-------------|
| `accessibility` | Full ARIA tree |
| `forms` | Form fields as JSON |
| `html [selector]` | innerHTML of selector (throws if not found), or full page HTML if no selector given |
| `links` | All links as "text → href" |
| `text` | Cleaned page text |

### Interaction
| Command | Description |
|---------|-------------|
| `click <sel>` | Click element |
| `cookie <name>=<value>` | Set cookie on current page domain |
| `cookie-import <json>` | Import cookies from JSON file |
| `cookie-import-browser [browser] [--domain d]` | Import cookies from Chrome, Arc, Brave, or Edge (opens picker, or use --domain for direct import) |
| `dialog-accept [text]` | Auto-accept next alert/confirm/prompt. Optional text is sent as the prompt response |
| `dialog-dismiss` | Auto-dismiss next dialog |
| `fill <sel> <val>` | Fill input |
| `header <name>:<value>` | Set custom request header (colon-separated, sensitive values auto-redacted) |
| `hover <sel>` | Hover element |
| `press <key>` | Press key: Enter, Tab, Escape, ArrowUp/Down/Left/Right, Backspace, Delete, Home, End, PageUp, PageDown, or modifiers like Shift+Enter |
| `scroll [sel]` | Scroll element into view, or scroll to page bottom if no selector |
| `select <sel> <val>` | Select dropdown option by value, label, or visible text |
| `type <text>` | Type into focused element |
| `upload <sel> <file> [file2...]` | Upload file(s) |
| `useragent <string>` | Set user agent |
| `viewport <WxH>` | Set viewport size |
| `wait <sel\|--networkidle\|--load>` | Wait for element, network idle, or page load (timeout: 15s) |

### Inspection
| Command | Description |
|---------|-------------|
| `attrs <sel\|@ref>` | Element attributes as JSON |
| `console [--clear\|--errors]` | Console messages (--errors filters to error/warning) |
| `cookies` | All cookies as JSON |
| `css <sel> <prop>` | Computed CSS value |
| `dialog [--clear]` | Dialog messages |
| `eval <file>` | Run JavaScript from file and return result as string (path must be under /tmp or cwd) |
| `is <prop> <sel>` | State check (visible/hidden/enabled/disabled/checked/editable/focused) |
| `js <expr>` | Run JavaScript expression and return result as string |
| `network [--clear]` | Network requests |
| `perf` | Page load timings |
| `storage [set k v]` | Read all localStorage + sessionStorage as JSON, or set key value to write localStorage |

### Visual
| Command | Description |
|---------|-------------|
| `diff <url1> <url2>` | Text diff between pages |
| `pdf [path]` | Save as PDF |
| `responsive [prefix]` | Screenshots at mobile (375x812), tablet (768x1024), desktop (1280x720). Saves as {prefix}-mobile.png etc. |
| `screenshot [--viewport] [--clip x,y,w,h] [selector\|@ref] [path]` | Save screenshot (supports element crop via CSS/@ref, --clip region, --viewport) |

### Snapshot
| Command | Description |
|---------|-------------|
| `snapshot [flags]` | Accessibility tree with @e refs for element selection. Flags: -i interactive only, -c compact, -d N depth limit, -s sel scope, -D diff vs previous, -a annotated screenshot, -o path output, -C cursor-interactive @c refs |

### Meta
| Command | Description |
|---------|-------------|
| `chain` | Run commands from JSON stdin. Format: `[["cmd","arg1",...],...]` |

### Tabs
| Command | Description |
|---------|-------------|
| `closetab [id]` | Close tab |
| `newtab [url]` | Open new tab |
| `tab <id>` | Switch to tab |
| `tabs` | List open tabs |

### Server
| Command | Description |
|---------|-------------|
| `handoff [message]` | Open visible Chrome at current page for user takeover |
| `restart` | Restart server |
| `resume` | Re-snapshot after user takeover, return control to AI |
| `status` | Health check |
| `stop` | Shutdown server |
