# Risk Solution - Ready-to-deploy package

Files included:
- `index.html` — static landing page (put this at repo root)
- `Code.gs` — Google Apps Script to receive leads and email notifications
- This README with deployment steps

---

## Quick deploy steps (do exactly in this order)

### 1) Google Sheet
1. Open https://sheets.google.com and create a new Google Sheet.
2. Rename it `RiskSolutionLeads` (optional).
3. Copy the Sheet ID from URL (between `/d/` and `/edit`). Example: `1AbCdEfGh123...`

### 2) Apps Script setup
1. With the Sheet open, go to **Extensions → Apps Script**.
2. In the Apps Script editor, replace any default code with the content of `Code.gs`.
3. Replace `<<SPREADSHEET_ID>>` in `Code.gs` with your Sheet ID.
4. Save the script.

### 3) Deploy Web App (important)
1. In Apps Script editor, click **Deploy → New deployment**.
2. Choose **Deployment type: Web app**.
3. Set **Execute as:** Me
4. Set **Who has access:** Anyone (or Anyone, even anonymous)
5. Click **Deploy** and authorize when prompted.
6. Copy the **Web app URL** (starts with `https://script.google.com/macros/s/.../exec`). This is your `SCRIPT_URL`.

### 4) Add SCRIPT_URL to `index.html`
1. Open `index.html` and replace `<<SCRIPT_URL>>` with the Web app URL you copied.
2. Save/commit that change in your GitHub repo (root `index.html`).

### 5) GitHub Pages (host the site)
1. Ensure repository is **public**.
2. In GitHub repo -> Settings -> Pages choose **Deploy from a branch**:
   - Branch: `main`
   - Folder: `/ (root)`
3. Save. Wait a few minutes and visit `https://<your-username>.github.io/<repo>/` to see the site.

### 6) Test the form
1. Open the GitHub Pages site, fill the form and submit.
2. Check your Google Sheet for a new row and your email `shresthimishra8@gmail.com` for notification.

---
If you want, I can also generate a one-click shell script to automatically replace the `<<SCRIPT_URL>>` in `index.html` (for advanced users).