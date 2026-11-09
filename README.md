# CreatorIQ – Creator Analytics & Content Performance Dashboard

## Project Overview

CreatorIQ is a FastAPI + PostgreSQL analytics platform with a React dashboard. It stores creator content, audience, growth, revenue and sponsorship data, calculates reusable analytics, supports mock multi-platform synchronization, and provides YouTube integration when an API key is available.

The implementation follows the project brief: data flows from platform/source → transformation → PostgreSQL → analytics services → FastAPI → React dashboard. Analytics values are not hard-coded in the frontend.

---
## Architecture

```text
YouTube API / Manual Platform Data
              ↓
      Platform Services
              ↓
       Common Data Format
              ↓
          PostgreSQL
              ↓
       Analytics Services
              ↓
           FastAPI
              ↓
        Axios / React
              ↓
       CreatorIQ Dashboard
```

## Modules

- User registration/login and creator profile APIs
- Content CRUD and engagement analytics
- Audience CRUD, demographic reports and device usage
- Growth history, daily growth and chart-ready trends
- KPI summary, top content and platform comparison
- Revenue and sponsorship management
- Notifications and read/unread tracking
- PDF and Excel report generation
- Mock multi-platform synchronization for YouTube, Instagram, Facebook, LinkedIn, TikTok and X
- Optional live YouTube Data API synchronization with duplicate-safe upsert
- React dashboard with KPI cards, charts, platform selector and top-content table

## Database tables

`users`, `content`, `audience`, `growth`, `revenue`, `sponsorships`, `notifications`

The core analytics tables required by the sprint are `users`, `content`, `audience`, and `growth`.

## Backend setup – Windows PowerShell

### 1. PostgreSQL

Create a PostgreSQL database named `creatoriq` and a user/password matching your local `.env`, or edit `DATABASE_URL`.

Default example:

```text
postgresql://creatoriq:creatoriq@localhost:5432/creatoriq
```

This project uses a direct PostgreSQL connection. Docker is not required.

### 2. Python environment

From the project root:

```powershell
py -3.14 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If PowerShell blocks activation, use:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

### 3. Configure `.env`

`.env` is intentionally ignored by Git. Set your real local PostgreSQL password and, only if using live YouTube synchronization, set:

```text
YOUTUBE_API_KEY=your_key_here
```

Never commit `.env`, API keys, passwords or access tokens.

### 4. Create tables

Use either SQLAlchemy table creation:

```powershell
python scripts\create_tables.py
```

or Alembic:

```powershell
alembic upgrade head
```

Do not run an initial Alembic migration against a database that already contains the same tables unless you have reviewed the migration state.

### 5. Optional demo data

To create enough realistic records for charts, comparisons and reports:

```powershell
python scripts\seed_demo.py
```

This creates multi-platform content, 30 days of growth history, audience data, revenue and a sponsorship for `creator_id=1`.

### 6. Start FastAPI

```powershell
python -m uvicorn app.main:app --reload
```

Swagger:

`http://127.0.0.1:8000/docs`

OpenAPI JSON:

`http://127.0.0.1:8000/openapi.json`

## Main APIs

### Content

- `POST /content`
- `GET /content`
- `GET /content/{id}`
- `PUT /content/{id}`
- `DELETE /content/{id}`

Validation rejects negative metrics and titles shorter than three characters.

### Engagement & dashboard analytics

- `GET /analytics/content/{id}/engagement`
- `GET /analytics/top-content`
- `GET /analytics/platform-performance`
- `GET /analytics/summary`
- `GET /analytics/chart/engagement`
- `GET /analytics/chart/followers`
- `GET /analytics/platform-comparison`

`engagement_rate = (likes + comments + shares + saves) / reach × 100`

### Audience & growth

- `POST/GET /audience`
- `GET/PUT/DELETE /audience/{id}`
- `GET /analytics/audience`
- `POST/GET /growth`
- `GET/PUT/DELETE /growth/{id}`
- `GET /analytics/growth`
- `GET /analytics/audience-trends`

### Revenue & sponsorships

- Revenue CRUD under `/revenue`
- `GET /revenue/analytics/summary`
- Sponsorship CRUD under `/revenue/sponsorships`

### Social workflow

Mock/sample workflow:

```text
POST /social/connect
        ↓
POST /social/sync?platform=Instagram&creator_id=1
        ↓
Common Content format
        ↓
PostgreSQL
        ↓
Analytics APIs
```

Live YouTube workflow:

