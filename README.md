# UPI Shield 3

UPI Shield is a consent-based educational risk-advisory prototype. The React app uses **Supabase Auth and Postgres with RLS** for accounts and application data. Flask is retained only as a local Random Forest inference/report service; SQLite is no longer the active UI datastore.

## Windows 10 setup

Requirements: Python 3.11+ and Node.js. The Supabase project is already connected and the application tables/policies have been configured. In Supabase Auth URL Configuration, make sure the local preview origin and `/auth/callback` are allowed for email confirmation.

In a VS Code PowerShell terminal:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
py -m pip install -r requirements.txt
```

Place the provided synthetic CSV at `backend\data\upi_shield_synthetic_training_data.csv`. **The file was not present in this project checkout.** It must contain the `is_fraud` label and all required model fields in `backend\ml\train_model.py`. No records or model metrics are fabricated.

Train the model once to generate the persisted model and local model/preprocessing reports:

```powershell
py backend\ml\train_model.py
```

Start the local inference service in one terminal:

```powershell
py backend\app.py
```

Start the React/Vite app as usual in a second terminal. The frontend signs in through Supabase and stores profile, assessment, consent, and audit records in Supabase; the local Flask service supplies only model probability and local reports. Keep Flask running for risk assessments. Account creation uses the `register` Supabase Edge Function to create the user with `email_confirm: true`, then immediately signs in using the supplied password. No confirmation email or verification step is part of the UPI Shield flow. To grant an administrator role, update that user's `profiles.role` to `admin` through a trusted project-admin workflow—public sign-up cannot grant administrator privileges.

Run backend tests:

```powershell
py -m pytest backend\tests -q
```

## Data and privacy

Supabase Row Level Security restricts profiles, assessments, consent requests, and audit rows. A receiver's selected transaction summary is shared only after an explicit acceptance; withdrawal clears the shared snapshot. Receiver verification is optional at assessment time: users can uncheck the request, and the risk score is still calculated and saved from the submitted transaction features. A decline, unavailable receiver, or absent response does not alter that score. Receiver handles resolve through an authenticated database function without public profile listing. Never enter UPI PINs, OTPs, bank passwords, or payment credentials. The project does not connect to or execute UPI payments. Risk bands are prototype thresholds, not bank-approved rules. Synthetic metrics do not represent real-world UPI performance.

The SQL setup changed the connected Supabase database directly; no manual migration file is included. The signup rate limiter uses a private RLS-enabled table, and the Edge Function relies on Supabase-managed `SUPABASE_SERVICE_ROLE_KEY` server-side only. Existing records in an earlier local SQLite database are not automatically imported. See `backend/README.md` for the original Python model and test details.

The React app passes workspace TypeScript checks and a production Vite build. Python tests and model training were not run here because the CSV and Python runtime are unavailable in this workspace.
