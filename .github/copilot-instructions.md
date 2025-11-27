# OctoFit Tracker - AI Coding Agent Instructions

## Project Overview

**OctoFit Tracker** is a fitness application for tracking student activities, team competitions, and leaderboards. Built with Django backend + React frontend + MongoDB.

**Key Goals:**
- User authentication and profiles
- Activity logging and tracking  
- Team creation and management
- Competitive leaderboard
- Personalized workout suggestions

## Architecture

```
octofit-tracker/
├── backend/           # Django REST API + MongoDB (ORM via djongo)
│   ├── venv/         # Python virtual environment
│   └── octofit_tracker/  # Django project
└── frontend/          # React app (Bootstrap + React Router)
```

**Service Boundaries:**
- **Backend** (Django + djongo): RESTful API on port 8000, MongoDB database via Django ORM
- **Frontend** (React): Client app on port 3000
- **Database** (MongoDB): Port 27017 (private), accessed only through Django ORM
- **CORS**: Configured via `django-cors-headers` for frontend communication

## Critical Developer Workflows

### Never change directories in agent mode
Instead, reference directories in commands: `npm run build --prefix octofit-tracker/frontend`

### Setting Up Backend
```bash
# Create virtual environment (one-time)
python3 -m venv octofit-tracker/backend/venv

# Activate and install requirements
source octofit-tracker/backend/venv/bin/activate
pip install -r octofit-tracker/backend/requirements.txt
```

### Setting Up Frontend
```bash
# Bootstrap is already configured via installer
npm install --prefix octofit-tracker/frontend
npm start --prefix octofit-tracker/frontend  # Dev server on :3000
```

### Testing APIs
Use `curl` to test Django REST endpoints. Example:
```bash
curl http://localhost:8000/api/
```

### Forwarded Ports (Codespace)
- **8000 (public)**: Django backend API
- **3000 (public)**: React frontend dev server
- **27017 (private)**: MongoDB (no external access)

Do not propose other ports or make 27017 public.

## Backend-Specific Patterns

**Django Settings (`settings.py`):**
- Must include dynamic `ALLOWED_HOSTS` for Codespace environment:
  ```python
  import os
  ALLOWED_HOSTS = ['localhost', '127.0.0.1']
  if os.environ.get('CODESPACE_NAME'):
      ALLOWED_HOSTS.append(f"{os.environ.get('CODESPACE_NAME')}-8000.app.github.dev")
  ```

**URLs (`urls.py`):**
- Detect Codespace and use appropriate base URL for API root:
  ```python
  codespace_name = os.environ.get('CODESPACE_NAME')
  base_url = f"https://{codespace_name}-8000.app.github.dev" if codespace_name else "http://localhost:8000"
  ```

**Serializers:**
- Convert MongoDB `ObjectId` fields to strings in DRF serializers (required for JSON compatibility)

**Database:**
- Use Django ORM exclusively via djongo for database operations
- **Never** write direct MongoDB scripts; let Django handle schema management
- Check MongoDB service: `ps aux | grep mongod`
- Use MongoDB official tools: `mongodb-org` (package) and `mongosh` (client)

## Frontend-Specific Patterns

**Bootstrap Integration:**
- Import `bootstrap/dist/css/bootstrap.min.css` at the top of `src/index.js`
- Bootstrap grid, components, and utilities available throughout app

**Routing:**
- Use `react-router-dom` for single-page navigation

**Assets:**
- OctoFit app logo: `docs/octofitapp-small.png`

## Key Dependencies

**Backend:**
- Django 4.1.7, djangorestframework 3.14.0, django-allauth (auth), dj-rest-auth, djongo (MongoDB ORM), pymongo 3.12

**Frontend:**
- React (via create-react-app), bootstrap, react-router-dom

**Database:**
- MongoDB with djongo ORM layer (no direct scripts)

## Implementation Notes

1. **Error Handling:** Always validate user input in both API (serializers) and frontend (form validation)
2. **Authentication:** Use `django-allauth` and `dj-rest-auth` for JWT token-based auth flow
3. **CORS:** Configure in Django settings if frontend origin differs from backend domain
4. **Environment Detection:** Codespace environment variable `CODESPACE_NAME` determines dynamic URLs and host allowlists
5. **File Structure:** Keep Django apps modular (models, views, serializers, urls per app); keep React components organized by feature
