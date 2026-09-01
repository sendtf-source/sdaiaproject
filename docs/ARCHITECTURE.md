# Architecture Overview

## Purpose

This project implements a fraud-scoring service that takes transaction data, applies a pre-trained model, and returns a risk decision. The design follows clean software engineering principles and separates model logic from API concerns.

## High-level flow

1. A request enters the FastAPI application.
2. The request is validated using Pydantic models.
3. The domain entity transforms raw input into model features.
4. The service layer calls the model adapter.
5. The model adapter runs inference using the sklearn pipeline.
6. The result is converted into a decision: allow, review, or block.
7. The HTTP response is returned with a trace ID and latency metadata.

## Layer responsibilities

### Domain layer

The domain layer defines the vocabulary of the application:

- Transaction data model
- Channel enum
- FeatureVector representation
- Fraud decision enum
- FraudScore response object

This layer contains the business concepts and feature engineering logic, without depending on pandas or sklearn.

### Service layer

The service layer owns the scoring workflow:

- reads model output
- applies the threshold policy
- returns decision objects

This is where the fraud decision policy is implemented.

### Adapter layer

The adapter layer isolates the external ML dependency:

- loads the trained model bundle from disk
- prepares a DataFrame for sklearn
- runs prediction probability inference

This keeps the rest of the application independent from the model implementation details.

### API layer

The API layer exposes the service through HTTP:

- `POST /v1/predict` for scoring
- `GET /v1/health` for liveness
- `GET /v1/ready` for readiness

The API uses FastAPI and structured logging for observability.

### Batch entrypoint

The batch script provides an offline scoring workflow:

- loads the sample transaction dataset
- converts each record into a domain entity
- scores the dataset with the model
- writes the output to `scored.csv`

## Design principles

- Separation of concerns
- Clear dependency direction
- Fail-fast validation
- Single source of configuration
- Model loading only at startup
- No business logic in the API layer
- No raw environment reads spread throughout the app

## Configuration model

The project centralizes configuration in `src/fraud_service/config.py` using Pydantic settings. Environment variables are validated at startup to prevent silent configuration drift.

## Observability

Structured logs are emitted for:

- model loading
- request processing
- prediction serving
- latency tracking
- startup and shutdown events

Each request is assigned a trace ID so it can be tracked through the service life cycle.

## Project goals

The system demonstrates how a notebook-based ML workflow can be refined into an engineering-ready service with:

- maintainable architecture
- clean API contracts
- safer model serving patterns
- documented deployment assumptions
- reproducible project structure
