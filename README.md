API Performance Testing — Restful Booker
Performance testing portfolio project using Apache JMeter, InfluxDB and Grafana.

🎯 Objectives
Evaluate API stability under sustained concurrent load
Measure response time and throughput
Analyze P90 / P95 / P99 latency
Monitor error rate
Validate behavior under increasing concurrency
Identify concurrency and data-consistency issues
🛠 Tech Stack
Apache JMeter 5.6.3 · InfluxDB · Grafana · REST API · Groovy/JSR223 · GitHub

🧪 Test Scenario
Thread Group
│
├── Authentication
│
├── Review — 60%
│   ├── GET /booking
│   └── GET /booking/{id}
│
└── Manage Booking — 40%
    ├── POST /booking
    │   └── Extract createdBookingId
    └── Booking operation
        ├── PUT /booking/{createdBookingId} — 75%
        └── DELETE /booking/{createdBookingId} — 25%

Approximate operation distribution:

Operation	Share
GET /booking	60%
POST /booking	40%
PUT /booking/{id}	30%
DELETE /booking/{id}	10%
📊 Load Test
Profile: 50 concurrent users, 10 minutes, Non-GUI execution.

Metric	Result
Users	50
Duration	10 min
Total requests	9,262
Throughput	~15.4 req/s
Average response time	3 ms
Min	0 ms
Max	37 ms
Error rate	0.00%
Concurrent Users
Concurrent Users

Throughput
Throughput

Steady-state throughput in the Grafana view is approximately 12.5–13.3 req/s.

Throughput by Endpoint
Throughput by Endpoint

⏱ Response Time
P90
P90

Endpoint	P90
GET /booking review	6 ms
GET /booking	6 ms
DELETE /booking/bookingId	6 ms
PUT /booking/createdBookingId	5 ms
GET /booking/createdBookingId	5 ms
POST /booking	5 ms
POST /auth	3 ms
P95
P95

Endpoint	P95
GET /booking review	7 ms
GET /booking	7 ms
DELETE /booking/bookingId	6 ms
PUT /booking/createdBookingId	6 ms
GET /booking/createdBookingId	6 ms
POST /booking	5 ms
POST /auth	4 ms
P99
Metrics Overview

Endpoint	P99
GET /booking review	21.41 ms
GET /booking	9.00 ms
DELETE /booking/bookingId	16.03 ms
PUT /booking/createdBookingId	8.15 ms
GET /booking/createdBookingId	7.00 ms
POST /booking	7.00 ms
POST /auth	4.00 ms
🔥 Stress Test
A separate step-load test was executed:

100 → 150 → 200 concurrent users → graceful shutdown

Initial JMeter results:

26,023 requests
5 ms average response time
38 ms maximum
11 errors / 0.04%
Throughput increased approximately from 29 req/s at 100 users to 43 req/s at 150 users and 57 req/s at 200 users.

Stress-test Grafana screenshots and detailed endpoint error analysis can be added as additional evidence if required.

⚠️ Data Consistency / Race Condition
The booking dataset is dynamic. During concurrent execution, this can happen:

User A: GET /booking → receives ID 123
User B: DELETE /booking/123
User A: GET /booking/123 → 404

This is a concurrency/data-consistency issue and should be distinguished from technical performance failures. The test does not simply hide all 404 responses by marking them successful.

🖥️ System Resource Monitoring
A clean system-resource measurement was collected during the Stress Test using the built-in Windows Performance Monitor with a 5-second sampling interval.

Resource	Min	Avg	Max
CPU	0.95%	4.33%	9.16%
Memory (Committed)	76.79%	77.37%	77.80%
The JMeter host remained far from CPU saturation: average CPU utilization was 4.33% and the maximum observed value was 9.16%. Memory utilization was stable at approximately 77% throughout the monitored period.

Memory metric note: the collected counter is % Committed Bytes In Use / % использования выделенной памяти. It represents committed memory utilization and should not be interpreted as a direct percentage of physical RAM in use.

Memory utilization
Memory utilization

CPU utilization
CPU utilization

Min / Avg / Max resource statistics
CPU and memory statistics

📈 Monitoring
Apache JMeter
      │
      ▼
   InfluxDB
      │
      ▼
    Grafana

Monitored metrics:

Virtual Users
Throughput / RPS
Endpoint throughput
Average response time
P90 / P95 / P99
Error rate
Endpoint-level performance
▶️ Run
cd C:\Projects\apache-jmeter-5.6.3\bin

jmeter.bat -n -t "C:\Projects\apache-jmeter-5.6.3\Projects\api_load_test\testplan\api_load_test.jmx" -l "C:\Projects\apache-jmeter-5.6.3\Projects\api_load_test\testplan\results.jtl"

📁 Repository Structure
api-performance-testing/
├── README.md
├── jmeter/
│   └── api_load_test.jmx
├── results/
│   ├── load-test/
│   └── stress-test/
└── screenshots/
    ├── 01-concurrent-users.png
    ├── 02-throughput.png
    ├── 03-throughput-by-endpoint.png
    ├── 04-response-time-p90.png
    ├── 05-response-time-p95.png
    ├── 06-metrics-overview.png
    ├── 07-memory-graph.png
    ├── 08-cpu-graph.png
    └── 09-resource-stats.png

🏆 Key Results
Load: 50 users / 10 min → ~15.4 req/s → 3 ms avg → 0.00% errors

Latency: P90 ≤ 7 ms · P95 ≤ 7 ms · P99 ≤ 21.41 ms

Stress: up to 200 users → 26,023 requests → 5 ms avg → 0.04% errors

System resources: CPU 4.33% avg / 9.16% max · Memory 77.37% avg / 77.80% max

👨‍💻 Skills Demonstrated
Performance Test Planning · Load Testing · Stress Testing · Apache JMeter · REST API testing · Dynamic correlation · JSON Extractor · JSR223/Groovy · Throughput Controller · Authentication · InfluxDB · Grafana · Windows Performance Monitor · Percentile analysis · Concurrency/race-condition analysis · Performance reporting
