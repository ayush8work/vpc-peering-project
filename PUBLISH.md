# How to publish (5 minutes, no installs needed)

## A. Get the GitHub link
1. Go to **github.com** and sign in (create a free account if needed).
2. Click **+ (top right) ▸ New repository**.
   - Repository name: `vpc-peering-project`
   - Visibility: **Public** (required for free GitHub Pages)
   - Do **not** tick "Add a README" (we already have one).
   - Click **Create repository**.
   - *This is your* **GitHub link**: `https://github.com/<username>/vpc-peering-project`

## B. Upload the project
3. On the new (empty) repo page, click the link **"uploading an existing file"**.
4. **Drag-and-drop the whole content of the `vpc-peering-project` folder** — `index.html`, `PUBLISH.md`, `README.md`, `RUNBOOK.md`, the `css/`, `js/`, `scripts/`, `report/` folders, and your `img/` evidence screenshots.
5. Commit message: `VPC peering project - initial release`, click **Commit changes**.

## C. Get the live link (GitHub Pages)
6. In the repo: **Settings ▸ Pages** (left sidebar).
7. Under "Build and deployment": Source = **Deploy from a branch**, Branch = **main**, Folder = **/(root)** → **Save**.
8. Wait 1–2 minutes, refresh the Pages page; at the top it shows:
   `Your site is live at https://<username>.github.io/vpc-peering-project/`
   - *This is your* **live project link** ✅

## D. Before you submit — small edits
- Replace `<YOUR_GITHUB_USERNAME>` with your GitHub username in: `index.html` (hero button + footer), `README.md` top line, and the report. (Do it locally before upload, or edit directly on github.com with the ✏ pencil button.)
- Verify the live site loads, then add both links to the report PDF.
- **Never upload `peering-lab.pem`** or any file containing the private key.

## Optional (if you install Git later)
```powershell
git init
git add .
git commit -m "VPC peering project"
git branch -M main
git remote add origin https://github.com/<username>/vpc-peering-project.git
git push -u origin main
```
