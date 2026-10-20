# Gated Resource Library — Setup Guide

A landing page where visitors sign up (name + email) to unlock downloadable guides.
Sign-ups and downloads are saved to a Google Sheet through Google Apps Script, and the page is hosted for free on GitHub Pages.

```
your-repo/
├── index.html        ← the landing page
├── files/            ← put your PDFs, templates, etc. here
│   ├── starter-guide.pdf
│   └── ...
└── README.md
```
`Code.gs` goes into Google Apps Script (not into GitHub).

---

## Step 1 — Create the Google Sheet + script

1. Go to [sheets.new](https://sheets.new) and name the sheet (e.g. "Library Sign-ups").
2. In the menu: **Extensions → Apps Script**.
3. Delete the sample code and paste everything from `Code.gs`. Click **Save**.
4. In the function dropdown at the top, choose **`setup`** and click **Run**.
   Google will ask for permission → **Review permissions → your account → Advanced → Go to project → Allow**.
5. Back in your sheet you'll now see two tabs: **Signups** and **Downloads**.

## Step 2 — Deploy the script as a web app

1. In Apps Script click **Deploy → New deployment**.
2. Click the gear ⚙ → **Web app**.
3. Set:
   - **Execute as:** Me
   - **Who has access:** Anyone
4. Click **Deploy** and copy the **Web app URL** (ends in `/exec`).
5. Paste that URL into a browser — you should see `{"ok":true,...}`.

> ⚠️ If you change `Code.gs` later, use **Deploy → Manage deployments → ✏️ Edit → Version: New version → Deploy**. That keeps the same URL.

## Step 3 — Configure the landing page

Open `index.html` and edit the two marked sections in the `<script>`:

```js
const SCRIPT_URL = "https://script.google.com/macros/s/XXXX/exec";

const GUIDES = [
  { id: "starter-guide", title: "The Starter Guide", desc: "...",
    category: "Guides", type: "PDF", size: "2.1 MB",
    file: "files/starter-guide.pdf" },
  // add as many as you like
];
```

- `id` must be unique (no spaces).
- `category` creates the filter buttons automatically.
- `file` can be a path in your repo (`files/...`) or any full link (Google Drive, Dropbox).
  For Google Drive, use a direct-download link: `https://drive.google.com/uc?export=download&id=FILE_ID`.

Also change **YourBrand**, the headline text and the colors at the top of the `<style>` block.

## Step 4 — Publish on GitHub Pages

1. Create a new **public** repository on [github.com/new](https://github.com/new).
2. Upload `index.html`, `README.md` and your `files/` folder (**Add file → Upload files**), or with git:
   ```bash
   git init
   git add .
   git commit -m "Launch resource library"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
   git push -u origin main
   ```
3. In the repo go to **Settings → Pages**.
4. Under **Source** choose **Deploy from a branch → main → / (root)** → **Save**.
5. After ~1 minute your site is live at `https://YOUR_USERNAME.github.io/YOUR_REPO/`.

## Step 5 — Test it

1. Open your live site and click any download.
2. Fill in the form → the file downloads and all cards unlock.
3. Check your Google Sheet — a new row appears in **Signups**, and each download is logged in **Downloads**.

Returning visitors stay unlocked (saved in their browser). "Sign out" in the top-right resets it, which is handy for testing.

---

## How it works

- The form sends a POST to your Apps Script URL. The script validates the email, skips duplicates, and appends a row.
- Every download is also logged, so you can see which guides are most popular.
- A hidden "honeypot" field blocks basic spam bots; inputs are sanitized against formula injection.

## Good to know

- **The gate is a soft gate.** It's great for collecting leads, but files in a public GitHub repo can be reached by anyone who has the direct link. If a file must be truly private, host it in Google Drive (restricted) and have the script email it to the user instead.
- **Privacy:** if you have visitors in the EU/UK or California, add a short privacy policy link near the form and only email people who ticked the consent box.
- **Custom domain:** add it under **Settings → Pages → Custom domain**.

## Troubleshooting

| Problem | Fix |
|---|---|
| No rows in the sheet | Check `SCRIPT_URL`; make sure access is set to **Anyone**; redeploy as a **New version**. |
| "Setup needed" error on the form | You haven't replaced `PASTE_YOUR_WEB_APP_URL_HERE`. |
| Download 404s | File path in `GUIDES` doesn't match the filename in `files/` (case-sensitive). |
| Site not updating | GitHub Pages can take 1–2 minutes; hard-refresh the page. |
