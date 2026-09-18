# vllm-inference-platform-ansible

vLLM Inference Platform — Ansible Playbooks.
One playbook turns a bare AlmaLinux‑RHEL machine into a ready‑to‑serve vLLM inference node with full monitoring.
Pre‑flight checks → NVIDIA driver → vLLM service → DCGM‑Exporter → Prometheus → Grafana.

Every node gets identical configuration.
Grafana exposes TTFT, token throughput, KV‑cache utilization, per‑GPU usage & memory.
Data‑driven observability for inference capacity and latency.

Tested on AlmaLinux/RHEL.9, NVIDIA A10 
## Architecture
                    ┌─────────┐
                    │ Grafana │ :3000
                    └────┬────┘
                         │
                    ┌────┴─────────┐
                    │  Prometheus  │ :9090
                    └─┬─────────┬──┘
                      │         │       scrape /metrics
          ┌───────────┴──┐  ┌───┴────────────┐
          │ DCGM-Exporter│  │      vLLM      │ :8000
          │    :9400     │  │  /v1  /metrics │
          └───────┬──────┘  └───────┬────────┘
                  │                 │
              ┌───┴─────────────────┴───┐
              │       GPU node(s)       │
              │ AlmaLinux 9 · A10 │
              └───────────────────────────┘

## Components
| Component        | What it does |
|------------------|--------------|
| vLLM             | OpenAI‑compatible inference server, systemd‑managed service, native Prometheus metrics enabled |
| DCGM‑Exporter    | GPU hardware metrics: utilization, memory, temperature, power |
| Prometheus       | Scrapes metrics from vLLM and DCGM‑Exporter endpoints |
| Grafana          | Datasource and dashboards provisioned at deploy time; no manual UI setup |
| Benchmark Script | Local load tester for vLLM; generates concurrent requests to measure TTFT, token throughput under real load |
| Ansible Roles    | Idempotent automation for full‑node bootstrapping; pre‑flight validation, dependency setup & service lifecycle management |

## Environment
- OS: AlmaLinux 9
- GPU: NVIDIA A10 
