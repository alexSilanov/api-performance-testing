# API Performance Testing — Restful Booker

> Performance testing portfolio project using **Apache JMeter, InfluxDB, Grafana and Windows Performance Monitor**.

---

## 🎯 Objectives

- Evaluate API stability under sustained concurrent load
- Measure response time and throughput
- Analyze **P90 / P95 / P99** latency
- Monitor error rate
- Validate API behavior under increasing concurrency
- Identify concurrency and data-consistency issues
- Correlate API performance with system resource utilization

---

## 🛠 Tech Stack

| Tool                            | Purpose                                |
| ------------------------------- | -------------------------------------- |
| **Apache JMeter 5.6.3**         | Load and stress testing                |
| **Restful Booker REST API**     | System under test                      |
| **Groovy / JSR223**             | Dynamic correlation and scripting      |
| **JSON Extractor**              | Extract booking IDs from API responses |
| **Throughput Controller**       | Model business traffic distribution    |
| **InfluxDB**                    | Time-series performance metrics        |
| **Grafana**                     | Real-time performance dashboards       |
| **Windows Performance Monitor** | CPU and memory monitoring              |
| **Docker**                      | Reproducible environment               |

---

## 🐳 Quick Start with Docker

```bash
# Build JMeter image with required plugins
docker compose build jmeter

# Start all services
docker compose up -d
```

Services after start:

| Service         | URL                      | Credentials     |
|-----------------|--------------------------|-----------------|
| Restful Booker  | http://localhost:3001    | —               |
| Grafana         | http://localhost:3000    | admin / admin   |
| InfluxDB        | http://localhost:8086    | jmeter / jmeter |

### Run tests via Docker

```bash
# Load Test
docker compose exec jmeter jmeter -n -t /tests/api_load_test.jmx \
  -l /results/load-test/results.jtl \
  -e -o /results/load-test/jmeter-report

# Stress Test
docker compose exec jmeter jmeter -n -t /tests/api_load_test.jmx \
  -l /results/stress-test/results.jtl \
  -e -o /results/stress-test/jmeter-report
```

---

## 🧪 Test Scenario

The test plan models two main user journeys: **Review** and **Manage Booking**.

```text
Thread Group
│
├── Authentication
│
├── Review — 60%
│   ├── GET /booking
│   │   └── JSON Extractor → bookingId
│   └── GET /booking/{bookingId}
│
└── Manage Booking — 40%
    ├── POST /booking
    │   └── JSON Extractor → createdBookingId
    │
    └── Booking operation
        ├── PUT /booking/{createdBookingId} — 75%
        └── DELETE /booking/{createdBookingId} — 25%
```

### Approximate operation distribution

| Operation            | Share   |
| -------------------- | ------- |
| GET /booking         | **60%** |
| POST /booking        | **40%** |
| PUT /booking/{id}    | **30%** |
| DELETE /booking/{id} | **10%** |

Dynamic booking IDs are correlated using **JSON Extractor** and **JSR223/Groovy**.

**Test Plan:** [`jmeter/api_load_test.jmx`](jmeter/api_load_test.jmx)

---

## 📊 Load Test

**Profile:** 50 concurrent users · 10 minutes

| Metric                  | Result         |
| ----------------------- | -------------- |
| Concurrent users        | **50**         |
| Duration                | **10 min**     |
| Total requests          | **9,262**      |
| Throughput              | **~15.4 req/s**|
| Average response time   | **3 ms**       |
| Maximum response time   | **37 ms**      |
| Error rate              | **0.00%**      |
| P95 (worst)             | **≤ 7 ms**     |
| P99 (worst)             | **21.41 ms**   |

**Detailed summary:** [results/load-test/summary_load_test.md](results/load-test/summary_load_test.md)

### Key Graphs

![Concurrent Users](screenshots/01-concurrent-users.png)
![Throughput](screenshots/02-throughput.png)
![Throughput by Endpoint](screenshots/03-throughput-by-endpoint.png)
![P90](screenshots/04-response-time-p90.png)
![P95](screenshots/05-response-time-p95.png)

### Conclusion
The API remained stable at 50 concurrent users with **0.00% errors**. Response times stayed in the low-millisecond range.

---

## 🔥 Stress Test

**Profile:** Step load 100 → 150 → 200 concurrent users

| Metric                  | Result             |
| ----------------------- | ------------------ |
| Maximum virtual users   | **200**            |
| Duration                | **~11 min 25 sec** |
| Total requests          | **23,756**         |
| Average throughput      | **36.0 req/s**     |
| Peak throughput         | **52.9 req/s**     |
| Average response time   | **~6.14 ms**       |
| Maximum response time   | **96 ms**          |
| Errors                  | **4**              |
| Error rate              | **~0.017%**        |

**Detailed summary:** [results/stress-test/summary_stress_test.md](results/stress-test/summary_stress_test.md)

### Key Graphs

![Concurrent Users](screenshots/10-stress-users.png)
![Throughput](screenshots/11-stress-throughput.png)
![P90](screenshots/13-stress-p90.png)
![P95](screenshots/14-stress-p95.png)

### Main Observations
- Throughput scaled with concurrency up to **~53 req/s**
- `GET /booking review` became the main latency outlier under peak load
- A small number of 404 errors occurred due to race conditions on dynamic booking data

---

## 🖥️ System Resource Monitoring

Monitored with **Windows Performance Monitor** (5-second sampling).

| Resource | Average   | Maximum    |
| -------- | --------- | ---------- |
| CPU      | **6.7%**  | **16.5%**  |
| Memory   | **83.6%** | **84.2%**  |

CPU remained low even at 200 concurrent users — the test machine was **not CPU-bound**.

---

## ⚠️ Data Consistency / Race Condition

Under concurrent load the following situation is possible:

```text
User A: GET /booking → receives ID 123
User B: DELETE /booking/123
User A: GET /booking/123 → 404 Not Found
```

This is an expected **concurrency / data-consistency** behavior caused by multiple virtual users operating on the same dynamic dataset.  
During the Stress Test only **4 HTTP 404 responses** were recorded (~0.017%).

---

## 📁 Repository Structure

```text
api-performance-testing/
├── docker/
│   └── jmeter/
│       ├── Dockerfile
│       └── plugins.txt
├── docker-compose.yml
├── jmeter/
│   └── api_load_test.jmx
├── results/
│   ├── load-test/
│   │   └── summary_load_test.md
│   └── stress-test/
│       └── summary_stress_test.md
├── screenshots/
└── README.md
```

---

## 🏆 Key Results

| Test        | Users | Requests | Throughput     | Avg RT   | Errors   |
|-------------|-------|----------|----------------|----------|----------|
| **Load**    | 50    | 9,262    | ~15.4 req/s    | 3 ms     | 0.00%    |
| **Stress**  | 200   | 23,756   | 36.0 req/s (peak 52.9) | ~6.14 ms | ~0.017% |

**Main finding:** The API successfully handled the increase from 50 to 200 concurrent users without CPU saturation. The main observed issue was a small number of race-condition 404 responses on dynamic booking data.

---

## 👨‍💻 Skills Demonstrated

Performance Test Planning · Load Testing · Stress Testing · Apache JMeter · REST API Testing · Non-GUI Execution · Dynamic Correlation · JSON Extractor · JSR223/Groovy · Throughput Controller · InfluxDB · Grafana · Windows Performance Monitor · P90/P95/P99 Analysis · Docker · Reproducible Test Environment · Race-Condition Analysis · Performance Reporting
