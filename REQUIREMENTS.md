# 4DOTS Requirements

This file records the tested requirements for the current working deployment.

## Runtime And Tools

| Requirement | Tested value | Purpose |
| --- | --- | --- |
| Node.js | `24` | JavaScript syntax checks and Wrangler |
| Python | `3.12` | Local and CI test runner |
| pytest | `9.0.3` | Automated tests |
| Cloudflare Pages | Static Pages project | Hosts `docs/` |
| Google Apps Script | V8 runtime | Receives form submissions |
| Google Sheet | Private, new/empty tab | Stores leads |

Node version managers read `.node-version` or `.nvmrc`. Cloudflare should have
the build environment variable `NODE_VERSION=24`.

Install local Python requirements:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
```

## Production Configuration

Public frontend values in `docs/app.js`:

- `CONFIG.formEndpoint`: deployed Apps Script `/exec` URL
- `CONFIG.turnstileSiteKey`: public Turnstile site key

Secret Apps Script properties:

- `SHEET_ID`
- `TURNSTILE_SECRET`

Recommended Apps Script properties:

- `SHEET_NAME` (defaults to `Leads`)
- `ALLOWED_HOSTNAMES`
- `RETURN_URL`

The Turnstile site key and `TURNSTILE_SECRET` must belong to the same widget.
Every production hostname must be present in both Turnstile Hostname Management
and `ALLOWED_HOSTNAMES`.

## Sheet Schema

Use a new or empty sheet tab. `Code.gs` creates this exact header row:

```text
Received At
Name
Phone
Email
Services
Industry
Website
Source
Consent
Status
Submission ID
```

There is no `Message` column. Do not manually insert or reorder columns without
updating both `HEADERS` and the row written by `appendLead_()`.

## Deployment Rules

- Frontend-only change: push to `main`; Cloudflare Pages redeploys automatically.
- Existing Apps Script code change: save `Code.gs`, edit the existing web-app
  deployment, choose **New version**, and deploy. The `/exec` URL stays the same.
- New Apps Script deployment: update `CONFIG.formEndpoint` in `docs/app.js`,
  then redeploy Cloudflare Pages because the `/exec` URL changes.
- Script Property-only change: normally no Apps Script redeployment is needed.
- Correct manual Pages command: `npx wrangler pages deploy docs`.
- Do not use plain `npx wrangler deploy`; that targets Workers.

## Verification

```bash
.venv/bin/pytest
node --check docs/app.js
cp apps-script/Code.gs /tmp/4dots-Code.js
node --check /tmp/4dots-Code.js
tidy -errors -quiet docs/index.html
git diff --check
```

`tidy` reports harmless warnings for the standard `inputmode` attributes.
