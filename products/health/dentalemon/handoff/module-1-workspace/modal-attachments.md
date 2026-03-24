---
screen: Attachments
type: modal
route: "N/A (sheet within /workspace/:patientId)"
module: "Module 1: Dental Workspace"
tier: 1
components:
  - Sheet
  - SheetHeader
  - Button
  - Card
wireframe: inline-ascii
---

## Modal: Attachments (of Dental Workspace)

**Route:** N/A (sheet within `/workspace/:patientId`)
**POV:** A dentist attaching intraoral photos, X-rays, referral letters, or other files to a patient's record during a session.

### User Story

> As a dentist, I want to attach photos, X-rays, and documents to a patient record so that diagnostic files are stored with the clinical data.

### CTAs

| CTA | Type | Action | Variant |
|-----|------|--------|---------|
| Upload File | Primary | Opens file picker / camera | `default` |
| Done | Ghost | Closes sheet | `ghost` |
| 🗑 (delete) | Ghost | Removes an attached file | `ghost` |

### ShadCN Components

| Component | Usage | Props/Variant |
|-----------|-------|---------------|
| `Sheet` | Bottom sheet container (slides up from bottom) | `side='bottom'` |
| `SheetHeader` | "Attachments" title | — |
| `Button` | "Upload File" CTA (opens file picker or camera) | `variant='default'` |
| `Card` | Attachment preview card (thumbnail + filename + file size + date) | — |
| `Button` | Delete icon per attachment | `variant='ghost'`, `size='icon'` |

### Key Interactions

- **Upload** → opens device file picker or camera. Supports: images (JPEG, PNG), PDFs, DICOM (X-rays).
- **Thumbnail preview** → uploaded files show as cards with thumbnail, filename, size, and upload date.
- **Delete attachment** → triggers an AlertDialog: "Delete [filename]? This cannot be undone." with Cancel + Delete (destructive) buttons. Confirmation required for ALL deletions (clinical files like X-rays and signed consent PDFs warrant consistent safeguarding, regardless of age).
- **Tap attachment card** → opens full-screen preview of the file.
- **Files are stored locally** → synced to cloud backup when connectivity available.

### Layout

1. **Header** — `px-6 py-4 flex justify-between border-b`. "Attachments" title + ✕ close.
2. **Upload Zone** — `px-6 py-4`. "Upload" outline button + drag-and-drop area (`h-[80px] border-dashed border-2 rounded-lg flex items-center justify-center`): "Drag files here or click Upload."
3. **Attachment Grid** — `px-6 py-4 grid grid-cols-3 gap-4`. Cards for each attachment: file icon + filename + file size + timestamp + overflow ⋯ (View, Download, Delete).
4. **Footer** — `px-6 py-3 border-t`. File count: "N attachments" (left). Storage info if applicable (right).

### Screen States

| State | Behavior |
|-------|----------|
| **Loading** | Shimmer cards while loading existing attachments. |
| **Empty** | Upload icon + "No attachments yet. Upload photos, X-rays, or documents." + Upload File button. |
| **Error** | Toast: "Upload failed" + Retry. Existing attachments remain visible. |

### CRUD Interactions

#### Adding (upload)
- **Trigger:** "Upload File" button
- **Opens:** Device file picker or camera
- **Validation:** File type must be image, PDF, or DICOM. Max file size: 25MB.
- **On success:** Attachment card appears in the list with thumbnail preview.
- **On cancel:** File picker closes, no change.

#### Deleting
- **Trigger:** Delete icon on attachment card
- **Confirmation:** AlertDialog always: "Delete [filename]? This cannot be undone." with Cancel + Delete (destructive).
- **On success:** Card removed from list. Toast: "Attachment removed."

### What This Modal Does

- Uploads and stores photos, X-rays, referral letters, and documents
- Displays attachment previews with thumbnails
- Supports camera capture for intraoral photos
- Manages file attachments per patient record

### What This Modal Does NOT Do

- Does NOT organize files into folders or categories — flat list per patient
- Does NOT support DICOM viewing — files are stored but require external viewer
- Does NOT integrate with email or X-ray services — Phase 2 (Dentrix X-ray email feature)

### Wireframe

```
┌─────────────────────────────────────────┐
│ Attachments                             │
├─────────────────────────────────────────┤
│  ┌──────┐  ┌──────┐  ┌──────┐          │
│  │ 📷   │  │ 📄   │  │ + Upload │      │
│  │thumb │  │thumb │  │  File   │       │
│  │      │  │      │  │         │       │
│  └──────┘  └──────┘  └─────────┘       │
│  xray.jpg   ref.pdf                     │
│  2.1 MB     340 KB                      │
│  03/24/26   03/24/26                    │
│                                         │
│                          [Done]         │
└─────────────────────────────────────────┘
```
