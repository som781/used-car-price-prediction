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
   pre-trained KNN model (knn_model_for_encrypted_data.sav) loaded once at request time
              │
              ▼
   predicted price → rendered back in the result template
```

The login flow uses Django's auth system; predictions sit behind authentication.

There is also an **AES (PyCryptodome) layer wrapped around form fields**. To be honest: this was a learning exercise in calling crypto APIs from Django, not a deliberate security design. On a single-tenant localhost form, encrypting then decrypting fields in the same process protects against essentially nothing — and the key is in source, in ECB mode, both of which would be real bugs in production. It's documented here so a reader doesn't mistake it for thoughtful threat modeling.

## Tech stack

- **Backend / web:** Django 4.1
- **ML:** scikit-learn KNN classifier (`knn_model_for_encrypted_data.sav`), pandas, numpy, joblib
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
