# Employee Attendance & Analytics API

A FastAPI + MongoDB backend service implementing employee attendance tracking, regularization, and department analytics per the HROne engineering specification.

## Tech Stack
- **Python**: 3.11+
- **Framework**: FastAPI + Uvicorn
- **Database**: MongoDB 6.0+ (Tested against MongoDB 7.0)
- **Driver**: PyMongo

## Setup & Running

1. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

2. **Environment configuration**:
   Set `MONGO_URI` and `MONGO_DB` in your environment or via a local `.env` file (real environment variables take priority):
   ```bash
   export MONGO_URI="mongodb://localhost:27017"
   export MONGO_DB="attendance_db"
   ```

3. **Optional - Seed sample data**:
   ```bash
   python sample_seed.py
   ```

4. **Start the server**:
   ```bash
   uvicorn app.main:app --port 8000
   ```

## Key Features & Guarantees
- **Startup Indexing**: All required indexes on `employees` and `attendance_logs` are created automatically and idempotently on application startup.
- **Race Safety**: Atomic updates and unique constraints ensure concurrent punch-in, punch-out, and regularization requests yield correct HTTP status codes (409 for conflicts) with zero race-condition data loss.
- **Pure Database Analytics**: All monthly summaries, department summaries, late-comers leaderboards (with competition ranking), and daily trends (with gap-filling and moving averages) are executed as MongoDB aggregation pipelines backed by index scans (`IXSCAN`).
- **Explain Endpoint**: `GET /admin/explain/{endpoint}` exposes raw `executionStats` demonstrating indexed execution across all primary queries and aggregation pipelines.

## Project Structure
- `app/main.py`: Complete application code with all endpoints and business logic.
- `requirements.txt`: Python package dependencies.
- `REVIEW.md`: Comprehensive review of starter code defects, symptoms, and fixes.
- `DECISIONS.md`: Concise explanation of architectural decisions, indexes, and race handling.
- `openapi.yaml`: API contract specification.
- `DATA_MODEL.md`: MongoDB schema documentation.