```text
POST /social/youtube/sync
        ↓
YouTube Data API
        ↓
Transform to common CreatorIQ format
        ↓
Upsert by platform + external_content_id
        ↓
PostgreSQL
        ↓
Existing analytics APIs
```

The YouTube integration does not fabricate unavailable metrics. For example, shares, saves and watch time are not invented when the selected YouTube API response does not provide them.

### Notifications & reports

- `POST/GET /notifications`
- `PUT /notifications/{id}/read`
- `GET /reports/summary`
- `GET /reports/pdf`
- `GET /reports/excel`

## React frontend

From `frontend`:

```powershell
cd frontend
npm install
npm run dev
```

Open the URL printed by Vite, normally:

`http://localhost:5173`

Optional backend URL:

Create `frontend/.env` locally with:

```text
VITE_API_URL=http://127.0.0.1:8000
```

Do not commit frontend secrets.

The dashboard obtains KPI, chart, platform-comparison and top-content values through Axios from FastAPI. The platform selector sends a platform filter to the analytics APIs.

## Swagger testing checklist

1. Run `python scripts\create_tables.py`.
2. Run `python scripts\seed_demo.py`.
3. Start FastAPI.
4. Open `/docs`.
5. Test `POST /content` with the sprint sample.
6. Test content GET, GET by ID, PUT and DELETE.
7. Test engagement, top-content and platform-performance APIs.
8. Test audience and growth CRUD.
9. Test audience, growth and trend analytics.
10. Test revenue and sponsorship APIs.
11. Test `POST /social/sync?platform=Instagram&creator_id=1`.
12. Verify the new records using `GET /content`.
13. Test dashboard analytics APIs.
14. Test `/reports/pdf` and `/reports/excel`.
15. If a YouTube API key and valid channel ID are configured, test `/social/youtube/sync`.
16. Verify tables and inserted records in pgAdmin.

## pgAdmin verification

In pgAdmin, connect to the same PostgreSQL server/database used by `DATABASE_URL`, then open:

`Databases → creatoriq → Schemas → public → Tables`

Verify:

- users
- content
- audience
- growth
- revenue
- sponsorships
- notifications

For content verification, open `content → View/Edit Data → All Rows`.

## Git submission

Before committing:

```powershell
git status
git diff
```

Make sure `.env` and credentials are not staged.

Then:

```powershell
git add .
git commit -m "feat: integrate youtube analytics and complete milestone 2"
git push
```

## Important implementation notes

- PostgreSQL is the primary database.
- Business/analytics calculations live in service modules rather than being duplicated in routers.
- The content table uses `(platform, external_content_id)` as the duplicate-synchronization key when an external ID is available.
- Manual/sample platform data is stored in PostgreSQL and is consumed through the same analytics APIs as YouTube data.
- The React dashboard does not contain hard-coded analytics values.
- Real social APIs are not required for the mock synchronization workflow.
- YouTube credentials are loaded from `.env` and are never stored in Python source.

## 📂 Project Structure

```text
├── backend/
│   ├── app/
│   │   ├── api/            # API endpoints/routers (auth, users, content, analytics, social)
│   │   ├── core/           # Configuration, security, JWT helpers
│   │   ├── db/             # Database session setup and Base models
│   │   ├── models/         # SQLAlchemy ORM models
│   │   ├── schemas/        # Pydantic validation models
│   │   ├── services/       # Business logic and external API integrations
│   │   └── main.py         # FastAPI application entry point
│   ├── alembic/            # Database migrations
│   ├── Dockerfile
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/     # UI components & Recharts visualizations
│   │   ├── pages/          # Dashboard views (Analytics, Audience, Revenue)
│   │   ├── services/       # Axios API client routines
│   │   └── App.js
│   ├── Dockerfile
│   └── package.json
├── docker-compose.yml
└── README.md

```

## APIs Implemented

### Authentication
- `POST /auth/register` – Register a new user
- `POST /auth/login` – Authenticate and receive a JWT token

### Content
- `POST /content` – Create a content record
- `GET /content` – List all content records
- `GET /content/{id}` – Get a content record by ID
- `PUT /content/{id}` – Update a content record
- `DELETE /content/{id}` – Delete a content record

### Analytics
- `GET /analytics/content/{content_id}/engagement` – Engagement rate for a single content item
- `GET /analytics/top-content` – Top 5 content items ranked by engagement rate
- `GET /analytics/platform-performance` – Aggregated performance grouped by platform
- `GET /analytics/summary` – Dashboard KPI summary (views, likes, comments, shares, reach, followers, avg. engagement rate)
- `GET /analytics/chart/engagement` – Chart-ready engagement rate over time
- `GET /analytics/chart/followers` – Chart-ready follower growth over time
- `GET /analytics/platform-comparison` – Side-by-side comparison of all platforms

