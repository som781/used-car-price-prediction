# used-car-price-prediction

A **Django web app** that predicts the resale price of a used car from its specs (year, kilometers driven, fuel type, transmission, etc.). Trained regression model wrapped behind a login-gated form. Built in 2023 as a full-stack ML exercise.

> **Status:** archived prototype (2023). Kept as-is for reference.

## How it works

```
   user fills the form (year, fuel, km, transmission, …)
              │
              ▼
   Django view (prediction/views.py)
              │  preprocess + label-encode categoricals
              ▼
   serialized scikit-learn regressor (loaded once at request time)
              │
              ▼
   predicted price → rendered back in the result template
```

The login flow uses Django's auth system; predictions sit behind authentication. There's an AES layer (PyCryptodome) over user-submitted fields — more an exercise in adding crypto than a security-critical feature, since the key is in source.

## Tech stack

- **Backend / web:** Django 4.1
- **ML:** scikit-learn regressor (serialized to disk), pandas, numpy, joblib
- **Crypto:** pycryptodome AES-ECB (demo only — see caveat below)
- **Database:** SQLite (Django default)
- **Frontend:** Django templates + Bootstrap

## Project layout

```
car_prize/
├── manage.py
└── carapp/
    ├── prediction/        ML view: loads model, runs prediction
    ├── login/             auth views/templates
    ├── templates/         HTML templates
    ├── static/            CSS/JS/images
    ├── db.sqlite3         SQLite DB (created on first run)
    └── Requirement.txt    pinned deps
```

## Running

```bash
cd car_prize/carapp
python -m venv .venv && source .venv/bin/activate
pip install -r Requirement.txt
python manage.py migrate
python manage.py runserver
# → http://127.0.0.1:8000
```

Sign up via the login flow, then access the prediction form.

## Known limitations / caveats

- **AES key is hardcoded** in `prediction/views.py`. Fine for a learning project, would be a real bug in production — keys belong in environment variables / a secrets manager, and ECB mode shouldn't be used for anything sensitive.
- **No model training pipeline in this repo** — only the serving side. The model was trained separately.
- **No input validation / range checks** on the form fields.
- **No tests.**
