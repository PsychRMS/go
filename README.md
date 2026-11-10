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
