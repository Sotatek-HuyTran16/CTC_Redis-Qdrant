# CTC-Redis-Qdrant

## Overview

**CTC-Redis-Qdrant** là project triển khai nền tảng ứng dụng AI trên **Canonical Kubernetes**, kết hợp các **database phân tán** và **external services** nhằm đáp ứng yêu cầu **High Availability, Security và Observability**.

Project bao phủ toàn bộ vòng đời:
- Tạo và maintain Kubernetes cluster
- Triển khai dịch vụ nền tảng (Harbor registry, Bind DNS, object storage MinIO, PyPi Server)
- Deploy ứng dụng **Dify** và các database (Redis, Qdrant, PostgreSQL) lên Kubernetes
- Deploy monitoring stack (prometheus, exporter) và logging stack (loki, promtail)

---

## Main Components

### Kubernetes Platform
- Canonical Kubernetes
- Calico CNI
- UFW firewall hardened
- RBAC & Pod Security Standards

### Databases
- Redis HA (Sentinel)
- Qdrant
- PostgreSQL

### Application
- Dify (API, Worker, Web)

### External Services
- Harbor Registry
- Bind DNS
- MinIO Object Storage
- PyPi Server

## Monitor and Log
- Prometheus / Grafana / Exporter
- Loki / Promtail