### Audience & Growth
- `POST /audience`, `GET /audience`, `GET /audience/{id}`, `PUT /audience/{id}`, `DELETE /audience/{id}` – Audience record CRUD
- `POST /growth`, `GET /growth`, `GET /growth/{id}`, `PUT /growth/{id}`, `DELETE /growth/{id}` – Growth record CRUD
- `GET /analytics/audience` – Audience demographic report (followers, reach, impressions, gender/age distribution, top countries/cities, device usage)
- `GET /analytics/growth` – 30-day growth report (daily growth and growth %)
- `GET /analytics/audience-trends` – Chart-ready follower/reach trend data

### Social Media
- `POST /social/connect` – Simulate connecting a social media account
- `GET /social/platforms` – List connected platforms
- `POST /social/youtube/sync` – Fetch real data from the YouTube Data API, transform it, and store/update it in PostgreSQL

---
## Data Transformation Workflow

Every platform's raw response is mapped into a single common format before being stored, so the analytics layer never needs platform-specific logic:

```
platform
external_content_id
content_title
views
likes
comments
shares
reach
published_date
```

This format is designed to be extensible — Instagram, TikTok, Facebook, LinkedIn, and X can be added later by writing a new `*_service.py` file that produces the same shape, with no changes required to the analytics APIs.

---

## Synchronization Workflow

`POST /social/youtube/sync?creator_id={id}&channel_id={channel}&max_results={n}`:

1. Validates `creator_id` and `channel_id`.
2. Calls the YouTube service to fetch and transform video data.
3. For each video, checks whether a matching content record already exists (by `external_content_id`).
   - **Exists** → update the existing record's metrics.
   - **Does not exist** → create a new content record.
4. Commits all changes to PostgreSQL in a single transaction (rolled back on failure).
5. Returns a summary: `records_synced`, `records_created`, `records_updated`, `records_skipped`.

This prevents duplicate content records when synchronization is run more than once.

---

## Testing Procedure

1. **Swagger UI** (`/docs`): used to manually test every endpoint listed above, including valid and invalid inputs (e.g., invalid `content_id`, invalid `channel_id`, missing required fields).
2. **pgAdmin**: after each write operation (content CRUD, audience/growth CRUD, YouTube sync), the corresponding table was inspected directly in pgAdmin to confirm the data was correctly persisted.
3. **End-to-end verification**: after running `POST /social/youtube/sync`, the existing analytics endpoints (`/analytics/summary`, `/analytics/top-content`, `/analytics/platform-comparison`, `/analytics/chart/engagement`, `/analytics/chart/followers`) were re-tested to confirm they correctly include the newly synchronized YouTube data without any YouTube-specific analytics logic.

---

## 🛠️ Tech Stack

* **Backend:** Python 3.10+, FastAPI, SQLAlchemy, Pydantic, Alembic, Celery, Redis
* **Frontend:** JavaScript, React.js, Tailwind CSS, Chart.js / Recharts, Axios
* **Database:** PostgreSQL (Primary), MongoDB (Secondary), Redis (Cache)
* **Authentication:** JWT, OAuth2
* **Integrations:** YouTube Data API, Instagram Graph API, TikTok API, Facebook Graph API, LinkedIn API

---
## ⚙️ Getting Started

```
### Prerequisites

* [Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/) installed
* Python 3.10+ (for local backend development)
* Node.js 18+ (for local frontend development)

### Running with Docker

1. **Clone the repository:**
```bash
git clone [https://github.com/springboardmentor330h/Creator-Analytics-Content-Performance-Dashboard.git](https://github.com/springboardmentor330h/Creator-Analytics-Content-Performance-Dashboard.git)
cd Creator-Analytics-Content-Performance-Dashboard

```

2. **Set up Environment Variables:**
Create a `.env` file in the root directory based on `.env.example`:
```env
DATABASE_URL=postgresql://user:password@db:5432/creatoriq_db
SECRET_KEY=your_jwt_secret_key
YOUTUBE_API_KEY=your_youtube_api_key

```

3. **Start services:**
```bash
docker-compose up --build

```

* **Frontend:** `http://localhost:3000`
* **Backend API Docs (Swagger):** `http://localhost:8000/docs`

---

## Author

Harsh Kumar — CreatorIQ Project
