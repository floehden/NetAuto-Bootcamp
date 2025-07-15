# DevOps Observability & Monitoring Workshop

A practical, project-driven workshop covering observability basics, Prometheus metrics collection, Node Exporter for host monitoring, and Grafana for dashboards.  
By the end, you’ll be able to instrument, collect, and visualize metrics from Linux systems.

---

## Course Overview

This fast-paced course introduces the essentials of modern monitoring.  
You’ll learn why observability matters, set up a metrics collection pipeline using Prometheus and Node Exporter, and build live dashboards with Grafana.

---

## Prerequisites

- Basic Linux command‑line & shell skills

---

## Schedule

| Day | Topic                    | Hands-On Labs & Demos                                    |
|-----|--------------------------|----------------------------------------------------------|
| 1   | Observability Fundamentals | The Three Pillars; Value for business and engineering    |
| 2   | Prometheus Metrics & Setup | Install and configure Prometheus; Scrape demo app metrics|
| 3   | Node Exporter             | Install and configure Node Exporter; Collect host metrics|
| 4   | Grafana Dashboards        | Install Grafana; Visualize metrics; Build custom dashboards|

---

### Day 1 - Observability Fundamentals

- **Concepts:**  
  - What is observability?  
  - The Three Pillars: Metrics, Logs, Traces  
  - Business and engineering value of observability  
  - Observability vs. classic monitoring

- **Activities:**  
  - Discuss real-world observability scenarios  
  - Identify key metrics for a sample system

---

### Day 2 - Prometheus Metrics & Setup

- **Concepts:**  
  - Prometheus architecture and data model  
  - Metric types: counter, gauge, histogram, summary  
  - Pull model and scraping targets  
  - Basics of PromQL (Prometheus Query Language)

- **Hands-On:**  
  - Install Prometheus server  
  - Edit `prometheus.yml` to add a sample job  
  - Scrape demo application metrics  
  - Explore metrics in Prometheus UI

---

### Day 3 - Node Exporter

- **Concepts:**  
  - What is Node Exporter?  
  - Common Linux system metrics: CPU, memory, disk, network  
  - Adding exporters as scrape targets in Prometheus

- **Hands-On:**  
  - Install Node Exporter  
  - Start the Node Exporter service  
  - Update Prometheus config to scrape Node Exporter  
  - Explore system metrics in Prometheus UI

---

### Day 4 - Grafana Dashboards

- **Concepts:**  
  - Why Grafana for visualization?  
  - Connecting Prometheus as a data source  
  - Panels, dashboards, variables, and alerting basics

- **Hands-On:**  
  - Install Grafana  
  - Connect Grafana to Prometheus  
  - Build dashboards for CPU, memory, and system health  
  - Share or export dashboard JSON

---

## References

- [Prometheus Documentation](https://prometheus.io/docs)
- [Node Exporter Guide](https://prometheus.io/docs/guides/node-exporter/)
- [Grafana Documentation](https://grafana.com/docs/)

