---
name: browser-cookies
description: "Import cookies from real Chromium browsers (Comet, Chrome, Arc, Brave, Edge) into the headless browse session for authenticated testing. Use when asked to 'import cookies', 'login to the site', 'authenticate the browser', 'set up cookies', 'use my browser session', 'use my logged-in session', or 'I need to be logged in'. Also trigger when testing requires authentication, the user mentions needing to be logged in for QA, a page returns a login redirect during browse commands, or you encounter an auth wall while browsing. Do NOT use for setting individual cookies manually, clearing/deleting cookies, or managing cookie consent banners, as those are direct browse commands."
metadata:
  strawpot:
    dependencies:
      - browse
---

# Browser Cookies

Import logged-in sessions from your real Chromium browser into the headless browse session. This lets you test authenticated pages without manually entering credentials.

## How it works

1. Find the browse binary (set up via the browse skill)
2. Run `cookie-import-browser` to detect installed browsers and open a picker UI
3. User selects which cookie domains to import in their browser
4. Cookies are decrypted from the browser's cookie store and loaded into the Playwright session

## Steps

### 1. Find the browse binary

The browse binary must already be built. Locate it:

```bash
if command -v browse &>/dev/null; then
  B=browse
  echo "READY: $B"
else
  echo "NEEDS_SETUP: browse binary not found in PATH"
fi
```

If `NEEDS_SETUP`: tell the user the browse binary is not installed and they need to set it up first. Refer them to the browse skill for installation instructions. Do not proceed until the binary is available.

### 2. Import cookies

There are two modes. Use the one that fits the user's request.

#### Mode A: Interactive picker (default)

When the user asks to import cookies without specifying a domain:

```bash
$B cookie-import-browser
```

This auto-detects installed Chromium browsers and opens an interactive picker UI in the user's default browser. The picker lets them:
- Switch between installed browsers (Comet, Chrome, Arc, Brave, Edge)
- Search for domains
- Click "+" to import a domain's cookies
- Click trash to remove imported cookies

Tell the user: **"Cookie picker opened in your browser. Select the domains you want to import, then tell me when you're done."**

Wait for the user to confirm before proceeding.

#### Mode B: Direct import

When the user specifies a browser and/or domain (e.g., "import cookies for github.com from Chrome"):

```bash
$B cookie-import-browser <browser> --domain <domain>
```

Replace `<browser>` with the lowercase browser name (`comet`, `chrome`, `arc`, `brave`, `edge`). Replace `<domain>` with the target domain.

If the user specifies a domain but not a browser, default to `chrome`, since it's the most commonly installed Chromium browser. If unsure which browser the user means, ask.

### 3. Verify

After the user confirms they're done (Mode A) or after the direct import completes (Mode B):

```bash
$B cookies
```

Show the user a summary of imported cookies, including domain names and cookie counts. Confirm which domains are now available in the session.

## Supported browsers

All Chromium-based browsers with local cookie storage:
- **Comet** (comet)
- **Chrome** (chrome)
- **Arc** (arc)
- **Brave** (brave)
- **Edge** (edge)

## Notes

- **macOS Keychain prompt**: The first import per browser may trigger a macOS Keychain dialog asking for permission. Tell the user to click "Allow" or "Always Allow" if they see it.
- **Cookie picker port**: The picker UI is served on the same port as the browse server, so no extra process is needed.
- **Privacy**: Only domain names and cookie counts are shown in the picker UI. No cookie values are exposed.
- **Session persistence**: The browse session persists cookies between commands. Once imported, cookies work immediately for all subsequent browse commands in the same session.
