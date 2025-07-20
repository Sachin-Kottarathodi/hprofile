---
title: "System Design - Distributed Log Ingestion"
date: 2025-07-20
tags: ["system-design", "hld"]
Categories: ["tech"]
series: ["System Design"]
draft: false
---


# 🧠 System Design Interview Summary: Log Ingestion & Query System
 
**Interviewer:** Vega  
**Topic:** Design a system for log ingestion, storage, and querying across multi-tenant agents

---

## ✅ High-Level Architecture

1. **Agents** send logs (JSON) via HTTP to a **rate-limited ingress service**
2. Ingress service writes logs to **Kafka** (HA cluster)
3. Two main Kafka consumers:
   - **Object Store Consumer**: Stores raw logs in GCS/S3
   - **Indexing Consumer**: Pushes structured logs to **Elasticsearch**
4. **Elasticsearch Cluster** (with snapshots) holds searchable logs
5. **Query Layer** exposes APIs (or Kibana) to end users
6. **Metadata DB** stores user info, tenant configs, RBAC rules
7. **Telemetry pipeline** for usage and system health insights

---

## 💡 Key Design Decisions

### 🔹 Data Format
- **JSON** for ingestion (readable, schema-tolerant)
- **Protobuf** or **compressed archives** for long-term storage

### 🔹 Schema Evolution
- Agent schemas versioned per tenant
- Schema registry to ensure backward compatibility
- Only expected fields are accepted/processed

### 🔹 Indexing & Querying
- Indexed fields include timestamp, log level, service name, etc.
- Optional full-text search on `message` field
- Search APIs backed by RBAC and tenant-scoped filters

### 🔹 ElasticSearch Resilience
- High availability setup with shard replication
- Snapshots taken periodically for disaster recovery
- Index Lifecycle Management (ILM) policies to manage TTL

---

## 🏢 Multi-Tenancy Strategy

| Type | Description |
|------|-------------|
| **Pseudo Multi-Tenancy** | Shared Kafka/Elastic infra, tenant isolation via schema/index prefixes |
| **True Multi-Tenancy** | Dedicated infra (Kafka, GCS, Elastic) per tenant — promoted based on volume or SLA |
| **RBAC** | Controlled via metadata DB and AD group mapping |

---

## 🛡️ Security, Rate Limiting & Abuse Protection

- **Ingress Layer** (HTTP proxy) applies:
  - Request validation
  - API token auth
  - Rate limiting per tenant
- Circuit breaker patterns to prevent cascading failures
- DDOS prevention via quotas & burst control

---

## 📊 Telemetry & Monitoring

- Logs from agents and query APIs sent to a telemetry Kafka topic
- Prometheus/Grafana stack monitors system health and usage patterns
- Audit logs retained for admin usage

---

## 🔁 Resilience & Failure Handling

- Kafka consumers checkpointed to restart safely
- Elasticsearch snapshot restore plans
- Retry queues for failed ingestion
- Dead Letter Queue (DLQ) for malformed logs

---

## 📦 Object Storage Strategy

- Long-term log archive in GCS/S3 with partitioning by tenant/date
- Lifecycle rules to auto-transition to cold storage (e.g., after 30 days)
- Optional encryption per tenant (KMS)

---

# 📌 PIA Notes — Follow-Ups & Deep Dive Areas

### 🔸 Retention & Lifecycle
- Per-tenant TTL?
- ILM policy configurations in Elasticsearch?
- GCS lifecycle rules for archival?

### 🔸 Schema Strategy
- Schema registry integration
- How to version agent schemas cleanly?
- What to do if agent sends unexpected fields?

### 🔸 Query Layer UX
- Native UI (like Kibana) or custom frontend?
- Role-based query controls
- Audit logs of queries

### 🔸 Telemetry & Alerts
- Can users set their own alert rules?
- Do we support lag/delay/error rate alerting?
- Anomaly detection roadmap?

### 🔸 Multi-Tenancy Expansion
- What triggers upgrade to true multi-tenancy?
- Cost model for per-tenant provisioning
- How to support hybrid mode?

### 🔸 Backpressure & Rate Limiting
- Propagation of backpressure to agents?
- Real-time quota metrics exposed to tenants?

---

# ✅ Next Steps

- [ ] Deep dive into 1–2 areas (Telemetry, RBAC, or ILM?)
- [ ] Add rough architecture diagram (ASCII or PlantUML?)

---