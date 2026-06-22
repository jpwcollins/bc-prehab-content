# bc-prehab-content

Public clinical content for the **BC Prehab App**. The app fetches these files
from the `main` branch via the jsDelivr CDN and shows them offline-first. Editing
here lets you update content **without** a new app build or App Store review.

## Files

- `index.json` — the manifest. Lists each dataset with its `file`, `formatVersion`, and `version` (a `YYYY-MM-DD` date).
- `medications.json` — preoperative medication advice list.
- `clinicalCalculators.json` — clinical calculator directory.
- `hospitals.json` — hospital directory (each hospital has an optional `specificInfoUrl`).

## How to publish a content change

1. Edit the dataset file (e.g. `medications.json`).
2. Bump that dataset's `version` date in `index.json` to today (`YYYY-MM-DD`).
   The app only re-downloads when the remote date is **newer** than the installed date.
3. Commit and push to `main`.

That's it. No git tags, no releases, no manual cache purge — the included GitHub
Action (`.github/workflows/purge-cdn.yml`) refreshes the jsDelivr edge cache on
every push to `main`.

## Hospital "Specific information"

Each hospital in `hospitals.json` may include a `specificInfoUrl`:

```json
{ "id": "burnaby", "name": "Burnaby Hospital", "specificInfoUrl": "https://example.org/burnaby-instructions.pdf" }
```

- A URL ending in `.pdf` opens in the app's built-in document viewer.
- Any other web page opens in the browser.
- Empty, absent, or invalid → the app calmly shows "No specific information is provided at this time."

The URL can point anywhere — the hospital's own site or a file committed into this repo.

## Format versions

`formatVersion` must stay `1` unless the app is updated to understand a new
shape. The app **only** accepts content whose `formatVersion` exactly matches
what the installed build supports; newer or older formats are rejected and the
app keeps its last-good (or built-in) copy.

## Safety

- Never commit malformed JSON — the app validates every file and rejects bad data, keeping the prior copy.
- The app sends a plain anonymous GET only: no identifiers, headers, analytics, or usage data.
