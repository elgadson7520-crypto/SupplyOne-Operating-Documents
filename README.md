# SupplyOne Operating Documents

Region-wide operating-document control center for SupplyOne's Southwest plants.
Single-page web app, deployed on Vercel, syncs document metadata from Google Drive folders.

**Live site:** https://supplyone-operating-documents.vercel.app
**Maintained by:** Eric Gadson · Continuous Improvement Manager · Southwest Region

---

## What it does

* Hierarchical sidebar: **Plant → Machine → Document category**
* Auto-syncs from one or more Google Drive folders every 60 seconds
* Inline document viewer (PDFs, DOCX, XLSX, PPTX) — no leaving the page
* Status tracking: Current / Due Soon (≤90d) / Overdue / Draft, computed from `Next Review Date`
* Two roles: Viewer (read-only) and Editor (add / edit / delete)
* CSV export of the full document list

---

## Plants currently configured

| Plant         | Machines |
|---------------|----------|
| Dallas        | 130 38x100 FFG, 140 38x100 FFG, 210 66x110 RDC, 220 66x80 RDC, SRPACK |
| Oklahoma City | 110 37x82 FFG, 120 50x110 FFG, 210 66x110 RDC, 220 66x110 RDC, 230 66x110 RDC |
| Albuquerque   | 110 47x95 FFG, 120 38x80 FFG, 210 66x110 RDC, 510 64x160 Prnt |
| Chicago       | (machines TBD) |
| Arkansas      | (machines TBD) |
| Tucson        | (machines TBD) |
| Wisconsin     | (machines TBD) |

To add machines to a plant, edit the `MACHINES` object inside the `<script>` block of `index.html`.

---

## Drive folder sources

The 4 Drive folders below are auto-synced into Dallas. To add more (per-plant or otherwise),
edit `DRIVE_CONFIG.sources` inside `index.html`:

```js
const DRIVE_CONFIG = {
  apiKey: 'AIza…',
  refreshInterval: 60,
  sources: [
    { folderId: '1CcewI-y7TyzhhGD3QKzFYFrEMCTiNn0T', location: 'Dallas' },
    { folderId: '11igl01vTeb-P-PECGPnSneEEcSaKT6Nd', location: 'Dallas' },
    { folderId: '1UkjkcJIEI9BvgTnIE9mX-JSJaA9TDBAh', location: 'Dallas' },
    { folderId: '1-vY-t3K3DJRVE3c12xeDVFaQS27C_CQp', location: 'Dallas' }
    // To add Chicago: { folderId: 'YOUR_FOLDER_ID', location: 'Chicago' }
  ]
};
```

**Each Drive folder must be set to "Anyone with the link can view"** for the API to read it.
A private folder will fail to sync; the rest still load.

### Filename convention

The auto-sync extracts machine + doctype from filenames matching:

```
PREFIX-TYPE-NUM_Title_Words.ext
```

Examples that work:
* `SRPACK-SOP-001_Box_Type_Reference_Guide.docx` → SRPACK / SOP / Box Type Reference Guide
* `SRPACK-QMF-002_Operator_Training_Competency_Record.docx` → SRPACK / QMF / Operator Training & Competency Record

Files that don't match the pattern still appear, but bucket as **External** type with no machine assigned. Editors can fix the assignment manually.

Recognized doctype prefixes: `QMF, QMI, QMP, SOP, WIQ, Safety, External`

---

## Deployment

This is a **single-file static site**. No build step.

* GitHub repo → Vercel auto-deploys on every push to `main`.
* Vercel framework preset: **Other** (or "No framework").
* Output directory: leave empty.
* Root: `/` — `index.html` lives at the project root.

---

## Access

| Role   | Password    |
|--------|-------------|
| Viewer | `view2025`  |
| Editor | `edit2025`  |

⚠️  These are hardcoded in `index.html` for now. **Anyone with browser dev tools can read them.** Treat as access-gating, not real auth. Replace with a real backend (Supabase, Firebase, Auth0) before relying on this for anything sensitive.

---

## Maintenance cheat sheet

| Task                          | Where to edit                                |
|-------------------------------|----------------------------------------------|
| Add a plant                   | `LOCATION_ORDER` + `MACHINES` + `LOC_LABELS` |
| Add a machine to a plant      | `MACHINES['<plant>'].push('<machine>')`      |
| Add another Drive folder      | `DRIVE_CONFIG.sources`                       |
| Change refresh frequency      | `DRIVE_CONFIG.refreshInterval` (seconds)     |
| Rotate Drive API key          | `DRIVE_CONFIG.apiKey`                        |
| Change role passwords         | `PASSWORDS`                                  |

After editing, commit + push to `main`. Vercel redeploys in ~30–60 seconds.

---

## Roadmap (suggestions, not committed)

* localStorage persistence for manual entries (currently lost on refresh)
* Per-machine overdue/due-soon badges in the sidebar
* Real backend (Supabase) replacing hardcoded passwords
* Auto-create Asana/Jira task when a doc enters the 30-day review window
* "Standard Work card" generator: 1-page printable per machine
* Training matrix view (operators × machines × required docs)
* 5S audit module on the same machine list
