# NeverFrozen — Final Integrated Project

This package keeps the existing NeverFrozen frontend, FastAPI backend, SQLite persistence, AI/risk modules, KYC capture flow, and Solidity/Hardhat audit contract. The interactive 3D admin dashboard is available at `/admin3d`; the original admin dashboard remains available.

## Stack note
The web API in this supplied project is **Python FastAPI**, not a Node/Express API. Node.js/npm are used by the React/Vite frontend and the Hardhat blockchain workspace. I have kept the supplied architecture rather than silently replacing the backend.

## Requirements
- Python 3.10+ (Python 3.11 is a practical choice for ML package compatibility)
- Node.js LTS and npm
- Internet access for initial dependency installation

## 1. Start the backend (VS Code terminal 1)
From the extracted project root:

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
python -m uvicorn app.main:app --reload --port 8000
```

If PowerShell blocks activation, run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass`, then activate again. Keep the terminal open. Open `http://127.0.0.1:8000/docs` to see the API. The root `/` may return `{"detail":"Not Found"}`; that is expected because the API does not define a homepage.

## 2. Start the frontend (VS Code terminal 2)
Open a second terminal at the project root:

```powershell
cd frontend
npm install
npm run dev
```

Open the URL printed by Vite, usually `http://localhost:5173`. Demo logins: `admin@neverfrozen.dev` / `Admin@123`; `demo@neverfrozen.dev` / `Demo@123`. Use synthetic data only.

## 3. Open the interactive dashboard
Sign in as the admin, then click **Open 3D view** on the admin page, or navigate to `http://localhost:5173/admin3d`. The view includes draggable/rotatable 3D bars, KPI cards, a what-if risk simulator, customer filters/sorting, a notification/nudge panel, hash-only tamper-evident ledger demo, and a renewal workflow. The dashboard requests real app data from the backend where endpoints are available; any clearly labelled demo/sample visual data should not be presented as live bank data.

## 4. Start the local blockchain (optional)
The dashboard/backend can be started without Hardhat. To demonstrate the Solidity contract in a local test network, open a third terminal:

```powershell
cd blockchain
npm install
npm test
npm run node
```

Keep the local node running. Open a fourth terminal and deploy:

```powershell
cd blockchain
npm run deploy
```

Use only local Hardhat test accounts. Never send real funds to them or use their private keys on a live network. Check `blockchain/scripts/deploy.js` for deployment output/configuration.

## 5. KYC capture
From a customer account, open the KYC flow, upload a test document image and allow browser camera access for a selfie capture. Browser camera access normally works on `localhost`. OCR/face comparison dependencies are optional in this prototype; without the extra system libraries, results can explicitly show demo-mode checks. Do not use real identity documents in an untrusted development environment.

## Troubleshooting
- `email-validator is not installed`: fixed in this package's `backend/requirements.txt`; rerun `pip install -r requirements.txt`.
- `GET /` gives 404: expected; open `/docs` instead.
- Frontend says failed to fetch: make sure backend is still running at port 8000 and then reload the Vite page.
- `npm` not recognized: install Node.js LTS and reopen VS Code.
- ML package install fails on Windows: use Python 3.11 in a fresh virtual environment; some optional face-recognition packages require native build tools.
- Port is occupied: stop the previous server with Ctrl+C or choose a free port and update the frontend proxy/config accordingly.

## Scope and safety
This is an academic prototype with synthetic demo customer records, not an integration with real banks or regulatory KYC providers. Camera capture, OCR, face match, liveness, risk estimates, and blockchain audit should be described according to the actual mode shown in the UI. The local hash-chain/ledger demo is not itself proof of a transaction being committed to a public blockchain. Never store raw ID images, selfies, face embeddings, or OCR text on-chain.
