# Django Blog Website

A full-stack blog platform with public-facing article browsing (by category,
keyword search, comments) and a custom admin dashboard for managing
categories, posts, and users — separate from Django's built-in `/admin/`.

## Features

- Public blog with home feed, featured posts, category pages, and keyword search
- User registration and login (Django's auth system)
- Comment system for logged-in users
- Custom staff dashboard (`/dashboard/`) to create/edit/delete categories, posts, and users
- Image uploads for post cover images, served from `/media/`
- Bootstrap-styled templates via `django-crispy-forms`

## Tech Stack

| Layer | Choice | Why |
|---|---|---|
| Language | Python 3.12 | Matches Django's requirements and the rest of the codebase; widely supported. |
| Framework | Django 5.1 | A content-heavy CRUD app (posts, categories, comments, users, auth, an admin-style dashboard) is exactly what Django is built for — it ships with auth, an ORM, a template engine, and form handling out of the box, so almost none of that had to be written by hand. |
| Database | SQLite (`db.sqlite3`) | Zero-config, file-based, and included with Python — ideal for a single-app project, local development, and portfolio/demo purposes. Django's ORM means swapping to PostgreSQL/MySQL later only requires changing `DATABASES` in `settings.py`, not rewriting queries. |
| Forms/UI | `django-crispy-forms` + `crispy-bootstrap4` | Renders Django forms with Bootstrap 4 markup automatically, instead of hand-writing `<div class="form-group">` wrappers for every field. |
| Image handling | `Pillow` | Required by Django's `ImageField` (used for blog cover images) to validate and process uploaded images. |
| Static/media serving | Django's built-in dev static/media serving | Fine for local development; in production this would be swapped for a CDN, S3, or served via Nginx/WhiteNoise. |

## Project Structure

```
Blog website/
├── blog_main/       # Project settings, root urls, home/login/register views
├── blogs/           # Public blog app: models (Category, Blogs, Comment), views, urls
├── dashboards/       # Staff-only CRUD dashboard app
├── templates/        # HTML templates (base layout, dashboard, public pages)
├── static/           # CSS/images
├── media/            # User-uploaded blog images (created at runtime)
├── db.sqlite3         # SQLite database (already seeded with sample data)
├── manage.py
└── requirements.txt
```

## Running the Project in VS Code.

### 1. Open the folder
Open the `Blog website` folder in VS Code (`File > Open Folder`).

### 2. Create and activate a virtual environment
Open a terminal in VS Code (`` Ctrl+` ``):

```bash
python -m venv venv
```

Activate it:
- **Windows (PowerShell):** `venv\Scripts\Activate.ps1`
- **Windows (cmd):** `venv\Scripts\activate.bat`
- **macOS/Linux:** `source venv/bin/activate`

In VS Code, also select this interpreter: `Ctrl+Shift+P` → **Python: Select Interpreter** → choose `./venv`.

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Apply migrations

The included `db.sqlite3` already has the schema applied and sample data
(6 categories, 19 blog posts, a superuser, and a few regular users), so this
step should report "No migrations to apply." Run it anyway to be safe:

```bash
python manage.py migrate
```

### 5. Create your own admin user (recommended)

The seeded superuser (`admin`) has a password you won't know. Create your
own to access `/admin/` and the dashboard:

```bash
python manage.py createsuperuser
```

### 6. Run the server

```bash
python manage.py runserver
```

Visit `http://127.0.0.1:8000/`. Log in at `/login/` and go to `/dashboard/`
to manage content, or `/admin/` for Django's built-in admin.

## Issues Found and Fixed

- **`requirement.txt` → `requirements.txt`**: renamed to the standard filename
  so `pip install -r requirements.txt` and most tooling/CI recognize it
  automatically.
- **Search page 500 error**: visiting `/blogs/search/` directly (i.e. without
  a `?keyword=` query string, which happens if a user hits that URL without
  going through the search box) crashed with `ValueError: Cannot use None as
  a query value`, because `request.GET.get('keyword')` returned `None` and
  was passed straight into an `icontains` filter. Fixed in `blogs/views.py`
  to default to an empty string and return no results instead of crashing.
- Removed the committed `venv/`/`env/` folder and `__pycache__` directories —
  these are environment-specific and already covered by `.gitignore`; they
  shouldn't be shipped with the source, and each machine should generate its
  own.

Everything else (models, URLs, templates, static/media config) checked out:
`python manage.py check` reports no issues, and the home, login, register,
category, and dashboard/admin (redirect-to-login) routes all return correct
status codes..

## Notes / Possible Next Steps

- `DEBUG = True` and a hardcoded `SECRET_KEY` are fine for local development
  but should be moved to environment variables before any real deployment.
- `TIME_ZONE` is set to `UTC` — change if you want post timestamps in a
  specific local time zone.
