# 3D dashboard update

Adds a live, interactive 3D dashboard at http://localhost:5173/admin3d (admin and bank logins).
It uses your real backend data. Nothing else in the project changes.

## Apply (easiest): copy files over
1. Close the servers (Ctrl+C in each terminal).
2. Unzip this pack INTO your `neverfrozen` project folder, so paths match, and choose **Replace** when asked.
   Files touched: backend/app/main.py, backend/tests/test_api.py, frontend/src/{api.js, App.jsx},
   frontend/src/pages/Admin.jsx. New: frontend/src/pages/Dashboard3D.jsx, frontend/src/dashboard3d.css.
   (If you edited those 5 files yourself, use the patch below instead of replacing.)
3. Restart backend and frontend as before. No new packages are needed.

## Apply (alternative): patch file
From the project root, with git installed:  `git apply 3d-dashboard-existing-files.patch`

## Use it
Log in as admin@neverfrozen.dev / Admin@123 -> click **Open 3D view**.
Click **Run AI scan**, drag the bar chart, move the simulator sliders, click table rows.
To see ledger blocks, first complete one KYC renewal as demo@neverfrozen.dev.
