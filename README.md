# MCAS Redirector

Chromium extension (Manifest V3) that redirects corporate domains to their
Microsoft Defender for Cloud Apps (MDCA) `.mcas.ms` counterparts, so you reach
them already signed in via your work account.

## Install (developer mode)

1. Open `chrome://extensions` (or equivalent in any Chromium browser).
2. Toggle **Developer mode** (top-right).
3. Click **Load unpacked**.
4. Select this folder (`mcas-plugin/`).

No build step. No background script. Pure declarative.

## How it works

- `manifest.json` registers a static `declarative_net_request` ruleset.
- `rules.json` holds one redirect rule per corporate domain.
- `declarativeNetRequest` fires **before** the request leaves the browser, so
  the original URL is never sent on the wire (no leak, no flicker).
- Only `main_frame` navigations are rewritten — subresources and XHR that may
  legitimately hit the real domains are left alone.

### Rewrite shape

`https://<sub>.example.com/<path>` → `https://<sub>.example.com.mcas.ms/<path>`

Subdomains are preserved (e.g. `myco.atlassian.net` → `myco.atlassian.net.mcas.ms`),
as are path, query, and fragment.

### Loop safety

Each regex anchors the host end with `(/|$)`, so an already-`.mcas.ms` URL
(e.g. `github.com.mcas.ms/`) does **not** re-match its own rule. No infinite
redirect.

## Supported domains

| Rule ID | Domain              |
|---------|---------------------|
| 1       | github.com          |
| 2       | atlassian.net       |
| 3       | console.aws.amazon.com |
| 4       | signin.aws.amazon.com  |
| 5       | sharepoint.com      |

## Add a domain

1. Copy any rule block in `rules.json`.
2. Give it a unique `id`.
3. Swap the host in both `regexFilter` and `regexSubstitution`.
4. Add a matching `host_permissions` entry in `manifest.json`
   (e.g. `*://*.example.com/*`).
5. Reload the extension on `chrome://extensions`.

## Notes / caveats

- MDCA SSO bounces through domains not in the ruleset (e.g.
  `login.microsoftonline.com`) pass through untouched — correct behavior.
- A corporate domain not in the list goes direct (no MCAS). Add it when found.
- Not published to the Web Store; loaded unpacked only. Web Store ruleset
  limits are irrelevant.
