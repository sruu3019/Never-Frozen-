# NeverFrozen: AI + Blockchain Predictive KYC Renewal

Predicts which customers will miss their periodic KYC renewal, nudges them on the channel they respond to,
re-verifies them digitally (OCR + face match + liveness) and records the result as a hash-only status on a
Solidity registry that partner banks can check.

```
neverfrozen/
├── backend/      FastAPI + SQLite + XGBoost models + verification + blockchain bridge
├── blockchain/   Hardhat project: KYCStatusRegistry.sol, tests, deploy script
├── frontend/     React (Vite) landing, login, register, customer, KYC flow, admin dashboards
└── .vscode/      one-click tasks
```

## 1. Install once (prerequisites)
| Tool | Version | Check |
|---|---|---|
| Python | 3.10 or newer | `python --version` |
| Node.js | 18 or newer (LTS) | `node --version` |
| VS Code | any | extensions: **Python**, **Solidity (Nomic Foundation)**, **ES7+ React snippets** |

## 2. Open in VS Code
`File > Open Folder...` and choose the `neverfrozen` folder. Open terminals with `Terminal > New Terminal`
(use the **+** button to open several; you will need 2 to 4).

## 3. Backend (Terminal 1)
**Windows (PowerShell)**
```powershell
cd backend
python -m venv .venv
.venv\Scripts\Activate.ps1          # if blocked: Set-ExecutionPolicy -Scope Process Bypass
pip install -r requirements.txt
python -m app.train                 # optional: trains models and prints accuracy/AUC
python -m uvicorn app.main:app --reload --port 8000
```
**macOS / Linux**
```bash
cd backend
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python -m app.train
python -m uvicorn app.main:app --reload --port 8000
```
Check http://127.0.0.1:8000/health and the API docs at http://127.0.0.1:8000/docs.
First start creates `neverfrozen.db`, trains models if missing and seeds 400 synthetic customers.

## 4. Frontend (Terminal 2)
```bash
cd frontend
npm install
npm run dev
```
Open **http://localhost:5173**.

## 5. Demo logins
| Role | Email | Password |
|---|---|---|
| Customer (KYC due in 12 days, disengaged) | demo@neverfrozen.dev | Demo@123 |
| Admin (compliance) | admin@neverfrozen.dev | Admin@123 |
| Bank officer (read only) | bank@neverfrozen.dev | Bank@123 |

**Demo script (5 minutes)**
1. Log in as **admin**, click **Run AI scan**: reminders are created for at-risk customers and the nudge optimiser table starts learning channel open-rates.
2. Log in as **customer** (demo): see health score, lapse probability with plain-language reasons, and the new notification. Click **Mark read**.
3. **Renew KYC now**: upload any ID image, start the camera, press Capture, blink and turn your head, then **Verify identity**.
4. Dashboard now shows *Verified* and a ledger record. Copy the customer hash.
5. Log in as **bank**, paste the hash into **Institution lookup** to show cross-institution verification.

## 6. Real blockchain (optional, Terminal 3 and 4)
Without this the backend uses a hash-chained SQLite ledger (the dashboard shows `simulated`).
```bash
cd blockchain
npm install
npx hardhat test                                      # runs the contract tests
npx hardhat node                                      # Terminal 3: leave running
npx hardhat run scripts/deploy.js --network localhost # Terminal 4: writes deployment.json
```
Restart the backend. `GET /health` should now report `"chain_mode": "blockchain"` and every verification
creates a real transaction. Polygon Amoy: create `blockchain/.env` with `DEPLOYER_KEY` (a throw-away test wallet
with test POL) and deploy with `--network amoy`; set `NF_RPC_URL` and `NF_PRIVATE_KEY` for the backend.
Hardhat downloads the Solidity compiler on first run, so it needs internet.

## 7. Optional: real OCR and face matching
The app runs in clearly labelled **demo mode** without these (the result screen shows `[demo]`).
* **OCR**: install Tesseract (Windows installer from UB-Mannheim, macOS `brew install tesseract`, Linux `apt install tesseract-ocr`), then `pip install pytesseract`.
* **Face match**: `pip install cmake` then `pip install face_recognition` (builds dlib; on Windows this needs the "Desktop development with C++" Build Tools). If it fails, skip it; demo mode still works.

## 8. Tests
```bash
cd backend && python -m pytest -q      # API, auth, roles, scan, KYC flow, ledger integrity
cd blockchain && npx hardhat test      # smart contract
```

## 9. How the pieces map to the synopsis
| Module | File |
|---|---|
| Risk tiering + lapse prediction (XGBoost) | `backend/app/ml.py`, `data_gen.py` |
| Nudge optimisation (epsilon-greedy bandit) | `ml.choose_channel`, `/admin/scan` in `main.py` |
| OCR, face match, liveness | `backend/app/verification.py` |
| Blockchain audit | `blockchain/contracts/KYCStatusRegistry.sol`, `backend/app/chain.py` |
| Dashboards | `frontend/src/pages/*.jsx` |

## 10. Honest limitations (say these in your viva before they are asked)
* All customer behaviour is **synthetic**; accuracy and "lapses avoided" figures come from simulated data and stated assumptions (15% vs 45% save-rate), not real banks.
* Liveness is a **prototype motion heuristic**, not certified anti-spoofing. Face/OCR need the optional libraries for real checks.
* Renewal periods follow RBI's risk-based 2 / 8 / 10 year rule for high / medium / low risk; confirm against the latest RBI Master Direction.
* New registrations get demo-default account features; in production the bank supplies them through secure APIs.
* The Hardhat key in `config.py` is a public test key. Never reuse it on a real network.
* SQLite is used for easy setup; swapping to PostgreSQL means replacing `db.py`.

## Troubleshooting
* `ModuleNotFoundError: app`: run uvicorn from inside the `backend` folder with the venv active.
* Frontend shows "Failed to fetch" or 500: backend is not running on port 8000.
* Port busy: change `--port`, and the proxy target in `frontend/vite.config.js`.
* Camera blocked: allow it in the browser address bar (works on `localhost`), or use the demo shortcut.


## Architecture note
The API service in this package is Python FastAPI (`backend/app/main.py`). Node.js/npm power the React/Vite frontend tooling and the Hardhat/Solidity blockchain workspace; this package does not contain a separate Express API server.
