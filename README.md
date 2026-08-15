# HR Management System

A web-based Human Resources Management System that centralises employee records, attendance, leave, payroll, performance, benefits, training, and recruitment in a single application. Attendance can be recorded through facial recognition from a webcam feed, and staff communicate through a built-in real-time chat.

This was my end-of-studies project (Projet de Fin d'Études), built over four months from February to June 2024.

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-3.0-000000?logo=flask&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?logo=mysql&logoColor=white)
![Socket.IO](https://img.shields.io/badge/Socket.IO-010101?logo=socketdotio&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue)

## Table of contents

- [Modules](#modules)
- [User roles](#user-roles)
- [Tech stack](#tech-stack)
- [Screenshots](#screenshots)
- [Getting started](#getting-started)
- [Configuration](#configuration)
- [Facial-recognition attendance](#facial-recognition-attendance)
- [Project structure](#project-structure)
- [Security notes](#security-notes)
- [Roadmap](#roadmap)
- [License](#license)

## Modules

- **Employee management** — create, update, and search employee records, organised by department and role.
- **Departments and roles** — define departments and role-based permissions that drive what each user can see and do.
- **Attendance** — record attendance manually or through facial recognition from a live camera feed.
- **Leave management** — submit leave requests, track approval status, and manage per-employee leave balances.
- **Payroll** — store and manage payroll records linked to employees.
- **Performance** — performance reviews, goal setting, and reporting, including a dedicated manager review flow.
- **Benefits** — publish benefit programs and let employees select the ones they want to enrol in.
- **Training** — publish training programs and manage employee enrolment.
- **Recruitment** — post jobs and collect applications through a public application page.
- **Communication** — real-time chat between employees over WebSockets, with channels and message history.
- **Calendar** — an integrated calendar view for scheduling.
- **Audit trail** — access logs and an audit trail of key actions.

## User roles

The interface adapts to four roles, each with its own set of pages:

| Role | Scope |
|------|-------|
| Admin | Full system configuration and oversight |
| HR | Employee, payroll, leave, benefits, training, and recruitment management |
| Manager | Team performance reviews and approvals |
| Regular employee | Personal profile, attendance, leave requests, benefit and training selection, chat |

## Tech stack

| Layer | Technology |
|-------|------------|
| Language | Python 3.10+ |
| Framework | Flask 3 |
| ORM | Flask-SQLAlchemy / SQLAlchemy 2 |
| Database | MySQL / MariaDB (via PyMySQL) |
| Authentication | Flask-Login (sessions) and Flask-JWT-Extended (tokens) |
| Password hashing | bcrypt |
| Real-time | Flask-SocketIO |
| Facial recognition | face_recognition (dlib) and OpenCV |
| Templating | Jinja2 |

## Screenshots

> Add a few screenshots so the repository page shows the interface at a glance. Put image files under `docs/screenshots/` and reference them here, for example:
>
> ```markdown
> ![Login](docs/screenshots/login.png)
> ![HR dashboard](docs/screenshots/hr-dashboard.png)
> ```
>
> Use screenshots with placeholder or test data only — do not publish real employees' names, photos, or records.

## Getting started

### Prerequisites

- Python 3.10 or newer
- MySQL or MariaDB
- A working C/C++ build toolchain and CMake (required to build `dlib` for facial recognition)
- A webcam, if you want to use facial-recognition attendance

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/hr-management-system.git
cd hr-management-system

# 2. Create and activate a virtual environment
python -m venv venv
# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure environment variables
cp .env.example .env
# then edit .env (see Configuration below)

# 5. Create the database
#    Log in to MySQL and run: CREATE DATABASE systemhr;

# 6. Run the application (tables are created automatically on first run)
python app.py
```

The app starts on http://localhost:5000.

> **Note on `dlib`:** it compiles from source and needs CMake and a C++ compiler. On Windows, install "Build Tools for Visual Studio"; on Debian/Ubuntu, `sudo apt install build-essential cmake`. If you don't need facial recognition, you can remove `dlib`, `face-recognition`, and `opencv-python` from `requirements.txt` and skip the camera feature.

## Configuration

All configuration is read from a `.env` file (not committed). Copy `.env.example` to `.env` and set:

| Variable | Purpose |
|----------|---------|
| `SECRET_KEY` | Flask session signing key |
| `JWT_SECRET_KEY` | JWT signing key (keep it different from `SECRET_KEY`) |
| `DATABASE_URL` | SQLAlchemy connection string, e.g. `mysql+pymysql://user:pass@localhost/systemhr` |
| `UPLOAD_FOLDER` | Directory for uploaded files |
| `CORS_ORIGINS` | Allowed origins for cross-origin requests |

Generate strong secrets with:

```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

## Facial-recognition attendance

Attendance can be captured by matching a live webcam frame against stored face encodings. Reference images live in a `Faces/` directory (one subfolder per person), which the application encodes at runtime.

**This directory is intentionally excluded from version control.** It contains biometric data of real people, which must never be published. To try the feature locally, create the folder yourself and add your own reference images:

```
Faces/
└── firstname_lastname/
    ├── photo1.jpg
    └── photo2.jpg
```

If you demonstrate this feature, use your own photos or clearly consented test data, and keep the images on your machine only.

## Project structure

```
hr-management-system/
├── app.py                  # Application entry point, Socket.IO server, camera feed
├── database.py             # SQLAlchemy instance
├── requirements.txt
├── .env.example
├── Routes/                 # Feature blueprints (employees, leave, payroll, chat, ...)
├── database/
│   └── Models/             # SQLAlchemy models (Employee, Attendance, Payroll, ...)
├── Middleware/
│   └── Authorization.py    # Role-based access checks
├── utils/
│   └── passwordhash.py     # bcrypt hashing helpers
├── templates/
│   ├── admin/  HR/  Manager/  Regular-employee/  messages/
│   └── login.html, apply.html, ...
├── static/                 # CSS, JS, images
├── Faces/                  # Reference face images (NOT in version control)
└── uploads/                # Runtime uploads (NOT in version control)
```

## Security notes

This is an academic project. Password storage is handled correctly with bcrypt, but if you intend to reuse or deploy this code, harden the following first:

- **Externalise all secrets.** The session key, JWT key, and database credentials must come from environment variables, never from source. The included `.env.example` and `.gitignore` support this.
- **Use distinct, random keys** for Flask sessions and JWT signing, and rotate any key that was ever committed.
- **Restrict CORS** to known origins instead of `*`.
- **Disable debug mode** (`debug=True`) in any non-local environment.
- **Enforce strong passwords** and require a password change on first login instead of predictable defaults.
- **Serve uploads and biometric data from outside the web root** and store them encrypted at rest.
- **Comply with data-protection rules** (such as GDPR) before processing real biometric data; facial templates are sensitive personal data.

## Roadmap

Possible improvements:

- Move database schema management to migrations (Flask-Migrate / Alembic) instead of `create_all()`.
- Add automated tests.
- Replace the live-camera attendance loop with a check-in endpoint that accepts a single captured frame.
- Containerise with Docker for reproducible setup.

## License

Released under the MIT License. See [LICENSE](LICENSE) for details.
