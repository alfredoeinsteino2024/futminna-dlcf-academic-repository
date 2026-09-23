# FUTMINNA DLCF Academic Resource Repository

A mobile-first web portal for the Deeper Life Campus Fellowship (DLCF), Federal University of Technology, Minna. Academic reps upload lecture notes, slide packs, past questions and textbooks, and the files sort themselves into a structured Google Drive repository.

## Features

- Bulk upload with drag and drop, file preview and per-file removal.
- Files up to 100MB, sent in 3MB chunks with retries and a live progress bar.
- Paste a Google Drive or Google Docs link when a student has no file to upload. It is saved in the repository as a Drive shortcut.
- Automatic folder routing: School, then SEET/SIPET division (engineering only), Department, Academic Level.
- File naming: `[CourseCode] OriginalFileName`, or just the file name when no course code is given.
- Director admin page to add several storage locations and switch which one receives new uploads.
- Optional upload passcode and a separate admin passcode.

## Project structure

```
futminna-dlcf-academic-repository/
├── web/
│   ├── index.html        Upload portal (Tailwind CSS, no build step)
│   ├── admin.html        Director page for storage locations
│   └── dlcf-logo.png     Full-size logo (a small copy is embedded in index.html)
├── apps-script/
│   └── Code.gs           Google Apps Script backend
└── README.md
```

Deploy only the `web/` folder to Vercel. Keep `Code.gs` out of it, because anything inside `web/` is publicly downloadable.

## How it works

The browser reads each file in 3MB slices and sends them as Base64 JSON to a Google Apps Script Web App. The script creates any missing folders, opens a Drive resumable-upload session and forwards each slice to it, so no single request is large. Requests are sent as plain text so the browser skips the CORS preflight that Apps Script cannot answer.

## Setup

### 1. Google Drive

Create a folder for the repository and copy its ID (the text after `/folders/` in the URL).

### 2. Apps Script backend

1. Go to [script.google.com](https://script.google.com), create a new project and paste in `apps-script/Code.gs`.
2. Set the constants at the top:
   - `PARENT_FOLDER_ID`: the folder ID from step 1.
   - `ADMIN_PASSCODE`: required to use `admin.html`. Leave empty to disable the admin page.
   - `UPLOAD_PASSCODE`: optional. When set, uploaders must enter it in the form.
3. Run `doGet` once from the editor and approve the permissions.
4. Choose Deploy, then New deployment, then Web app.
   - Execute as: **Me**
   - Who has access: **Anyone**
5. Copy the Web App URL ending in `/exec`.

After any later change to `Code.gs`, redeploy with Deploy, Manage deployments, Edit, New version. The URL stays the same.

### 3. Frontend

1. Paste the `/exec` URL into `SCRIPT_URL` in `web/index.html` and `web/admin.html`.
2. Deploy to Vercel:

```bash
npm i -g vercel
cd web
vercel          # framework: Other, no build command
vercel --prod
```

### 4. Test

Upload a small PDF, then a file over 50MB, then a pasted Drive link. Check that the files land in the expected School, Department and Level folders.

## Managing storage locations

Open `/admin.html`, enter the admin passcode and press Load. You can add a location and choose which one is active. New uploads use the active location straight away.

Files are owned by the Google account that deployed the script, so they count against that account's storage. A second folder on the same account does not add space. To get more room:

- Add another Google account: that account deploys its own copy of `Code.gs` with its own `PARENT_FOLDER_ID` and the same `UPLOAD_PASSCODE`, and you add its Web App URL as a location.
- Or use a folder on a Google Workspace Shared Drive, which uses the organisation's quota.

Switching locations builds the same folder tree in the new location. Files already saved stay where they are.

## Security notes

- Never commit real passcodes or your real folder ID. Keep placeholders in the repository and set the real values only in the Apps Script editor.
- "Anyone" access means anyone with the Web App URL can call it. `UPLOAD_PASSCODE` is what limits who can upload, so set it.
- Anyone with the admin passcode can change where files are stored. Choose a strong one and do not share it.
- Consider keeping the repository private, since the Web App URL is visible in `index.html`.

## Limits

- Maximum file size: 100MB (`MAX_FILE_BYTES` in both `index.html` and `Code.gs`).
- Apps Script quotas apply. Very heavy use in one day can hit daily URL Fetch limits.
- A pasted link is saved as a shortcut only if the script owner can open the target. Otherwise it is saved as a small text note holding the link.

## Troubleshooting

- **Google sign-in page instead of JSON:** "Who has access" is not set to Anyone.
- **"Failed to fetch":** wrong or missing `SCRIPT_URL`, or the script was not redeployed after an edit.
- **"Incorrect upload passcode":** the value entered does not match `UPLOAD_PASSCODE` in the script that is receiving the upload.
- **"Upload session expired":** a single file took over six hours or the tab was left idle. Upload that file again.

## Credits

Built for the DLCF FUTMINNA Academic Directorate.
