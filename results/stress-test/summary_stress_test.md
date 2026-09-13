# Stress Test Summary

**Profile:** Step load 100 → 150 → 200 concurrent users

| Metric                    | Value              |
|--------------------------|--------------------|
| Maximum Virtual Users    | 200                |
| Duration                 | ~11 min 25 sec     |
| Total Requests           | 23,756             |
| Average Throughput       | 36.0 req/s         |
| Peak Throughput          | 52.9 req/s         |
| Average Response Time    | ~6.14 ms           |
| Maximum Response Time    | 96 ms              |
| Errors                   | 4                  |
| Error Rate               | ~0.017%            |

### Key Observations
- Throughput scaled with concurrency up to ~53 req/s at 200 users
- `GET /booking review` became the main latency outlier under peak load
- Only a small number of 404 errors occurred due to race conditions on dynamic booking data
- CPU remained low (max ~16.5%), memory was more utilized
