# Creating a ready-to-upload ZIP package containing index.html, Code.gs, and README for the user.

from pathlib import Path

import zipfile, os, json, textwrap



out_dir = Path("/mnt/data/risk_solution_package")

out_dir.mkdir(parents=True, exist_ok=True)



index_html = """<!doctype html>

<html lang="hi">

<head>

  <meta charset="utf-8" />

  <meta name="viewport" content="width=device-width,initial-scale=1" />

  <title>Risk Solution - Loan Enquiry</title>

  <style>

    body{font-family:Arial,Helvetica,sans-serif;padding:18px;background:#f5f5f5}

    .card{max-width:520px;margin:18px auto;padding:20px;background:#fff;border-radius:8px;box-shadow:0 2px 8px rgba(0,0,0,0.08)}

    input,select,textarea{width:100%;padding:10px;margin:8px 0;border:1px solid #ddd;border-radius:6px}

    button{background:#1a73e8;color:#fff;padding:10px 14px;border:none;border-radius:6px;cursor:pointer}

    .small{font-size:13px;color:#666}

    .hidden{display:none}

  </style>

</head>

<body>

  <div class="card">

    <h2>Risk Solution - Loan Enquiry Form</h2>

    <p class="small">सिर्फ 30 सेकंड में फॉर्म भरो — हमारी टीम जल्दी contact करेगी.</p>

    <form id="leadForm">

      <input name="name" placeholder="पूरा नाम" required />

      <input name="phone" placeholder="मोबाइल नंबर (10 digits)" pattern="[0-9]{10}" required />

      <input name="email" type="email" placeholder="Email (optional)" />

      <select name="loan_type" required>

        <option value="">लोन किसके लिए?</option>

        <option value="home">Home Loan</option>

        <option value="personal">Personal Loan</option>

        <option value="business">Business Loan</option>

        <option value="vehicle">Vehicle Loan</option>

      </select>

      <textarea name="message" placeholder="कौन सी मदद चाहिए (optional)"></textarea>



      <!-- Honeypot anti-spam -->

      <input type="text" name="hp_field" class="hidden" tabindex="-1" autocomplete="off" />



      <input type="hidden" name="source" value="risk-solution-site" />

      <button type="submit">Submit</button>

    </form>



    <div id="msg" style="margin-top:12px;display:none"></div>

  </div>



<script>

// IMPORTANT: Replace <<SCRIPT_URL>> with your Apps Script Web App URL after deployment

const SCRIPT_URL = '<<SCRIPT_URL>>';

const WA_LINK = 'https://wa.me/919369079665?text=Hello%20I%20applied%20for%20loan%20via%20Risk%20Solution';



function showMessage(html) {

  const el = document.getElementById('msg');

  el.style.display = 'block';

  el.innerHTML = html;

}



document.getElementById('leadForm').addEventListener('submit', async function(e){

  e.preventDefault();

  // basic honeypot and timing check

  const form = e.target;

  if (form.hp_field && form.hp_field.value) {

    return; // ignore spam bots

  }

  const fd = new FormData(form);

  const data = Object.fromEntries(fd.entries());



  try {

    const res = await fetch(SCRIPT_URL, {

      method:'POST',

      headers: {'Content-Type':'application/json'},

      body: JSON.stringify(data)

    });

    const j = await res.json();

    if (j.status === 'ok') {

      showMessage('Thanks! हमारी टीम जल्दी contact करेगी.<br><a href="' + WA_LINK + '" target="_blank">WhatsApp पर बात करें</a>');

      form.reset();

    } else if (j.status === 'ignored_spam') {

      showMessage('Submission ignored (spam suspected).');

    } else {

      showMessage('Kuch gadbad hui, try karo dobara.');

    }

  } catch(err){

    showMessage('Network error: ' + err);

  }

});

</script>

</body>

</html>

"""



code_gs = """// Code.gs

function doPost(e) {

  try {

    var ss = SpreadsheetApp.openById('<<SPREADSHEET_ID>>'); // <-- replace with your Sheet ID

    var sheet = ss.getSheetByName('Leads') || ss.insertSheet('Leads');

    if (sheet.getLastRow() === 0) sheet.appendRow(['Timestamp','Name','Phone','Email','LoanType','Message','Source','RawJSON']);



    var data = {};

    if (e.postData && e.postData.contents) {

      try { data = JSON.parse(e.postData.contents); } catch(err) { data = e.parameter || {}; }

    } else {

      data = e.parameter || {};

    }



    // simple spam hop

    if (data.hp_field && data.hp_field.trim() !== '') {

      return ContentService.createTextOutput(JSON.stringify({status:'ignored_spam'})).setMimeType(ContentService.MimeType.JSON);

    }



    sheet.appendRow([new Date(), data.name||'', data.phone||'', data.email||'', data.loan_type||'', data.message||'', data.source||'', JSON.stringify(data)]);



    var adminEmail = 'shresthimishra8@gmail.com';

    var subject = 'New Loan Lead: ' + (data.name || data.phone || '');

    var body = 'Name: ' + (data.name || '') + '\\nPhone: ' + (data.phone || '') + '\\nEmail: ' + (data.email || '') + '\\nLoan: ' + (data.loan_type || '') + '\\nMessage: ' + (data.message || '') + '\\nSource: ' + (data.source || '');

    MailApp.sendEmail(adminEmail, subject, body);



    return ContentService.createTextOutput(JSON.stringify({status:'ok'})).setMimeType(ContentService.MimeType.JSON);

  } catch (err) {

    return ContentService.createTextOutput(JSON.stringify({status:'error', message: err.toString()})).setMimeType(ContentService.MimeType.JSON);

  }

}

"""



readme_md = """# Risk Solution - Ready-to-deploy package



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

If you want, I can also generate a one-click shell script to automatically replace the `<<SCRIPT_URL>>` in `index.html` (for advanced users)."""



# Write files

(index_html_path := out_dir / "index.html").write_text(index_html, encoding="utf-8")

(code_gs_path := out_dir / "Code.gs").write_text(code_gs, encoding="utf-8")

(readme_path := out_dir / "README.md").write_text(readme_md, encoding="utf-8")



# Create zip

zip_path = Path("/mnt/data/risk_solution_package.zip")

with zipfile.ZipFile(zip_path, "w", compression=zipfile.ZIP_DEFLATED) as zf:

    for file in [index_html_path, code_gs_path, readme_path]:

        zf.write(file, arcname=file.name)



print("Created:", str(zip_path))
