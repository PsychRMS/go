# go.psychresidency.org — the program's link page

Program emails (evaluation invitations, reminders, resident feedback invitations) carry links like
`https://go.psychresidency.org/#e=…` instead of `https://script.google.com/macros/s/…/exec?e=…`.
Mail filters treat `script.google.com` links as suspicious because phishers use them a lot, so program
emails were landing in Junk. This small page, hosted free on GitHub Pages under the program's own domain,
forwards each link to the evaluator page (`…/exec?e=…`).

It forwards only the parameters `e`, `fb`, `stop` (only `stop=1`, only with `fb`) and `doc`, each checked
against a strict pattern. Anything else shows a plain "This link does not work" message. It loads nothing from
other sites, has no analytics, sends no referrer, and never writes link contents onto the page.

The link part after `#` (a fragment) is never sent to GitHub's servers, so tokens do not appear in GitHub's logs; only
the visitor's browser reads it. The page also accepts the same parameters after `?` (same checks), but the app always
sends `#` links; a link with both is refused. A link alone still does not open an evaluation or the feedback form: the
page emails a 6-digit sign-in number to the person's address.

## Setup (Program Director, about 20 minutes, once)

### 1. Fill in the evaluator page address

1. Script editor of the **Production** project → **Deploy** → **Manage deployments** → the evaluator deployment
   whose access is **Anyone** → copy its **Web app URL** (ends in `/exec`). It is the same value as the Script
   Property `RMS_EVALUATOR_URL`.
2. Open `index.html` in a text editor, find the line
   `var TARGET = 'https://script.google.com/macros/s/REPLACE_WITH_DEPLOYMENT_ID/exec';`
   and replace the address between the quotes with the one you copied. Save. Change nothing else.

### 2. Create the GitHub repository

1. Sign in to GitHub as an owner of the **PsychRMS** organisation.
2. **New repository** → Owner `PsychRMS`, name `go`, **Public** (GitHub Pages on a free plan needs public), no template.
3. **Add file** → **Upload files** → upload the three files from this folder: `index.html`, `CNAME`, `README.md`
   → **Commit changes** to branch `main`.

### 3. Turn on GitHub Pages

1. In the `go` repository: **Settings** → **Pages**.
2. **Source**: *Deploy from a branch*; **Branch**: `main`, folder `/ (root)` → **Save**.
3. **Custom domain**: `go.psychresidency.org` → **Save** (the `CNAME` file already says this).
4. Wait for the DNS check (step 4) to pass, then tick **Enforce HTTPS**. It may take up to an hour before the tick box
   is available while GitHub issues the certificate.

### 4. Add the DNS record (Squarespace Domains)

1. Squarespace → **Domains** → `psychresidency.org` → **DNS** → **DNS settings** → **Add record**.
2. Type **CNAME**, Host **`go`**, Data / points to **`psychrms.github.io`**, TTL default → **Save**.
3. Do not change the existing MX, SPF, DKIM or DMARC records.

### 5. Check it

Open `https://go.psychresidency.org/` in a browser: it should say **"This link does not work"** (no link given) and the
padlock should show a valid certificate. Then copy a real evaluation link from the app (Copy link), and open
`https://go.psychresidency.org/#e=` followed by the part after `?e=`: the evaluator page should open.

### 6. Switch the app over

Script editor → **Project Settings** → **Script Properties** → add `RMS_LINK_BASE` = `https://go.psychresidency.org`
(exactly: https, no slash at the end, no path). Run `runProductionChecks()`: the row
**Own-domain email links (RMS_LINK_BASE)** should be PASS (links are `https://go.psychresidency.org/#…`). From now on every emailed, copied or WhatsApp link uses
the own domain. Links already sent keep working (they point to the old address, which still works).

To undo: delete the Script Property `RMS_LINK_BASE`; links go back to the `script.google.com` address at once.

## Attendance links: sign-in and conductor (dev.53)

The app can also make the attendance links short: the resident sign-in link `https://go.psychresidency.org/#s=MMR947`
(forwarded to `<sign-in app>/exec?code=MMR947`) and the conductor link `https://go.psychresidency.org/#c=<32-character key>`
(forwarded to `<sign-in app>/exec?view=conductor&k=<key>`). The QR code and the WhatsApp text carry the same short link.
These go to the **attendance sign-in web app** (the old attendance app; its address is the Script Property
`RMS_SIGNIN_URL` of the Production project), not to the evaluator page, so the page has a second address, `SIGNIN_TARGET`.
Only a session code of 3–20 capital letters or digits (`s`) or a 32-character lower-case hex key (`c`) is forwarded, each
alone; anything else shows "This link does not work". While `SIGNIN_TARGET` still says `PASTE_LEGACY_EXEC_URL` the page
shows "This link is not configured yet" for these links and forwards nothing.

To switch the attendance links over (PD or system@, once; the evaluator part above must already work):

1. Copy the value of the Script Property `RMS_SIGNIN_URL` (Production project → **Project Settings** → **Script
   Properties**). It ends in `/exec`.
2. In the `go` repository on GitHub open `index.html` → pencil (**Edit**) → find
   `var SIGNIN_TARGET = 'PASTE_LEGACY_EXEC_URL';` and put the copied address between the quotes. Change nothing else →
   **Commit changes** to `main`. GitHub Pages publishes it within a few minutes.
3. Check: open `https://go.psychresidency.org/#s=` followed by the code of today's session — the sign-in page for that
   session should open.
4. Only then add the Script Property `RMS_LINK_SIGNIN_READY` = `TRUE` in the Production project. From now on new sign-in
   and conductor links (copied, WhatsApp, QR, the "Attendance Link" cell of new sessions) are short. Without it — or without
   `RMS_LINK_BASE` — the app keeps making the direct `…/exec?code=` links, so residents never get a short link the page
   cannot open. Links already shared keep working either way.

If the sign-in app gets a new deployment (a new `/exec` address): first update `RMS_SIGNIN_URL`, then `SIGNIN_TARGET` here
(step 2). To undo: delete `RMS_LINK_SIGNIN_READY`.

## Updating this page in the `go` repository

Every change to `index.html` in the app's source (`v2/go/index.html`) must be copied to the `go` repository by hand:
GitHub → `PsychRMS/go` → `index.html` → **Edit** → paste the new file, then put back the two addresses (`TARGET` and
`SIGNIN_TARGET`) from the old version before you commit. A file with a placeholder still in it shows "not configured"
for that kind of link instead of forwarding.

## If the evaluator deployment changes

A **new** deployment has a new `/exec` address: update `TARGET` in `index.html` in the `go` repository (edit the file on
GitHub → Commit) and the Script Property `RMS_EVALUATOR_URL`. Editing an existing deployment (new version) keeps the
address and needs nothing here.

### After every redeploy (each release)

1. Script editor → **Deploy** → **Manage deployments** → edit the **Anyone** (evaluator) deployment → **Version: New version**
   → **Deploy**. Do not create a new deployment: that would change the `/exec` address.
2. Check that its **Web app URL** still equals `RMS_EVALUATOR_URL` and the `TARGET` line in `index.html` here.
3. Run `runProductionChecks()`: the row **Own-domain email links (RMS_LINK_BASE)** says "confirm go/index.html TARGET equals
   RMS_EVALUATOR_URL (…)" with the address to compare.
4. Open one copied link (an evaluation link, or the clinic schedule link `…/#doc=…`) to confirm it still opens.
