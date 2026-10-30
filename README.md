# Topic 13: Error Handling, Retries & Recovery Strategies
**n8n Hands-On Assessment - Level 2**

[![n8n Version](https://img.shields.io/badge/n8n-v1.0%2B-EA4B71?logo=n8n)](https://n8n.io/)
[![Status](https://img.shields.io/badge/Status-Completed-success)](https://github.com/Rushikesh-karche/Topic-13-Error-Handling-Retries-Recovery)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 📌 Project Overview
This repository contains the complete implementation for **Topic 13: Error Handling, Retries & Recovery Strategies**. It demonstrates an enterprise Dead Letter Queue (DLQ) and self-healing automated recovery architecture in n8n.

The workflow captures fatal pipeline errors using the Error Trigger node, extracts forensic diagnostic telemetry and raw payload data, persists incidents into a DLQ store, classifies errors as Transient vs. Permanent, automatically schedules recoverable transactions for delayed replay, and escalates permanent poison-pill errors to engineering support.

---

## 📂 Repository Contents
| File | Description |
| :--- | :--- |
| **[`Topic 13 - Error Handling & DLQ Recovery.json`](./Topic%2013%20-%20Error%20Handling%20%26%20DLQ%20Recovery.json)** | Complete 10-node n8n workflow with pre-configured pinData |
| **[`n8n_topic_13_requirement_document.md`](./n8n_topic_13_requirement_document.md)** | Full technical requirement specification & test matrix |
| **[`n8n_topic_13_requirement_document.pdf`](./n8n_topic_13_requirement_document.pdf)** | Formatted, print-ready requirement documentation PDF |
| **[`Topic_13_Implementation_Plan.md`](./Topic_13_Implementation_Plan.md)** | Step-by-step practical build guide and node blueprints |
| **[`Topic_13_Implementation_Plan.pdf`](./Topic_13_Implementation_Plan.pdf)** | Official assessment reference document |

---

## 🏗️ Workflow Architecture
```text
[ N13-01: Manual_Trigger ]
            |
            v
[ N13-02: Code_SimulateOrder ]
            |
            v
[ N13-03: Code_ExecuteRiskyProcess ]
            |
            v
[ N13-04: Error_Trigger ] ──> [ N13-05: Code_ExtractDiagnostics ]
                                       |
                                       v
                                 [ N13-06: Code_PersistToDLQ ]
                                       |
                                       v
                                 [ N13-07: Switch_ClassifyRecovery ]
                                       /                    \
                       (AUTOMATED_REPLAY)          (MANUAL_INVESTIGATION)
                                     /                        \
                                    v                          v
                        [ N13-08A: Code_Schedule      [ N13-08B: Code_Escalate
                          Replay ]                      Incident ]
                                    \                          /
                                     \                        /
                                      v                      v
                                 [ N13-09: Code_RecoveryAudit ]
```

---

## 🧪 Validation & Test Matrix
1. **Scenario 1 (Transient 504 Timeout - Self-Healing Replay):**
   - Error: `ETIMEDOUT` / HTTP 504.
   - Classification: `isTransientFault: true`, `recoveryCategory: "AUTOMATED_REPLAY"`.
   - Result: Stored in DLQ, routed to `Code_ScheduleReplay`, 10s backoff scheduled, master audit confirms zero data loss.
2. **Scenario 2 (Permanent Schema Error - Poison-Pill Quarantine):**
   - Error: `VALIDATION_ERROR` / HTTP 422.
   - Classification: `isTransientFault: false`, `recoveryCategory: "MANUAL_INVESTIGATION"`.
   - Result: Quarantined in DLQ, routed to `Code_EscalateIncident`, flagged "Do not retry automatically".

---

## 🚀 How to Import into n8n
1. Download **`Topic 13 - Error Handling & DLQ Recovery.json`**.
2. Open n8n, click **+ Add Workflow** $\rightarrow$ **Import from File**.
3. Click **Test Workflow**—all nodes execute instantly using embedded test pinData!
