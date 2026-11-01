# FlowSight – Network Anomaly Detection

**Isolation Forest vs. Autoencoder**, both implemented from scratch in Python/NumPy, compared in a controlled CPU benchmark on network-flow data. The project also includes a SOC analyst training website.

Course project: *AI in Cybersecurity* (Summer Semester 2026), Lecturer: Andrei Kojukhov.
Authors: Irvin Rosenfeld, Magen Hai Lakau.

> **Note:** the course is based on NVIDIA Morpheus, but this implementation does **not** use Morpheus, RAPIDS, Triton or a GPU. See [Limitations](#limitations).

## Research question

How much inference speed can we gain without increasing alert noise?

## Approach

```
EVE JSON flows → 18 features → Standardization → Anomaly scores → Thresholds
```

| | Isolation Forest | Autoencoder |
|---|---|---|
| Implementation | Custom Python / NumPy | Custom Python / NumPy |
| Model | 100 trees, 256 samples per tree | 18–6–18 network, tanh encoder, linear decoder |
| Learning | Random partitioning of normal flows | Reconstruction of normal flows |
| Configuration | Seed 42 | Adam, 60 epochs |

Both models share the same feature scaling and event order, so the comparison is controlled.

## Experimental setup

- **Training:** 3,000 normal flows
- **Evaluation:** 5,000 labeled flows (4,600 normal, 400 anomalies: scanning, UDP flooding, large outbound transfers)
- **Data:** synthetic, Suricata-style EVE JSON flows
- **Settings:** seed 42, calibration contamination 0.02, inference batch size 1
- **Timing:** inference only (preprocessing and file I/O excluded)
- **Environment:** Windows 11, Python 3.12.13, NumPy 2.3.5, 20 logical CPUs

## Results

| Metric | Isolation Forest | Autoencoder |
|---|---|---|
| P95 inference latency | 0.1623 ms | 0.0083 ms (**19.6× lower**) |
| Throughput (flows/s) | 8,060 | 113,241 |
| Recall | 100% (400/400) | 100% (400/400) |
| Precision | 79.37% | 78.74% |
| F1 score | 88.50% | 88.11% |
| False positives | 104 | 108 |
| False-positive rate | 2.26% | 2.35% |

**Takeaways**
- The Autoencoder had much lower inference latency in this run.
- Both models detected all 400 anomalies, but roughly one in five alerts was a false alarm.
- The F1 difference (0.39 percentage points) is small, and no significance test was performed.

## SOC training website

A practice tool for analysts, using scripted scenarios (separate from the measured experiment):
1. Compare the models (latency, detection quality, confusion matrices)
2. Explore a load estimate against measured inference capacity
3. Investigate a SOC scenario using the evidence and a checklist
4. Record the analyst decision and download a practice report

The website has no live model API, and the email examples are not a trained phishing classifier.

Live demo: https://flowsight-streaming-lab.liliannbalbir.chatgpt.site/

## MITRE ATT&CK interpretation

The models output anomaly scores only. They do not predict ATT&CK labels. Analyst mappings of the synthetic scenarios:

| Scenario | Candidate technique |
|---|---|
| Port-scan-like TCP flows | T1046 Network Service Discovery |
| High-rate UDP flows | T1498.001 Direct Network Flood |
| Large outbound TLS transfer | T1048 only conditionally, needs more context |

## Limitations

- A single run on synthetic data. Synthetic patterns may be easier to separate than real traffic.
- No confidence intervals, no repeated seeds.
- No Morpheus, RAPIDS, Triton or GPU benchmark.
- Exact CPU model and process memory usage were not recorded.

## Future work

- Evaluate on real labeled traffic with repeated seeds
- Measure end-to-end delay
- Implement Morpheus / RAPIDS / Triton before comparing CPU and GPU

## Repository structure

<!-- TODO: update to match your actual files -->
```
.
├── models/        # Isolation Forest and Autoencoder (NumPy)
├── data/          # synthetic EVE JSON generator / sample data
├── benchmark/     # evaluation and timing scripts
├── website/       # SOC training website
└── README.md
```

## How to run

<!-- TODO: fill in with your real commands -->
```bash
pip install numpy
python benchmark/run.py
```

## Contributions

<!-- TODO: fill in the real split -->
- **Magen Hai Lakau:** ...
- **Irvin Rosenfeld:** ...

## References

- [Suricata EVE JSON format](https://docs.suricata.io/en/suricata-8.0.0/output/eve/eve-json-format.html)
- [MITRE ATT&CK T1046](https://attack.mitre.org/techniques/T1046/), [T1498.001](https://attack.mitre.org/techniques/T1498/001/), [T1048](https://attack.mitre.org/techniques/T1048/)
