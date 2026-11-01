# MEVGuard - Backend Architecture

## 1. Introduction

The MEVGuard backend is responsible for processing application requests and executing the simulations required by the platform.

It acts as the communication layer between the frontend interface and the backend business logic.

## 2. Technology Stack

- Python - Backend programming language
- FastAPI - Backend API framework
- PostgreSQL - Relational database
- SQLAlchemy - Database interaction
- Alembic - Database migration management

## 3. Backend Components

### 3.1 API Layer

The API layer receives requests from the frontend application.

Its responsibilities include:
- Accepting HTTP requests.
- Validating incoming data.
- Calling the relevant simulation functionality.
- Returning structured responses.

### 3.2 Trade Protection Module

This module demonstrates MEV-related trading risks and protection approaches.

Its functionality includes:
- Sandwich attack simulation.
- Slippage tolerance validation.
- Private mempool protection simulation.
- Commit-reveal simulation.
- Time-lock simulation.

### 3.3 Decentralization Analysis Module

This module analyzes builder and relay concentration using supplied observations.

Its functionality includes:
- Builder market-share analysis.
- Relay concentration analysis.
- Herfindahl-Hirschman Index (HHI) calculation.
- PBS auction simulation.
- Builder outage simulation.

### 3.4 Network Efficiency Module

This module demonstrates transaction processing and network efficiency scenarios.

Its functionality includes:
- Transaction queue simulation.
- Block capacity analysis.
- Gas cost estimation.
- Transaction submission strategy comparison.

### 3.5 Database Layer

The database layer supports application data storage and experiment persistence where configured.

PostgreSQL is used as the relational database, SQLAlchemy supports database interactions, and Alembic manages schema migrations.

## 4. API Documentation

FastAPI provides an interactive Swagger UI interface for exploring and testing available API endpoints.

Default URL:

http://127.0.0.1:8000/docs

## 5. Database Migrations

Alembic manages database schema changes.

Migration command:

```powershell
alembic upgrade head
```

This command should be executed from the backend directory.

## 6. Backend Execution

The backend can be started from the project root using:

```powershell
.\start-backend.ps1
```

Default backend address:

http://127.0.0.1:8000

## 7. Environment Configuration

Backend environment variables are maintained in the backend/.env file.

Sensitive information such as database credentials and secret keys should never be committed to GitHub.

## 8. Conclusion

The MEVGuard backend provides the API and simulation functionality required by the frontend.

Its architecture supports trade protection, decentralization analysis, and network efficiency simulations.