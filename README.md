# Asthma Prediction Web Application

A full-stack web application that uses a pre-trained machine learning model to predict asthma risk based on patient health information, providing personalized recommendations and tracking prediction history.

## Tech Stack

- **Backend**: Flask, Flask-SQLAlchemy, Flask-JWT-Extended, Flask-CORS, PyMySQL
- **Frontend**: React (Vite), Tailwind CSS, React Router, Recharts, Axios
- **Database**: MySQL
- **ML**: scikit-learn (LogisticRegression with preprocessing pipeline)

## Project Structure

```
system/
├── asthma_prediction_model.pkl       # Pre-trained model
├── preprocessing_pipeline.pkl        # Preprocessing pipeline
├── backend/
│   ├── app.py                        # Flask app + entry point
│   ├── config.py                     # DB URI, JWT secret
│   ├── requirements.txt
│   ├── models/
│   │   ├── user.py                   # User model
│   │   └── prediction.py            # Prediction model
│   ├── routes/
│   │   ├── auth.py                   # Register, login, me
│   │   ├── predict.py               # Predict, history
│   │   └── admin.py                 # Admin endpoints
│   ├── services/
│   │   ├── ml_service.py            # Load model, feature engineering, predict
│   │   └── recommendation_service.py # Generate recommendations
│   └── utils/
│       └── decorators.py            # admin_required decorator
├── frontend/
│   ├── package.json
│   ├── vite.config.js
│   ├── index.html
│   └── src/
│       ├── main.jsx
│       ├── App.jsx                   # Router
│       ├── index.css                 # Tailwind directives
│       ├── api/axios.js             # Axios with JWT interceptor
│       ├── context/AuthContext.jsx   # Auth state
│       ├── components/
│       │   ├── Navbar.jsx
│       │   ├── ProtectedRoute.jsx
│       │   ├── AdminRoute.jsx
│       │   ├── PredictionForm.jsx   # 20-field health form
│       │   ├── RiskGauge.jsx        # Visual risk display
│       │   ├── RecommendationCard.jsx
│       │   └── StatsCard.jsx
│       └── pages/
│           ├── LoginPage.jsx
│           ├── RegisterPage.jsx
│           ├── PredictPage.jsx      # Form + result
│           ├── DashboardPage.jsx    # Patient history + charts
│           ├── ResultPage.jsx       # Single prediction detail
│           └── AdminDashboardPage.jsx
```

## Setup & Running

### 1. Database

Start MySQL and create the database:

```sql
CREATE DATABASE asthma_prediction;
```

### 2. Backend

```bash
cd backend
source venv/bin/activate
pip install -r requirements.txt
python app.py
```

The backend runs on `http://localhost:5000`.

### 3. Seed Admin User

```bash
cd backend
source venv/bin/activate
flask create-admin
```

Default credentials: `admin@example.com` / `admin123`

You can customize with flags:

```bash
flask create-admin --username myadmin --email admin@mysite.com --password securepass
```

### 4. Frontend

```bash
cd frontend
npm install
npm run dev
```

The frontend runs on `http://localhost:5173` with API requests proxied to the backend.

## API Endpoints

### Auth (`/api/auth`)

| Method | Endpoint    | Description          | Auth     |
|--------|-------------|----------------------|----------|
| POST   | `/register` | Create new user      | No       |
| POST   | `/login`    | Login, get JWT token | No       |
| GET    | `/me`       | Get current user     | Required |

### Predictions (`/api/predict`)

| Method | Endpoint   | Description                  | Auth     |
|--------|------------|------------------------------|----------|
| POST   | `/`        | Submit prediction (20 fields)| Required |
| GET    | `/history` | Paginated prediction history | Required |
| GET    | `/<id>`    | Single prediction detail     | Required |

### Admin (`/api/admin`)

| Method | Endpoint            | Description              | Auth  |
|--------|---------------------|--------------------------|-------|
| GET    | `/stats`            | System-wide analytics    | Admin |
| GET    | `/users`            | Paginated user list      | Admin |
| PUT    | `/users/<id>/role`  | Update user role         | Admin |
| GET    | `/predictions`      | All predictions paginated| Admin |

## Feature Engineering

The ML service computes these engineered features to match the training pipeline:

- **BMI_Category**: Underweight (<18.5), Normal (<25), Overweight (<30), Obese (>=30)
- **Age_Group**: Child (<=12), Teenager (<=19), Young Adult (<=35), Adult (<=55), Senior (>55)
- **Symptom_Count**: Sum of 7 binary symptom fields (0-7)
- **Risk_Score**: Smoking (0-2) + Family History (0/2) + Allergies (0-2) + Air Pollution (0-2) = range 0-8

## Frontend Pages

- **Login/Register** — Authentication forms with validation
- **Predict** — 4-section health form (Personal, Lifestyle, Medical, Symptoms) with risk gauge and recommendations on submit
- **Dashboard** — Stats cards, risk-over-time line chart, prediction history table with pagination
- **Result** — Full detail view of a past prediction with health info, symptoms, and recommendations
- **Admin Dashboard** — System stats, risk distribution pie chart, predictions-per-day bar chart, user management with role dropdown, all predictions table

## Configuration

Environment variables (or edit `backend/config.py`):

| Variable         | Default                                          | Description       |
|------------------|--------------------------------------------------|-------------------|
| `DATABASE_URL`   | `mysql+pymysql://root:@localhost/asthma_prediction` | MySQL connection  |
| `SECRET_KEY`     | `dev-secret-key-change-in-production`            | Flask secret key  |
| `JWT_SECRET_KEY` | `jwt-secret-key-change-in-production`            | JWT signing key   |
