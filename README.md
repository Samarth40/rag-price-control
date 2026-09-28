# RAG Price Control Layer

[![GitHub Repository](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)](https://github.com/Samarth40/rag-price-control)
[![Diagram](https://img.shields.io/badge/gitdiagram-view%20architecture-blue)](https://gitdiagram.com/Samarth40/rag-price-control)

*Built by Samarth Shinde*

## Overview

The RAG Price Control Layer is a robust, production-ready blueprint designed to optimize Retrieval-Augmented Generation (RAG) workflows. By sitting in front of your core RAG backend, this application intelligently manages query routing and caching to drastically reduce language model costs and improve response latency.

Key features include:
- **Semantic Caching:** Instantly serve answers for near-duplicate queries at zero cost.
- **Model Cascading and Routing:** Dynamically route queries to cost-effective models for simple tasks, only escalating to more expensive, powerful models when confidence is low.
- **Tiered Retrieval:** Implement a two-stage retrieval process utilizing BM25 pre-filtering alongside vector search.
- **Comprehensive Observability:** Track costs, latencies, cache hit rates, and escalation events on a per-tenant basis.

## System Architecture

The workflow of a query through the price control layer:

1. A query is received and checked against the semantic cache.
2. If a cache hit occurs, the cached answer is returned immediately ($0 cost, minimal latency).
3. On a cache miss, the complexity router analyzes the request.
4. The query is routed to a lightweight, economical model.
5. If the economical model yields low confidence, the query is escalated to a high-capacity model.
6. Execution metrics (cost, latency, route taken) are logged in the observability database.
7. The new answer is stored in the semantic cache for future similar requests.

## Getting Started

### Prerequisites

Ensure you have Docker and Python installed on your system.

### Installation

1. Copy the environment template and configure your API keys:
   ```bash
   cp .env.example .env
   # Edit .env and supply your GROK_API_KEY
   ```

2. Start the Redis instance for semantic caching:
   ```bash
   docker compose up -d redis
   ```

3. Install the required Python dependencies:
   ```bash
   pip install -r requirements.txt
   ```

4. Launch the FastAPI backend:
   ```bash
   uvicorn app.main:app --port 8001 --reload
   ```

5. In a separate terminal session, start the Streamlit dashboard:
   ```bash
   export COST_CONTROL_API_URL=http://localhost:8001
   streamlit run streamlit_app.py
   ```

## API Reference

The backend provides the following endpoints:

- `POST /smart-query`
  Accepts a JSON payload containing `namespace`, `query`, and `context`. Returns either a cached response or a newly generated answer along with cost, latency, and routing metadata.

- `GET /stats/{namespace}?window_seconds=86400`
  Retrieves aggregate performance metrics (cache hits, costs, latencies, escalations) for a specific namespace over a defined time window.

## Integration Guidelines

To wire this layer with an existing RAG backend (such as a multi-tenant RAG-as-a-Service):

1. **Backend Adjustment:** Ensure your core RAG application provides a retrieval-only endpoint (e.g., `/query/retrieve`) that returns document chunks without invoking an LLM.
2. **Orchestrator Setup:** In `rag-price-control/app/services/orchestrator.py`, the `smart_query()` function accepts an optional `api_key`. If no context is provided directly, the application will automatically fetch context chunks from your core backend's retrieval endpoint.

By passing a tenant ID as the `namespace` parameter, all cache entries, costs, and statistics are scoped to individual tenants, allowing for precise tracking of savings and performance improvements.