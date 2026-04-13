## 1. Objective

Build a **Reward Orchestrator service** that acts as a centralized gateway between **reward intent** and **reward execution**.

## 2. Background & Problem Statement

Currently, reward execution is distributed across multiple services such as:

- Free Spin
- Fire Bonus
- Lucky Wheel

This leads to:

- Multiple integrations per client
- Duplication of logic (validation, retry, deduplication)
- Increased complexity and poor maintainability

## 3. Responsibilities

### 3.1 Receiving & Validating Reward Requests

- Accept reward requests from multiple upstream systems
- Validate request payload and required fields
### 3.2 Standardization

- Normalize reward processing across all reward types
- Ensure consistent processing flow
### 3.3 Reward Service Routing

- Route rewards to appropriate domain services
- Support multiple reward types
### 3.4 Reward Lifecycle Management

- Track reward status transitions:
	- `PENDING → PROCESSING → SUCCESS / FAILED`
### 3.5 Reliability

- Ensure idempotency
- Handle deduplication
- Manage failures within synchronous execution
### 3.6 Audit & Observability

- Maintain logs and execution history
- Enable traceability for debugging

## 4. High-Level Architecture

### 4.1 System Flow (Synchronous)

%% Retry / Backoff
    R{Retry Allowed?}
    RB[Retry Policy<br/>(Exponential Backoff)]
    F[Mark FAILED]

    %% Domain Services
    FS[FreeSpin Service]
    LS[Loyalty Service]
    PS[Promotion Service]

    %% Response
    K[Response<br/>SUCCESS / FAILED]

    %% Flow
    A --> B
    B --> C
    C -->|Persist PENDING| E
    C --> D

    %% Routing
    D -->|FREESPIN| FS
    D -->|LOYALTY| LS
    D -->|PROMOTION| PS

    %% Success Path
    FS -->|Success| D
    LS -->|Success| D
    PS -->|Success| D

    %% Failure Path
    FS -->|Failure| R
    LS -->|Failure| R
    PS -->|Failure| R

    %% Retry Flow
    R -->|Yes| RB --> D
    R -->|No| F

    %% Final Updates
    D -->|Update SUCCESS| E
    F -->|Update FAILED| E

    %% Response
    D --> K
    F --> K


