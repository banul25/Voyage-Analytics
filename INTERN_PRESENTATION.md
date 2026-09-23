# Voyage Analytics - Project Explanation for Instructor

## 1. Project Overview

**Voyage Analytics** is an MLOps capstone project that demonstrates how to productionize machine learning models for a travel platform. It's a complete end-to-end implementation that takes raw travel data, trains machine learning models, exposes them through APIs, and deploys them with Docker and Kubernetes.

**Core Idea**: Instead of just building ML models in notebooks, this project shows how to build **production-ready** ML services that can be scaled, monitored, and maintained.

---

## 2. Three Main ML Services

The project implements **3 different ML models** that work together:

### a) **Flight Price Prediction**
- **Model Type**: XGBoost Regression
- **Purpose**: Predicts flight prices based on route, class, agency, and date
- **Input Data**: `flights.csv` dataset
- **Example Inputs**: 
  - From/To locations
  - Flight class (economy, business, first)
  - Travel date and day of week
  - Distance and time
- **Output**: Predicted price

### b) **Gender Classification**
- **Model Type**: RandomForest Classifier
- **Purpose**: Classifies user gender based on user profile data
- **Input Data**: `users.csv` dataset
- **Output**: Gender prediction with confidence

### c) **Hotel Recommendation Engine**
- **Type**: Content-Based Recommendation System
- **Purpose**: Recommends hotels based on user preferences
- **Input Data**: `hotels.csv` dataset + user preferences
- **Bonus Feature**: Smart location search with fuzzy matching (handles typos!)

---

