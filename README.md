# Stream PySpark Job Execution & Monitoring with Prometheus & Grafana

An operational monitoring and observability infrastructure designed to track Apache Spark job metrics in real time. This project exposes PySpark internal metrics through a Prometheus servlet sink, collects scrape targets continuously, and visualizes system performance and scheduler health using dynamic Grafana dashboards.

---

## Architecture & Observability Workflow

The monitoring stack tracks Spark executor telemetry and DAG scheduler performance across three core layers:

```text
+-----------------------+      +------------------------+      +-------------------------+
|    PySpark Application| ---> |   Prometheus Server    | ---> |    Grafana Dashboard    |
| (Prometheus Servlet)  |      |   (Scrape Target :9090)|      |   (PromQL Visuals :3000)|
+-----------------------+      +------------------------+      +-------------------------+

```

### Operational Steps

1. **Application Configuration (`spark_job.py`)**:
* Initialized Spark Session with Prometheus metrics enabled (`spark.ui.prometheus.enabled` set to `true`).
* Configured `PrometheusServlet` sink mapped to `/metrics/executors/prometheus`.
* Performed real-time dataset filtering, join operations on customer and order data, and country-level revenue aggregations.


2. **Prometheus Target Scraping**:
* Configured Prometheus endpoint targets to pull live executor metrics from `localhost:4040`.
* Verified active health status on Prometheus target monitoring UI.


3. **Grafana Data Source & Dashboard Construction**:
* Linked Prometheus as the primary data source within Grafana.
* Formulated custom PromQL queries to build operational panels monitoring active jobs, completed jobs, failed jobs, active executor count, and CPU utilization percentage.



---

## Dashboard Metrics & PromQL Queries

| Metric Panel | Description | PromQL Expression | Grafana Unit |
| --- | --- | --- | --- |
| **Running Jobs** | Active DAG scheduler jobs | `{_name_=~".*DAGScheduler_job_activeJobs_Value"}` | Short |
| **Completed Jobs** | Total successfully executed jobs | `{_name_=~".*DAGScheduler_job_allJobs_Value"}` | Short |
| **Failed Jobs** | Count of failed execution jobs | `{_name_=~".*DAGScheduler_job_failedJobs_Value"} or vector(0)` | Short |
| **CPU Usage** | Rate of executor CPU time consumption | `rate({_name_=~".*executor_cpuTime_Count"}[1m]) / 1e9` | Percent (0.0–1.0) |
| **Executors Count** | Active threadpool tasks across executors | `count({_name_=~".*executor_threadpool_activeTasks_Value"})` | Short |

---

## Repository Structure

```text
.
├── src/
│   └── spark_job.py          # PySpark job configured with Prometheus metrics sink
├── config/
│   └── prometheus.yml        # Prometheus target configuration file
├── docs/
│   └── execution_and_monitoring.pdf # Monitoring and execution guide
└── README.md

```

---

## Run & Replicate

### Prerequisites

* **Apache Spark / PySpark**
* **Prometheus Server** (v3.x or compatible)
* **Grafana** (v10.x or compatible)

### Execution Steps

1. **Start PySpark Application**:
Execute the PySpark script to start processing and expose metrics at port `4040`:
```bash
python src/spark_job.py

```


2. **Launch Prometheus**:
Start the Prometheus server binary configured to scrape `http://localhost:4040/metrics/executors/prometheus`:
```powershell
.\prometheus.exe --config.file=prometheus.yml

```


3. **Launch Grafana**:
Start Grafana service and connect to Prometheus at `http://localhost:9090` as a data source.
4. **Import Dashboard**:
Create a new dashboard and apply the provided PromQL queries to display real-time cluster metrics.
