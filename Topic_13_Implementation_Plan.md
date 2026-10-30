# Implementation Plan: N8N Hands-On Intermediate Assessment
## Topic 13: Error Handling, Retries & Recovery Strategies

---

## 1. Objective
Build an enterprise Dead Letter Queue (DLQ) and self-healing recovery workflow in n8n. The system captures fatal errors using the Error Trigger node, extracts execution telemetry and unmutated payload data, persists records to a DLQ store, classifies errors as Transient vs. Permanent, automatically replays recoverable transactions, and escalates permanent failures to engineering support.

---

## 2. Canvas Architecture Overview
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

## 3. Prerequisites & Environment Setup
* [x] n8n instance version 1.0.0 or higher.
* [x] Pure JavaScript execution engine (no disallowed external packages).
* [x] Embedded pinData for instant visual testing and validation.

---

## 4. Step-by-Step Practical Build Guide

### Step 1: Configure Primary Execution Chain & Fault Injection
* **`Manual_Trigger`**: Initiates pipeline.
* **`Code_SimulateOrder`**: Stages transaction with `simulateFault: "GATEWAY_TIMEOUT_504"`.
* **`Code_ExecuteRiskyProcess`**: Simulates network retry exhaustion and preserves error state.

### Step 2: Global Crash Listener & Diagnostic Extraction
* **`Error_Trigger`**: Listens globally across workflow execution for unhandled failures.
* **`Code_ExtractDiagnostics`**:
  - Parses failed node name, error message, and HTTP status code.
  - Classifies errors: 502/503/504/429/timeout $\rightarrow$ `AUTOMATED_REPLAY`; schema/400/401/422 $\rightarrow$ `MANUAL_INVESTIGATION`.

### Step 3: Dead Letter Queue (DLQ) Persistence
* **`Code_PersistToDLQ`**:
  - Inserts entry into `staticData.dlqStore`.
  - Assigns unique `dlqId` and initializes `retryCount: 0`.

### Step 4: Recovery Classification Switch
* **`Switch_ClassifyRecovery`**:
  - **Output 0 (`AUTOMATED_REPLAY`):** Directs transient blips to replay.
  - **Output 1 (`MANUAL_INVESTIGATION`):** Directs fatal poison pills to SRE escalation.

### Step 5: Recovery Handlers
* **`Code_ScheduleReplay`**: Calculates exponential backoff (10s) and dispatches replay task.
* **`Code_EscalateIncident`**: Builds diagnostic incident alert for on-call SRE support.

### Step 6: Master Recovery Telemetry Audit
* **`Code_RecoveryAudit`**: Confirms failsafe engagement, zero data loss, and logs quarantine record.
