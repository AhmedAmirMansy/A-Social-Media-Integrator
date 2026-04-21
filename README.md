<p align="center">
  <img src="https://img.shields.io/badge/Python-3.9%2B-3776AB?style=for-the-badge&logo=python&logoColor=white" />
  <img src="https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/Apache%20Airflow-2.10.3-017CEE?style=for-the-badge&logo=apacheairflow&logoColor=white" />
  <img src="https://img.shields.io/badge/Apache%20Spark-PySpark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white" />
  <img src="https://img.shields.io/badge/Apache%20Kafka-Streaming-231F20?style=for-the-badge&logo=apachekafka&logoColor=white" />
  <img src="https://img.shields.io/badge/PostgreSQL-13-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" />
</p>

# 🦎 Salamanders — Stock Portfolio Analytics Pipeline

> **End-to-end data engineering pipeline** for stock portfolio analytics — from raw CSV ingestion through real-time streaming, distributed analytics, interactive dashboards, and an AI-powered natural language agent.

**Team Salamanders** · Group ID: 55

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Milestones](#-milestones)
  - [M1 — Data Cleaning & Integration](#milestone-1--data-cleaning--integration)
  - [M2 — Encoding, Streaming & Spark](#milestone-2--encoding-streaming--spark-analytics)
  - [M3 — Orchestration, Dashboards & AI Agent](#milestone-3--orchestrated-pipeline-dashboards--ai-agent)
- [Quick Start](#-quick-start)
- [Service URLs](#-service-urls)
- [Dashboard & Visualizations](#-dashboard--visualizations)
- [AI Agent](#-ai-agent)
- [Project Structure](#-project-structure)
- [Troubleshooting](#-troubleshooting)

---

## 🔭 Overview

This project builds a **complete data engineering pipeline** for a stock portfolio trading platform. It progresses across three milestones—each building on the previous—covering the full lifecycle of data engineering:

1. **Data Cleaning & Integration** — Ingest, clean, and merge multi-source trading data into a unified dataset.
2. **Feature Encoding & Real-time Streaming** — Encode categorical features, split data for streaming simulation, produce/consume via Kafka, and run Spark analytics.
3. **Orchestration & Intelligence** — Orchestrate the full pipeline with Apache Airflow, visualize insights on an interactive Streamlit dashboard, and query data conversationally through an AI agent.

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        APACHE AIRFLOW ORCHESTRATION                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐   ┌─────────────┐   ┌──────────────────────┐              │
│  │  Stage 1    │   │  Stage 2    │   │     Stage 3          │              │
│  │  Data       │──▶│  Encoding   │──▶│  Kafka Streaming     │              │
│  │  Cleaning & │   │  & Stream   │   │  Producer → Consumer │              │
│  │  Integration│   │  Prep       │   │                      │              │
│  └─────────────┘   └─────────────┘   └──────────┬───────────┘              │
│                                                  │                          │
│                                                  ▼                          │
│                                      ┌───────────────────┐                  │
│                                      │    Stage 4        │                  │
│                                      │    Spark          │                  │
│                                      │    Analytics      │                  │
│                                      └─────────┬─────────┘                  │
│                                           ┌────┴────┐                       │
│                                           ▼         ▼                       │
│                                  ┌────────────┐ ┌────────────┐              │
│                                  │  Stage 5   │ │  Stage 6   │              │
│                                  │  Streamlit │ │  AI Agent  │              │
│                                  │  Dashboard │ │  (NL→SQL)  │              │
│                                  └────────────┘ └────────────┘              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                       │
                              ┌────────▼────────┐
                              │   PostgreSQL    │
                              │  Data Warehouse │
                              └─────────────────┘
```

---

## 🧰 Tech Stack

| Layer | Technology |
|-------|------------|
| **Orchestration** | Apache Airflow 2.10.3 |
| **Streaming** | Apache Kafka (Confluent 7.4) + Zookeeper |
| **Analytics** | Apache Spark (PySpark) |
| **Data Warehouse** | PostgreSQL 13 |
| **Visualization** | Streamlit + Plotly |
| **AI / LLM** | Groq API (LLaMA 3.3 70B) |
| **Data Processing** | Pandas, NumPy, scikit-learn |
| **Containerization** | Docker & Docker Compose |
| **Language** | Python 3.9+ |

---

## 🚩 Milestones

### Milestone 1 — Data Cleaning & Integration

> **Directory:** [`M1/`](./M1)

The foundation of the pipeline: raw data ingestion, profiling, cleaning, and integration.

**What it does:**
- Loads 5 source datasets — `trades`, `daily_trade_prices`, `dim_customer`, `dim_stock`, `dim_date`
- Handles missing values (median imputation for prices)
- Detects and treats outliers (IQR-based with percentile clipping)
- Standardizes text fields (case normalization, whitespace trimming)
- Reshapes wide-format price data into long format
- Merges all datasets into a single **16-column** unified DataFrame
- Computes derived features: `average_trade_size` (rolling window), `total_trade_amount`
- Persists the cleaned dataset to **PostgreSQL**

**Key files:**
| File | Purpose |
|------|---------|
| `app/src/salamanders.py` | Main ETL script |
| `app/src/db_utils.py` | PostgreSQL connection utilities |
| `salamanders.ipynb` | Exploratory notebook version |
| `docker-compose.yaml` | PostgreSQL + pgAdmin + ETL container |

**Run M1:**
```bash
cd M1
docker-compose up -d --build
```

---

### Milestone 2 — Encoding, Streaming & Spark Analytics

> **Directory:** [`M2/`](./M2)

Extends M1 with feature encoding, real-time Kafka streaming, and distributed Spark analytics.

**What it does:**
- **Label Encoding** of 8 categorical columns (`stock_ticker`, `transaction_type`, `day_name`, etc.)
- Generates a **label encoding lookup table** for decoding
- Splits data into **95% offline** and **5% streaming** subsets
- Computes helper attributes (`avg_price_monthly`) for stream enrichment
- **Kafka Producer** sends stream records to a dedicated topic
- **Kafka Consumer** receives, cleans, encodes, and appends streaming records in real-time
- **Spark Master/Worker** cluster for distributed analytics
- Outputs the final merged dataset as `FINAL_STOCKS.csv`

**Key files:**
| File | Purpose |
|------|---------|
| `app/src/salamanders.py` | Full pipeline: clean → encode → stream → merge |
| `app/src/producer.py` | Kafka producer script |
| `app/src/spark_analysis.py` | PySpark analytics operations |
| `app/src/db_utils.py` | Database utilities |
| `docker-compose.yaml` | Full stack: Postgres, Kafka, Zookeeper, Spark, pgAdmin |

**Run M2:**
```bash
cd M2/app
docker-compose up -d --build
```

---

### Milestone 3 — Orchestrated Pipeline, Dashboards & AI Agent

> **Directory:** [`M3/`](./M3)

The final milestone: full pipeline orchestration with Airflow, interactive visualizations, and an AI-powered data assistant.

**What it does:**
- **Apache Airflow DAG** with 6 TaskGroups orchestrating the entire pipeline
- **Data Cleaning Group** — Missing values, outliers, integration, load to Postgres
- **Encoding Group** — Stream preparation, categorical encoding
- **Streaming Group** — Kafka producer/consumer with PostgreSQL persistence
- **Analytics Group** — PySpark session initialization + 10 analytical operations
- **Visualization Group** — Streamlit dashboard with 4 interactive tabs and 15+ charts
- **Agent Group** — AI agent that converts natural language queries to SQL

**Key files:**
| File | Purpose |
|------|---------|
| `dags/stock_portfolio_pipeline_salamanders.py` | Airflow DAG definition |
| `dags/modules/cleaning.py` | Data cleaning tasks |
| `dags/modules/encoding.py` | Feature encoding tasks |
| `dags/modules/streaming.py` | Kafka streaming tasks |
| `dags/modules/analytics.py` | Spark analytics tasks |
| `dags/modules/agent.py` | AI agent logic (NL → SQL) |
| `visualization/dashboard.py` | Streamlit analytics dashboard |
| `visualization/agent_ui.py` | AI agent chat interface |
| `docker-compose.yaml` | 10-service deployment |

**Run M3:**
```bash
cd M3
docker-compose up -d --build
```
Wait ~2 minutes for all services to initialize.

---

## 🚀 Quick Start

### Prerequisites

- [Docker](https://docs.docker.com/get-docker/) & [Docker Compose](https://docs.docker.com/compose/install/)
- At least **8 GB RAM** allocated to Docker (recommended)

### Run the Full Pipeline (M3)

```bash
# Clone the repository
git clone https://github.com/<your-username>/Salamanders.git
cd Salamanders/M3

# Start all services
docker-compose up -d --build

# Monitor logs
docker-compose logs -f
```

Once services are ready, open the Airflow UI and trigger the DAG:

1. Navigate to http://localhost:8080
2. Log in with **admin / admin**
3. Enable and trigger `stock_portfolio_pipeline_salamanders`

---

## 🌐 Service URLs

| Service | URL | Credentials |
|---------|-----|-------------|
| **Airflow Web UI** | http://localhost:8080 | `admin` / `admin` |
| **Analytics Dashboard** | http://localhost:8501 | — |
| **AI Agent Chat** | http://localhost:8502 | — |
| **Spark Master UI** | http://localhost:8081 | — |
| **PostgreSQL** | `localhost:5432` | `root` / `root` |

---

## 📊 Dashboard & Visualizations

The Streamlit dashboard provides **4 interactive tabs** with real-time filtering by sector:

### Tab 1 — Market Overview
- Trading volume by ticker (Top 10)
- Buy vs Sell distribution (pie chart)
- Trading activity by day of week
- Customer activity distribution

### Tab 2 — Deep Dive
- 📈 Price trend over time (line chart)
- Average price by sector
- Top 10 customers by trade amount (treemap)
- Industry breakdown

### Tab 3 — Advanced Analytics
- Sector comparison (volume vs transaction count)
- Liquidity tier analysis (stacked area)
- Account type breakdown (pie chart)
- Weekend vs weekday trading

### Tab 4 — Real-time Stream
- Live transaction feed (latest 50 records)
- Auto-refresh toggle (5–60 second intervals)

### KPI Metrics
| Metric | Description |
|--------|-------------|
| Total Trading Volume | Sum of all quantities |
| Avg Market Price | Mean stock price |
| Active Stocks | Unique tickers |
| Total Trade Value | Sum of trade amounts |
| Active Customers | Unique customer IDs |

---

## 🤖 AI Agent

An AI-powered data assistant accessible at http://localhost:8502.

**How it works:**
1. You type a natural language question about the stock data
2. The agent uses **Groq (LLaMA 3.3 70B)** to convert your question into SQL
3. The query runs against the PostgreSQL data warehouse
4. Results are displayed in a table with an AI-generated analysis

**Example queries:**
```
Show me the top 10 customers by total trade amount
What is the average stock price per sector?
Compare buy vs sell transactions on weekends
Which day of the week has the highest trading volume?
```

**Features:**
- Natural language → SQL conversion
- Tabular results display
- AI-powered analysis of query results
- CSV export
- Query history

---

## 📁 Project Structure

```
Salamanders/
│
├── M1/                                 # Milestone 1: Data Cleaning & Integration
│   ├── app/
│   │   ├── src/
│   │   │   ├── salamanders.py          # Main ETL pipeline
│   │   │   └── db_utils.py            # PostgreSQL utilities
│   │   ├── data/                      # Source CSV datasets
│   │   └── volumes/                   # PostgreSQL persistent data
│   ├── salamanders.ipynb              # Jupyter notebook version
│   ├── Dockerfile
│   ├── docker-compose.yaml
│   └── requirements.txt
│
├── M2/                                 # Milestone 2: Encoding & Streaming
│   └── app/
│       ├── src/
│       │   ├── salamanders.py          # Full pipeline (clean → encode → stream)
│       │   ├── producer.py            # Kafka producer
│       │   ├── spark_analysis.py      # PySpark analytics
│       │   └── db_utils.py
│       ├── data/                      # Source + generated datasets
│       ├── volumes/
│       ├── Dockerfile
│       ├── docker-compose.yaml
│       └── requirements.txt
│
├── M3/                                 # Milestone 3: Full Orchestrated Pipeline
│   ├── dags/
│   │   ├── stock_portfolio_pipeline_salamanders.py   # Airflow DAG
│   │   └── modules/
│   │       ├── cleaning.py            # Stage 1: Data cleaning
│   │       ├── encoding.py            # Stage 2: Feature encoding
│   │       ├── streaming.py           # Stage 3: Kafka streaming
│   │       ├── analytics.py           # Stage 4: Spark analytics
│   │       └── agent.py              # Stage 6: AI agent logic
│   ├── visualization/
│   │   ├── dashboard.py               # Streamlit analytics dashboard
│   │   └── agent_ui.py               # AI agent chat interface
│   ├── agent/                         # Agent I/O files & logs
│   ├── data/                          # All data files & encoders
│   ├── Dockerfile
│   ├── docker-compose.yaml
│   ├── requirements.txt
│   ├── README.md
│   └── VISUALIZATION.md
│
├── descriptions/                       # Project milestone PDFs
│   ├── milestone2.pdf
│   └── milestone3.pdf
│
└── README.md                          # ← You are here
```

---

## 🛠 Troubleshooting

### Services not starting?
```bash
# Check service status
docker-compose ps

# View logs for a specific service
docker-compose logs -f <service-name>

# Full restart
docker-compose down && docker-compose up -d --build
```

### Airflow DAG not visible?
```bash
# Restart the scheduler
docker-compose restart airflow-scheduler
```

### Dashboard showing no data?
Make sure the Airflow DAG has completed successfully first. The dashboard reads from the PostgreSQL data warehouse which gets populated during pipeline execution.
```bash
# Verify data exists in Postgres
docker exec m3-postgres-dw-1 psql -U root -d lab4db -c "SELECT COUNT(*) FROM final_stocks;"
```

### Kafka connection issues?
```bash
# Restart Kafka + Zookeeper
docker-compose restart zookeeper kafka
```

---

<p align="center">
  <b>Team Salamanders</b> 🦎 · Built with ❤️ for Data Engineering
</p>
