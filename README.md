# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

 This aiops_pipeline is taking the service-dat from payment-service. The events are being processed through a pipeline where the producer produces the event the consumer consumes the even anomalies are detected.
### 1) Service being monitored
- The monitored service is the payment-service.
- It is a transaction-processing service whose health is tracked through:
  - response time
  - CPU utilization
  - memory utilization
  - log events such as errors and warnings

### 2) Operational problem being addressed
- The main issue is degraded service health and potential failure in payment processing.
- The telemetry shows symptoms like:
  - unusually slow responses
  - high CPU and memory usage
  - timeout conditions and database connection errors
- In operational terms, the pipeline is trying to catch incidents early before they become full outages or customer-facing failures.

### 3) Purpose of AIOps in this assessment
- AIOps is used to automate anomaly detection from live data.
- Instead of waiting for manual investigation,the system identifies patterns that suggest a problem as anomaly events.
- The goal is to support faster detection, triage, and response to service issues in a modern operations environment.

## Operational data

### 1) Fields that represent metrics
- response_time_msp
- cpu_percent
- memory_percent

### 2) Fields that represent log information
- log_level
- message
### 3) How timestamps are used
- timestamp is the time marker for each observation.
- it appears to be used as a sequence of minute-by-minute events.
- it helps the system track changes over time and identify when the anomaly occurs.

### 4) Observations that appear normal
The normal-looking records are the ones with:
- low response time: roughly 120–150 ms
- CPU around 40-60%
- memory around 50–60%
- log_level = INFO
- message = "Payment request processed successfully"

These are the first, second, third, fourth, fifth, eighth, ninth, and tenth records.

### 5) Observations that appear unusual
The unusual records are the ones around 10:05 and 10:06:

  {
    "timestamp": "2026-09-20T10:05:00",
    "service": "payment-service",
    "response_time_ms": 610,
    "cpu_percent": 75,
    "memory_percent": 70,
    "log_level": "ERROR",
    "message": "Payment service timeout"
  },

{
    "timestamp": "2026-09-20T10:06:00",
    "service": "payment-service",
    "response_time_ms": 640,
    "cpu_percent": 94,
    "memory_percent": 91,
    "log_level": "ERROR",
    "message": "Database connection timeout"
  },

These are unusual because they show:
- a high increase in latency
- high CPU and memory 
- error-level logs
- timeout failures

## Part 3
- I added a print step to every reponse sent by the anomaly_detector for each event to analyse if it is processing the events correctly.
- So these were the observations:
- The unflagged events whose metrics had values less than the given threshold were given the response None.
- The flagged events (flagged as ANOMALY) had values greater than the defined thresholds. The    response for them contained the reasons for which they were flagged as ANOMALY.
- {'timestamp': '2026-09-20T10:05:00', 'service': 'payment-service', 'type': 'ANOMALY', 'reasons': ['High response time'], 'source': {'timestamp': '2026-09-20T10:05:00', 'service': 'payment-service', 'response_time_ms': 610, 'cpu_percent': 75, 'memory_percent': 70, 'log_level': 'ERROR', 'message': 'Payment service timeout'}}
- {'timestamp': '2026-09-20T10:06:00', 'service': 'payment-service', 'type': 'ANOMALY', 'reasons': ['High response time', 'High CPU utilization', 'High memory utilization'], 'source': {'timestamp': '2026-09-20T10:06:00', 'service': 'payment-service', 'response_time_ms': 640, 'cpu_percent': 94, 'memory_percent': 91, 'log_level': 'ERROR', 'message': 'Database connection timeout'}}

- Although the anomaly_detector.py is giving the desired results, one improvement that I can suggest is lowering the response time threshold. From my observation the normal response time for a usual event is less than 300 and 400 ms. It only rises beyond this in case of an Anomaly or Error. This can help save the time in waiting for the response and improve the latency.

## Part 4
The aiops_pipeline.py does not display the detected events correctly.
- Fixed it:
- Updated the pipeline to process anomalies through one shared in-memory topic so the producer and consumer operate on the same event stream. This fixes the broken end-to-end flow, allowing detected anomalies to be published, consumed, and reported as operational issues.
- for event in result["anomalies_detected"]:
        print(f"\nService: {event['service']}")
        print(f"Timestamp: {event['timestamp']}")
        print(f"Type: {event['type']}")
        print(f"Reasons: {', '.join(event['reasons'])}")
---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

