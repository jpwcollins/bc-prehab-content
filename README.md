# bc-prehab-content

Public clinical content for the **BC Prehab App**. The app fetches these files
from the `main` branch via the jsDelivr CDN and shows them offline-first. Editing
here lets you update content **without** a new app build or App Store review.

## Files

- `index.json` — the manifest. Lists each dataset with its `file`, `formatVersion`, and `version` (a `YYYY-MM-DD` date).
- `medications.json` — preoperative medication advice list.
- `clinicalCalculators.json` — clinical calculator directory.
- `hospitals.json` — hospital directory (each hospital has an optional `specificInfoUrl`).
- `privacy.html` — the app's privacy policy. Not clinical content and not
  fetched by the app; it lives here so GitHub Pages can serve it as the policy
  URL the App Store listing points at. See below.

## How to publish a content change

1. Edit the dataset file (e.g. `medications.json`).
2. Bump that dataset's `version` date in `index.json` to today (`YYYY-MM-DD`).
   The app only re-downloads when the remote date is **newer** than the installed date.
3. Commit and push to `main`.

That's it. No git tags and no releases — the included GitHub Action
(`.github/workflows/purge-cdn.yml`) refreshes the jsDelivr edge cache on every
push to `main`.

**Expect up to ~12 hours before users see a change.** jsDelivr caches `@main`
paths for 12 hours and caches the branch→commit resolution itself, so a push is
not instantly visible. The purge workflow bounds that delay but is neither
instant nor uniform — edge nodes expire independently, so two users in different
regions can briefly get different content. The app checks weekly by default, so
this rarely matters; just don't assume a correction is live the moment you push.

If a change isn't landing at all, check that the workflow ran, or purge by hand:

```bash
curl "https://purge.jsdelivr.net/gh/jpwcollins/bc-prehab-content@main/index.json"
```

## Hospital "Specific information"

Each hospital in `hospitals.json` may include a `specificInfoUrl`:

```json
{ "id": "burnaby", "name": "Burnaby Hospital", "specificInfoUrl": "https://example.org/burnaby-instructions.pdf" }
```

- A URL ending in `.pdf` opens in the app's built-in document viewer.
- Any other web page opens in the browser.
- Empty, absent, or invalid → the app calmly shows "No specific information is provided at this time."

The URL can point anywhere — the hospital's own site or a file committed into this repo.

## The privacy policy page

`privacy.html` is the published privacy policy for the app. It is a single
self-contained file: no scripts, no external fonts, no requests of any kind,
which is the least a privacy policy can do.

**It is served by GitHub Pages, not by jsDelivr.** jsDelivr returns `.html` with
`Content-Type: text/plain` and `X-Content-Type-Options: nosniff`, so a browser
shows the source instead of the page — do not link the CDN copy. Pages is
enabled on this repo (branch `main`, path `/`), and serves the policy at:

```
https://jpwcollins.github.io/bc-prehab-content/privacy.html
```

That is the URL for the App Store Connect privacy-policy field. A push to `main`
rebuilds the site in about a minute. Pages does not affect how the datasets are
served — the app keeps fetching those through jsDelivr.

The source of record is `PRIVACY.md` in the app repo; change both together.

## Format versions

`formatVersion` must stay `1` unless the app is updated to understand a new
shape. The app **only** accepts content whose `formatVersion` exactly matches
what the installed build supports; newer or older formats are rejected and the
app keeps its last-good (or built-in) copy.

## Safety

- Never commit malformed JSON — the app validates every file and rejects bad data, keeping the prior copy.
- The app sends a plain anonymous GET only: no identifiers, headers, analytics, or usage data.
