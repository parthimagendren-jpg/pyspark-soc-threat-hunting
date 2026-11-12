# PySpark SOC Threat Hunting — Malware / Insider Threat

**TPD3753 Big Data Analytics and Development — Individual Laboratory Test**
Author: JEYAGANESH ASOKKUMAR (2240309)

A big-data threat hunt on 40,000 security events with Apache Spark (PySpark). The notebook builds an ETL pipeline, injects 130 labelled attack records, runs five threat-hunting investigations, uses window functions to detect beaconing and brute force, maps the findings to MITRE ATT&CK and reports them in a SOC incident report.

## Key findings

| Finding | Risk | Evidence |
|---|---|---|
| Malware on finance PC 10.10.3.15: command-and-control beaconing, then FTP data theft | Critical | Beacon every 299.9 s ± 3.4 s; 20 Critical FTP uploads, all alerts ignored |
| Port scan and SSH brute force from 203.0.113.45 | High | 30 ports probed in 116 s; 50 failed logins in 588 s; 0 blocked |
| Insider `hr.user021` copied 10 salary files at night | High | Data volume z-score 31.0 vs. peers, although every event scored Low risk |

![Attack timeline](charts/chart1_attack_timeline.png)
![High-risk source ranking](charts/chart2_high_risk_sources.png)

## Repository contents

| Path | What it is |
|---|---|
| `notebook/TPD3753_LabTest_PySpark_Notebook.ipynb` | Full lab (Parts 1–10) with executed outputs |
| `data/analytics_dataset.parquet` | Cleaned analytics dataset: 40,000 rows × 30 columns |
| `data/threat_hunting_dataset.parquet` | Analytics dataset plus 130 injected attack records (`IsInjected = 1`) |
| `charts/` | Attack timeline and high-risk source ranking (PNG) |
| `report/SOC_Incident_Report.pdf` | SOC incident report: executive summary, key findings, risk assessment, recommendations |
| `docs/Video_Script_and_Viva_Prep.md` | Video defence script, oral-verification practice, declaration and requirement audit |

## How to run (Google Colab)

1. Download `cybersecurity_attacks.csv` from Kaggle: [Cyber Security Attacks (Incribo)](https://www.kaggle.com/datasets/teamincribo/cyber-security-attacks). It is not stored in this repository.
2. Open `notebook/TPD3753_LabTest_PySpark_Notebook.ipynb` in Google Colab.
3. Choose **Runtime → Run all**, then upload the CSV when the notebook asks for it.

The notebook is deterministic, so a correct run shows: 40,000 records; 2,268 Critical events; 40,130 records after injection; peak day 2023-06-14 with 136 events.

To run locally instead: `pip install -r requirements.txt` (PySpark also needs Java 17 or later for Spark 4, or Java 8/11/17 for Spark 3.5).

## Method

1. **ETL:** `raw_dataset.csv` → cleaning with `dropDuplicates()`, `fillna()`, `replace()` and `cast()` → features `RiskScore` (0–100), `ThreatLevel`, `AttackHour`, `AttackDate` and `IsOffHours` → `analytics_dataset.parquet`.
2. **Attack injection:** 130 records across five scenarios (Brute Force 50, Reconnaissance 30, Data Exfiltration 20, Malware Beaconing 20, Insider Threat 10) with realistic timing. External hosts use reserved RFC 5737 documentation addresses.
3. **Threat hunting:** five investigations (threat actor, asset targeting, temporal, risk, behavioural anomaly) using `filter()`, `groupBy()`, `agg()`, `join()`, `orderBy()` and Spark SQL. Hunting queries never use the injection labels; the labels are used only to validate the results.
4. **Window functions:** `lag()` for beacon timing, `row_number()` for brute-force attempts, `rank()` for insider ranking per network segment.
5. **MITRE ATT&CK:** T1595, T1110.001, T1071.001, T1048.003, T1213.

## Data note

The Kaggle dataset is synthetic: every IP address appears only once, so the attack campaigns come from the labelled injected records.
