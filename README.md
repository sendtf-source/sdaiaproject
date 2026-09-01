# Fraud Service for Transaction Risk Scoring

A production-oriented fraud detection service built as part of the SDAIA Academy program on engineering practices for AI systems. The project transforms a notebook-based fraud model into a structured service architecture with validation, configuration management, API endpoints, and batch-scoring workflows.

This project was completed during the SDAIA Academy training program from 31 August to 1 September 2026, under the program: Engineering Practices for AI Systems.

SDAIA Academy GitHub: https://github.com/SDAIAAcademy

## Project overview

The system reads transaction data, generates a fraud risk score using a pre-trained machine learning model, and makes a decision of allow, review, or block based on a configured risk threshold.

The service is designed to be safe for deployment and easier to maintain than a notebook prototype. It separates concerns across domain, service, API, and adapter layers, and it validates inputs before scoring.

## Why this project matters

This work demonstrates practical engineering practices for AI systems, including:

- clean architecture boundaries
- typed configuration with environment-variable validation
- input validation and fail-fast behavior
- API contract design with readiness and health checks
- model loading and warm-up at startup
- trace IDs and latency instrumentation
- production-friendly repository hygiene and documentation

## Architecture overview

The project follows a layered design:

- Domain layer: transaction model, feature transformation, and fraud decision types
- Service layer: scoring logic and decision policy
- Adapter layer: model inference integration with the trained sklearn pipeline
- API layer: FastAPI endpoints for prediction, health, and readiness
- Batch entrypoint: offline scoring across a CSV dataset

For a more detailed breakdown, see [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Repository structure

```text
.
├── data/
│   ├── golden_scores_v3.csv
│   ├── transactions_sample.csv
│   └── ...
├── docs/
│   └── ARCHITECTURE.md
├── models/
│   └── fraud_xgb_v3.joblib
├── payloads/
│   ├── valid/
│   └── malformed/
├── src/
│   └── fraud_service/
│       ├── adapters/
│       ├── api/
│       ├── config.py
│       ├── domain/
│       ├── logging_setup.py
│       ├── service/
│       ├── __init__.py
│       └── batch.py
├── tests/
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml
├── README.md
├── scored.csv
└── notebook_v1.ipynb
```

## Prerequisites

Before running the project, make sure you have:

- Python 3.12+
- pip
- Git
- A terminal or PowerShell session

## Setup

1. Clone the repository:

```bash
git clone https://github.com/sendtf-source/sdaiaproject.git
cd sdaiaproject
```

2. Create and activate a virtual environment:

On Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

On macOS/Linux:

```bash
python -m venv .venv
source .venv/bin/activate
```

3. Install dependencies:

```bash
pip install --upgrade pip
pip install -e .[dev]
```

## Required configuration

The application uses environment variables with the `FRAUD_` prefix. You can set them in a `.env` file or export them in the shell.

Example:

```env
FRAUD_MODEL_PATH=models/fraud_xgb_v3.joblib
FRAUD_BLOCK_THRESHOLD=0.85
FRAUD_LOG_LEVEL=INFO
```

Optional:

```env
FRAUD_REGISTRY_TOKEN=your_token_here
```

The project is designed to fail fast if required configuration is missing or invalid.

## Running the project

### 1) Batch scoring workflow

This runs the fraud model across the sample dataset and writes the results to `scored.csv`.

```bash
python -m fraud_service.batch
```

Expected output:

- the model loads successfully
- transaction decisions are generated
- a file named `scored.csv` is created in the project root

### 2) API service

Start the FastAPI application:

```bash
uvicorn fraud_service.api.app:app --host 0.0.0.0 --port 8000 --reload
```

Check the service:

```bash
curl http://localhost:8000/v1/health
curl http://localhost:8000/v1/ready
```

Example prediction payload:

```bash
curl -X POST http://localhost:8000/v1/predict \
  -H "Content-Type: application/json" \
  -d '{
    "transaction_id": "txn-001",
    "amount_sar": 520.75,
    "channel": "pos",
    "merchant_category": "GROCERY",
    "customer_id": "cust-123",
    "timestamp": "2026-09-01T12:30:00Z"
  }'
```

Expected result:

- HTTP 200 response
- JSON body containing the transaction ID, fraud probability, decision, and model version

## Model and data notes

The project uses a pre-trained bundled model stored in `models/fraud_xgb_v3.joblib` and sample transaction data in `data/transactions_sample.csv`.

The training data includes a label column for offline evaluation and benchmarking, but the serving contract deliberately excludes it from the runtime API.

## Good Git and project practices

This repository follows modern engineering practices for version control and reproducibility:

- small, incremental project changes rather than a single bulk upload
- clear project structure with source code separated from data and generated artifacts
- a `.gitignore` file to protect local environment settings and generated files
- explicit dependency management through `pyproject.toml`
- clean commit discipline with meaningful changes and traceable progression

The project also avoids committing secrets, local environment files, caches, coverage output, and generated scoring artifacts.

## Security and environment hygiene

Sensitive or generated files are excluded by the project configuration, including:

- `.env` files
- Python cache folders
- local virtual environments
- test coverage output
- generated score artifacts
- editor and OS metadata

## Testing and validation

The repository is set up for pytest-based validation and coverage checks:

```bash
pytest
```

The project also includes code quality tooling and architecture checks configured in `pyproject.toml`.

## Credits and attribution

This project was developed under the SDAIA Academy program titled:

Engineering Practices for AI Systems

Dates: 31 August 2026 to 1 September 2026

Institutional GitHub reference: https://github.com/SDAIAAcademy

## License

This project is intended for educational and training purposes within the SDAIA Academy program.

## Contact and usage notes

This repository is designed to demonstrate how a fraud model can move from an exploratory notebook into a structured, testable, and production-minded AI service.

The implementation is intentionally modular so it can be extended with:

- additional validation rules
- model registry integration
- monitoring and observability
- asynchronous job workers
- deployment automation and container orchestration
