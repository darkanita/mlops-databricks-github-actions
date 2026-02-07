# MLOps Databricks Starter

![Python](https://img.shields.io/badge/Python-3.9+-blue)
![MLflow](https://img.shields.io/badge/MLflow-2.0+-orange)
![Databricks](https://img.shields.io/badge/Databricks-Community-red)
![License](https://img.shields.io/badge/License-MIT-green)

🚀 **End-to-end MLOps pipeline using GitHub Actions, Databricks Community Edition, and MLflow**

This educational project demonstrates how to build a production-ready MLOps pipeline using entirely free tools. It implements automated training, experiment tracking, model registry, and CI/CD workflows for machine learning models.

## 📋 Table of Contents

- [Features](#-features)
- [Architecture](#-architecture)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Project Structure](#-project-structure)
- [Pipeline Workflow](#-pipeline-workflow)
- [Configuration](#-configuration)
- [Usage](#-usage)
- [MLflow Tracking](#-mlflow-tracking)
- [Extending the Project](#-extending-the-project)
- [Troubleshooting](#-troubleshooting)
- [Contributing](#-contributing)
- [License](#-license)

## ✨ Features

- **Automated CI/CD Pipeline**: GitHub Actions orchestrates the entire ML workflow
- **Databricks Integration**: Leverages Databricks Free Edition for scalable compute
- **MLflow Tracking**: Complete experiment tracking and model versioning
- **Modular Design**: Separate notebooks for data preparation, training, and evaluation
- **Production Ready**: Follows MLOps best practices for model lifecycle management
- **100% Free**: Uses only free-tier services (GitHub, Databricks Community Edition)

## 🏗️ Architecture
```
┌─────────────────┐
│  GitHub Repo    │
│  (Code & CI/CD) │
└────────┬────────┘
         │
         │ Push/PR triggers
         ▼
┌─────────────────────────────────────┐
│     GitHub Actions Workflow         │
│  ┌───────────────────────────────┐  │
│  │  1. Data Preparation Job      │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │  2. Model Training Job        │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │  3. Model Evaluation Job      │  │
│  └───────────────────────────────┘  │
└────────┬────────────────────────────┘
         │
         │ Execute notebooks via API
         ▼
┌─────────────────────────────────────┐
│        Databricks Free Edition      │
│  ┌────────────┐  ┌────────────────┐ │
│  │ Notebooks  │  │ Spark Cluster  │ │
│  └────────────┘  └────────────────┘ │
│  ┌────────────┐  ┌────────────────┐ │
│  │  MLflow    │  │  DBFS Storage  │ │
│  └────────────┘  └────────────────┘ │
└─────────────────────────────────────┘
```

## 📦 Prerequisites

### Accounts & Access
- GitHub account (free)
- Databricks Community Free account ([Sign up here](https://community.cloud.databricks.com/))

### Local Development (Optional)
- Python 3.9+
- Git

## 🚀 Quick Start

### 1. Fork/Clone This Repository
```bash
git clone https://github.com/darkanita/mlops-databricks-github-actions.git
cd mlops-databricks-github-actions
```

### 2. Set Up Databricks

1. **Create a Databricks Account**
   - Go to [Databricks Community Edition](https://community.cloud.databricks.com/)
   - Sign up for free

2. **Create a Cluster**
   - Navigate to Compute → Create Cluster
   - Cluster name: `mlops-cluster`
   - Databricks Runtime: `13.3 LTS ML` or later
   - Node type: Smallest available (default)
   - Click "Create Cluster"
   - **Copy the Cluster ID** from the URL (e.g., `1234-567890-abc123`)

3. **Generate Access Token**
   - Click on your user icon (top right) → Settings
   - Go to Developer → Access Tokens
   - Click "Generate New Token"
   - Comment: `GitHub Actions`
   - Lifetime: 90 days (or as needed)
   - **Save this token securely** - you won't see it again

4. **Note Your Workspace URL**
   - Usually: `https://community.cloud.databricks.com`

### 3. Configure GitHub Secrets

1. Go to your GitHub repository
2. Navigate to **Settings → Secrets and variables → Actions**
3. Click **"New repository secret"** and add:

| Secret Name | Value | Example |
|------------|-------|---------|
| `DATABRICKS_HOST` | Your Databricks workspace URL | `https://community.cloud.databricks.com` |
| `DATABRICKS_TOKEN` | Your personal access token | `dapi1234567890abcdef...` |
| `CLUSTER_ID` | Your cluster ID | `1234-567890-abc123` |

### 4. Trigger the Pipeline

**Option A: Push to Main Branch**
```bash
git add .
git commit -m "Initial setup"
git push origin main
```

**Option B: Manual Trigger**
1. Go to **Actions** tab in GitHub
2. Select **"MLOps Pipeline"** workflow
3. Click **"Run workflow"**

### 5. Monitor Execution

1. **GitHub Actions**
   - Go to the **Actions** tab
   - Watch the pipeline progress through each job
   
2. **Databricks**
   - Log into Databricks
   - Check **Workflows → Jobs** for running jobs
   - View **Experiments** for MLflow tracking
   - Check **Models** for registered models

## 📁 Project Structure
```
mlops-databricks-starter/
├── .github/
│   └── workflows/
│       └── mlops_pipeline.yml          # GitHub Actions workflow definition
├── notebooks/
│   ├── 01_data_preparation.py          # Data ingestion and preprocessing
│   ├── 02_model_training.py            # Model training with MLflow tracking
│   └── 03_model_evaluation.py          # Model evaluation and promotion
├── src/                                 # Python modules (optional)
│   ├── data_processing.py
│   ├── model.py
│   └── utils.py
├── tests/                               # Unit tests (optional)
│   └── test_model.py
├── requirements.txt                     # Python dependencies
├── .gitignore
├── LICENSE
└── README.md
```

## 🔄 Pipeline Workflow

### Job 1: Data Preparation
**What it does:**
- Creates sample customer churn dataset
- Performs basic data validation
- Saves processed data to DBFS
- **Output:** `/FileStore/mlops/data/customer_data.parquet`

**Triggers:** Push to `main` or `develop` branch

### Job 2: Model Training
**What it does:**
- Loads prepared data from DBFS
- Trains Random Forest classifier
- Logs parameters, metrics, and model to MLflow
- Registers model in MLflow Model Registry
- **Output:** Trained model + MLflow run

**Depends on:** Data Preparation job completion

**MLflow Tracking:**
- Experiment name: `/MLOps/experiments/model_training`
- Registered model: `customer_churn_model`

### Job 3: Model Evaluation
**What it does:**
- Loads latest registered model
- Evaluates on test data
- Generates classification report and confusion matrix
- Promotes model to "Staging" if F1 score > 0.7
- **Output:** Evaluation metrics + model promotion

**Depends on:** Model Training job completion

## ⚙️ Configuration

### Customizing the Pipeline

**Edit Model Parameters** (`notebooks/02_model_training.py`):
```python
params = {
    'n_estimators': 100,        # Number of trees
    'max_depth': 10,            # Max tree depth
    'min_samples_split': 5,     # Min samples to split
    'random_state': 42
}
```

**Change Data Source** (`notebooks/01_data_preparation.py`):
```python
# Replace sample data generation with your data source
# Example: Read from CSV
df = spark.read.csv("/path/to/your/data.csv", header=True, inferSchema=True)
```

**Adjust Evaluation Threshold** (`notebooks/03_model_evaluation.py`):
```python
if f1 > 0.7:  # Change this threshold
    # Promote to staging
```

### Environment Variables

You can add additional parameters via GitHub Actions workflow:
```yaml
- name: Run model training with MLflow
  env:
    CLUSTER_ID: ${{ secrets.CLUSTER_ID }}
    MODEL_VERSION: v1.0
    EXPERIMENT_NAME: /MLOps/experiments/custom_experiment
```

## 🎯 Usage

### Running Locally (Optional)
```bash
# Install dependencies
pip install -r requirements.txt

# Configure Databricks CLI
databricks configure --token

# Upload notebooks manually
databricks workspace import_dir notebooks /MLOps/notebooks --overwrite
```

### Viewing MLflow Experiments

1. Log into Databricks
2. Navigate to **Machine Learning → Experiments**
3. Find experiment: `/MLOps/experiments/model_training`
4. Compare runs, view metrics, and download artifacts

### Accessing Registered Models

1. Navigate to **Machine Learning → Models**
2. Find model: `customer_churn_model`
3. View versions, stages, and transitions

### Manual Model Promotion
```python
import mlflow

client = mlflow.tracking.MlflowClient()

# Promote to Production
client.transition_model_version_stage(
    name="customer_churn_model",
    version=1,
    stage="Production"
)
```

## 📊 MLflow Tracking

### Logged Parameters
- `n_estimators`: Number of trees in Random Forest
- `max_depth`: Maximum depth of trees
- `min_samples_split`: Minimum samples required to split
- `random_state`: Random seed for reproducibility

### Logged Metrics
- `accuracy`: Overall prediction accuracy
- `precision`: Positive predictive value
- `recall`: Sensitivity / true positive rate
- `f1_score`: Harmonic mean of precision and recall

### Logged Artifacts
- `model/`: Serialized scikit-learn model
- `feature_importance.txt`: Feature importance rankings

### Model Registry Stages
- **None**: Initial registration
- **Staging**: Passed evaluation threshold
- **Production**: Manually promoted for deployment
- **Archived**: Deprecated models

## 🔧 Extending the Project

### Add Data Validation
```python
# notebooks/01_data_preparation.py
from great_expectations.dataset import SparkDFDataset

ge_df = SparkDFDataset(spark_df)
assert ge_df.expect_column_values_to_not_be_null('customer_id').success
assert ge_df.expect_column_values_to_be_between('age', 18, 100).success
```

### Implement Hyperparameter Tuning
```python
# notebooks/02_model_training.py
from sklearn.model_selection import GridSearchCV

param_grid = {
    'n_estimators': [50, 100, 200],
    'max_depth': [5, 10, 15],
    'min_samples_split': [2, 5, 10]
}

grid_search = GridSearchCV(
    RandomForestClassifier(),
    param_grid,
    cv=5,
    scoring='f1'
)

with mlflow.start_run():
    grid_search.fit(X_train, y_train)
    mlflow.sklearn.log_model(grid_search.best_estimator_, "model")
```

### Add Model Drift Monitoring
```python
# Create new notebook: 04_model_monitoring.py
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset

report = Report(metrics=[DataDriftPreset()])
report.run(reference_data=reference_df, current_data=current_df)
```

### Deploy Model as API
```python
# Use MLflow serving
mlflow models serve -m "models:/customer_churn_model/Production" -p 5000

# Or deploy to Databricks Model Serving
client.create_model_serving_endpoint(
    name="churn-prediction-api",
    config={
        "served_models": [{
            "model_name": "customer_churn_model",
            "model_version": "1",
            "workload_size": "Small",
            "scale_to_zero_enabled": True
        }]
    }
)
```

## 🐛 Troubleshooting

### Common Issues

**Issue: GitHub Action fails with "Databricks authentication failed"**
```
Solution: 
1. Verify DATABRICKS_HOST includes https://
2. Regenerate DATABRICKS_TOKEN if expired
3. Check token permissions in Databricks
```

**Issue: Cluster not starting in Databricks**
```
Solution:
1. Community Edition has usage limits - wait if exceeded
2. Try deleting and recreating the cluster
3. Use a smaller cluster configuration
```

**Issue: Notebook execution times out**
```
Solution:
1. Increase timeout in GitHub Actions (default: 30s)
2. Use a larger cluster (if available)
3. Optimize notebook code for performance
```

**Issue: MLflow experiment not found**
```
Solution:
1. Manually create experiment in Databricks UI
2. Verify experiment path matches exactly
3. Check permissions for experiment access
```

**Issue: Job runs but no MLflow data appears**
```
Solution:
1. Check notebook output for errors
2. Verify mlflow.set_experiment() is called
3. Ensure cluster has MLflow installed
```

### Debug Mode

Enable verbose logging in GitHub Actions:
```yaml
- name: Run model training with MLflow
  env:
    DATABRICKS_DEBUG: true
  run: |
    databricks jobs run-now --job-id $JOB_ID --debug
```

### Checking Logs

**GitHub Actions Logs:**
- Actions tab → Select workflow run → Click on failed job

**Databricks Logs:**
- Workflows → Jobs → Select job → View run output
- Clusters → Select cluster → Event Log / Driver Logs

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

1. **Fork the repository**
2. **Create a feature branch** (`git checkout -b feature/amazing-feature`)
3. **Commit your changes** (`git commit -m 'Add amazing feature'`)
4. **Push to the branch** (`git push origin feature/amazing-feature`)
5. **Open a Pull Request**

### Contribution Ideas
- Add support for other ML frameworks (XGBoost, LightGBM, PyTorch)
- Implement A/B testing framework
- Add data versioning with DVC
- Create Docker deployment option
- Add integration tests
- Improve documentation with more examples

## 📚 Resources

### Documentation
- [Databricks Community Edition](https://docs.databricks.com/en/getting-started/community-edition.html)
- [MLflow Documentation](https://mlflow.org/docs/latest/index.html)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Databricks CLI](https://docs.databricks.com/en/dev-tools/cli/index.html)

### Tutorials
- [MLflow Tracking Tutorial](https://mlflow.org/docs/latest/tracking.html)
- [GitHub Actions for ML](https://github.com/features/actions)
- [Databricks ML Runtime](https://docs.databricks.com/en/machine-learning/index.html)

### Related Projects
- [MLflow Examples](https://github.com/mlflow/mlflow/tree/master/examples)
- [Awesome MLOps](https://github.com/visenger/awesome-mlops)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Built for educational purposes
- Inspired by real-world MLOps practices
- Community contributions welcome

## 📬 Contact

Have questions or suggestions? 

- Open an [Issue](https://github.com/yourusername/mlops-databricks-starter/issues)
- Start a [Discussion](https://github.com/yourusername/mlops-databricks-starter/discussions)
- Submit a [Pull Request](https://github.com/yourusername/mlops-databricks-starter/pulls)

---

**⭐ If this project helped you, please give it a star!**

**Made with ❤️ for the ML community**
