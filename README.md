# Freshcart_Pipelines1

**Author:** Raj Kumar (`26PCSO04`)  
**Repository:** [https://github.com/rajiitdmj/Freshcart_Pipelines1](https://github.com/rajiitdmj/Freshcart_Pipelines1)  
**Course:** Data Engineering — Assignment: *From Git to Gold*

---

## Overview

This repository implements an end-to-end modern Data Engineering platform combining four core modules plus the RabbitMQ-to-Snowflake Lakehouse streaming bridge:

1. **Part A — Git & CI/CD (`.github/workflows/ci.yml`, `tests/`)**: Version-controlled repository structure with automated `pytest` verification on every push to `main` (Python 3.12).
2. **Part B — FreshCart ELT on Snowflake (`elt/FreshCart_ELT_Snowflake.ipynb`)**: Medallion architecture (`BRONZE` → `SILVER` / `QUARANTINE` → `DIM` SCD2 → `GOLD`) with schema normalization, deduplication, currency validation (`INR`/`USD`), idempotent `SHA-256` `row_hash` `MERGE`, and 6 automated SQL data-quality checks.
3. **Part C — ML Pipeline with Mage (`iris_project/`, `iris_project/Mage_Iris_Pipeline.ipynb`)**: Orchestrated Iris classification DAG (`load_iris` → `validate_iris_schema` → `prepare_data` → `train_model` → parallel `save_results` & `evaluate_model`) with inline `@test` quality gates.
4. **Part D — Food-Order Stream with RabbitMQ (`stream/dispatcher.py`, `stream/driver.py`)**: AMQP topic exchange (`rapido-dispatch`) routing `city.zone.type` events to driver queues with wildcard patterns (`*` and `#`), manual `basic_ack`, and poison-pill rejection (`basic_nack(requeue=False)`).
5. **Bonus — Stream into the Lakehouse (`stream/lakehouse_consumer.py`)**: Micro-batch consumer collecting RabbitMQ order events over a 60-second window, writing landing files via `write_landing()`, and running idempotent Snowflake transformations.

---

## Repository Structure

```text
Freshcart_Pipelines1/
├── .github/
│   └── workflows/
│       └── ci.yml
├── elt/
│   └── FreshCart_ELT_Snowflake.ipynb
├── iris_project/
│   ├── __init__.py
│   ├── Mage_Iris_Pipeline.ipynb
│   ├── metadata.yaml
│   ├── pipelines/
│   │   └── iris_classifier/
│   │       └── metadata.yaml
│   ├── data_loaders/
│   │   ├── __init__.py
│   │   └── load_iris.py
│   ├── transformers/
│   │   ├── __init__.py
│   │   ├── validate_iris_schema.py
│   │   ├── prepare_data.py
│   │   ├── train_model.py
│   │   └── evaluate_model.py
│   └── data_exporters/
│       ├── __init__.py
│       └── save_results.py
├── stream/
│   ├── dispatcher.py
│   ├── driver.py
│   └── lakehouse_consumer.py
├── tests/
│   ├── __init__.py
│   └── test_prepare_data.py
├── .gitignore
├── pytest.ini
├── requirements.txt
└── README.md
```

---

## Quickstart

### 1. Run CI Unit Tests Locally
```bash
pip install -r requirements.txt
pytest -v
```

### 2. Run Mage Iris ML Pipeline
```bash
mage run iris_project iris_classifier --test --runtime-vars '{"max_depth": 3}'
```

### 3. Run RabbitMQ Dispatcher & Drivers
```bash
# Terminal 1: Start dispatcher
python stream/dispatcher.py

# Terminal 2-4: Start driver consumers
python stream/driver.py Ravi "bangalore.koramangala.*"
python stream/driver.py Meena "bangalore.*.food"
python stream/driver.py Arjun "bangalore.*.ride"
python stream/driver.py Control "bangalore.#"
```