## 3. Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Training Phase                            │
│              (Google Colab / Jupyter Notebook)               │
│         • Load flights.csv, hotels.csv, users.csv            │
│         • Train XGBoost, RandomForest models                 │
│         • Save as .joblib artifacts                          │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
         ┌─────────────────────────────┐
         │   MLflow Registry           │
         │  (Track metrics & params)   │
         └──────────────┬──────────────┘
                        │
                        ▼
         ┌─────────────────────────────┐
         │   Model Artifacts (joblib)  │
         │  Stored in artifacts/       │
         └──────────────┬──────────────┘
                        │
         ┌──────────────┴──────────────┐
         │                             │
         ▼                             ▼
    Flask REST API          Streamlit Dashboard
    (http://localhost:5000)  (Interactive UI)
         │                             │
         └──────────────┬──────────────┘
                        │
         ┌──────────────┴──────────────┐
         │                             │
         ▼                             ▼
      Docker Container         Kubernetes Cluster
    (Portable)              (Production Deployment)
```

---

## 4. Project Structure & Components

### **api/** - REST API Layer
- **Framework**: Flask
- **Purpose**: Exposes ML models as HTTP endpoints
- **Key Files**:
  - `routes.py` - Health check, model info, flight predictions
  - `gender_routes.py` - Gender classification endpoint
  - `recommend_routes.py` - Hotel recommendations & location search
  - `app.py` - Flask app factory (configures all blueprints)

### **src/** - Core Business Logic
- **features/** - Feature engineering shared across models
- **model/** - Model loading logic (loads .joblib files)
- **schemas/** - Pydantic data validation (ensures API inputs are correct)
- **services/** - Business logic
  - `prediction_service.py` - Flight price prediction
  - `gender_service.py` - Gender classification
  - `recommendation_service.py` - Hotel recommendations
  - `location_search_service.py` - Fuzzy matching for location autocomplete

### **artifacts/** - Trained Models & Data
- `flight_price_pipeline.joblib` - Trained XGBoost model
- `gender_model.joblib` - Trained RandomForest model
- `hotel_catalog.json` - Hotel database
- `data/` - CSV files (flights, hotels, users)

### **dashboard/** - Streamlit Web UI
- Interactive interface to test all 3 models
- Visualize predictions without needing to call API
- User-friendly way to explore the models

### **mlflow_tracking/** - MLOps Tracking
- Logs model metrics (accuracy, precision, recall, etc.)
- Tracks hyperparameters used
- Manages model versioning
- Central registry of all trained models

### **docker/** - Containerization
- `Dockerfile` - Containerizes the Flask API
- `docker-compose.mlflow.yml` - Runs MLflow server + database
- Makes deployment portable across machines

### **kubernetes/** - Orchestration
- `deployment.yaml` - How to deploy the app
- `service.yaml` - Exposes the app to the network
- `configmap.yaml` - Configuration for the cluster
- Enables scaling and high availability

### **tests/** - Quality Assurance
- 30+ automated tests
- ~85% code coverage
- Tests for all 3 models + API endpoints

---

## 5. Key Technologies

| Technology | Purpose | Why Used |
|---|---|---|
| **Python** | Core language | ML/Data science standard |
| **Flask** | REST API framework | Lightweight, production-ready |
| **XGBoost** | Flight price model | Excellent for regression tasks |
| **RandomForest** | Gender classifier | Robust classification |
| **Pydantic** | Data validation | Type safety + documentation |
| **Scikit-learn** | ML utilities | Standard ML library |
| **MLflow** | Model tracking | Industry-standard MLOps tool |
| **Streamlit** | Dashboard | Easy interactive UI |
| **Docker** | Containerization | Environment consistency |
| **Kubernetes** | Orchestration | Production deployment |
| **Pytest** | Testing | Automated quality assurance |

---

## 5.5. Key Talking Points - Brief Summary

### **1. The Three Services**
- **Flight Price Prediction (XGBoost)**: Predicts ticket prices from flight details (route, class, date). Enables dynamic pricing for travel platforms.
- **Gender Classification (RandomForest)**: Classifies user gender from profile data. Used for personalization and demographic analysis.
- **Hotel Recommendation (Content-Based)**: Recommends hotels matching user preferences. Works for new users by matching hotel features to preferences.

### **2. Architecture Layers**
- **Training**: Load CSV data → Engineer features → Train models → Save as .joblib → Log to MLflow for tracking
- **API**: Flask REST endpoints load trained models → Validate input with Pydantic → Return predictions as JSON
- **Dashboard**: Streamlit UI calls API endpoints → Users test models without code → Beautiful visualizations
- **Deployment**: Docker containerizes app for portability → Kubernetes orchestrates scaling, load balancing, auto-restart

### **3. Professional Practices**
- **MLflow Tracking**: Version control for ML models—tracks metrics, parameters, and artifacts like Git does for code
- **Automated Testing**: 30+ tests with 85% coverage catch bugs early and prevent regressions in production
- **Clean Code**: Organized modular structure (src/, api/, tests/) makes code maintainable and easy to understand
- **Containerization**: Docker + Kubernetes ensures app runs identically everywhere and scales automatically with traffic

### **4. How to Demonstrate**
```bash
# Start the API
python -m api.app

# Start the dashboard (in another terminal)
streamlit run dashboard/app.py

# Run tests with coverage (in another terminal)
pytest tests/ -v --cov=src
```

---

### **Step 1: Training** (Offline)
```
User trains models in Jupyter/Colab
    ↓
Models saved as .joblib files (binary format)
    ↓
Metrics logged to MLflow (accuracy, loss, etc.)
```

### **Step 2: API Service Loads Models**
```
Flask app starts
    ↓
Loads .joblib models into memory
    ↓
Ready to accept prediction requests
```

### **Step 3: User Makes Prediction Request**
```
POST /api/predict with flight details
    ↓
API validates input (Pydantic)
    ↓
Feature engineering applied
    ↓
XGBoost model predicts price
    ↓
Returns JSON response
```

### **Step 4: Dashboard Shows Results**
```
User inputs data in Streamlit UI
    ↓
Calls Flask API endpoints
    ↓
Displays predictions beautifully
```

### **Step 5: Production Deployment**
```
Docker image created
    ↓
Pushed to Kubernetes cluster
    ↓
Multiple instances running (load balancing)
    ↓
Scales automatically with traffic
```

---

## 7. Example API Workflow

### Flight Price Prediction Request
```json
POST /api/predict
{
  "from": "Recife (PE)",
  "to": "Florianopolis (SC)",
  "flightType": "firstClass",
  "agency": "FlyingDrops",
  "distance": 676.53,
  "flight_year": 2019,
  "flight_month": 9,
  "flight_day": 26,
  "flight_dayofweek": 3
}

Response:
{
  "predicted_price": 1250.75,
  "model_version": "1.0",
  "model_name": "flight_price_regression"
}
```

---

## 8. Quality & Testing

The project includes **automated tests**:
- ✅ Model loading tests
- ✅ API endpoint tests
- ✅ Prediction accuracy tests
- ✅ Gender classification tests
- ✅ Recommendation system tests
- ✅ Health check tests

**85% code coverage** ensures reliability and makes maintenance easier.

---

## 9. MLOps Best Practices Demonstrated

1. **Model Versioning** - MLflow tracks all model versions
2. **Artifact Management** - Central storage of models and data
3. **API Contract** - Pydantic schemas ensure consistency
4. **Testing** - Automated tests catch issues early
5. **Containerization** - Docker ensures consistency across environments
6. **Orchestration** - Kubernetes enables production scaling
7. **Monitoring** - MLflow provides centralized tracking

---

## 10. Quick Start Commands

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Run the API
python -m api.app

# 3. Run the dashboard
streamlit run dashboard/app.py

# 4. Run tests
pytest tests/ -v --cov=src

# 5. Start MLflow server
mlflow ui

# 6. Deploy with Docker
docker-compose -f docker/docker-compose.mlflow.yml up
```

---

## 11. Key Learnings (What Makes This "Production-Ready")

❌ **What Beginners Do**:
- Train model, save as pickle, done
- No testing
- No versioning
- Hard to reproduce

✅ **What This Project Does (Professional)**:
- Clean separation of concerns (src/, api/, tests/)
- Comprehensive testing + coverage reporting
- Model versioning with MLflow
- Reproducible training pipeline
- API contract validation (Pydantic)
- Multiple deployment options (Docker, K8s)
- Monitoring and logging
- Documentation for each module

---

## 12. Real-World Applications

This architecture is used in production by companies like:
- Uber, Airbnb (recommendations)
- Airlines, travel booking sites (price prediction)
- E-commerce platforms (personalization)
- Banks, fintech (fraud detection, risk modeling)

---

## Summary

**Voyage Analytics** is a complete demonstration of turning ML models into **production systems**. It shows:
1. How to organize ML code professionally
2. How to expose models via APIs
3. How to test and validate predictions
4. How to version and track models
5. How to containerize and deploy at scale

This is the **difference between ML research and ML engineering**.

---

**Want to explore more?** Check:
- `Update.md` for known issues and improvements
- Each module's README.md for detailed documentation
- Source code comments for implementation details
