# ⚡ K8s Fault Analyzer (Lite Edition) v2.1.1

> **An air-gapped, zero-hallucination, client-side diagnostic engine for rapid Kubernetes incident triage and root-cause isolation.**

[![Version](https://img.shields.io/badge/Version-2.1.1-380bbf?style=flat-square)](https://cursorlordsystems.com)
[![Architecture](https://img.shields.io/badge/Architecture-Deterministic_PBRCIM-f0b400?style=flat-square)](https://cursorlordsystems.com)
[![Execution](https://img.shields.io/badge/Execution-100%25_Offline_%2F_Air--Gapped-green?style=flat-square)](https://cursorlordsystems.com)
[![License](https://img.shields.io/badge/License-EVAL_%2F_Proprietary-blue?style=flat-square)](https://cursorlordsystems.com)

---

## 📌 Architectural Overview

Modern production failures across Kubernetes workloads are rarely simple syntax errors. They stem from broken state assumptions, silent networking deadlocks, mutating webhook proxies, and cascading infrastructure pressure that standard application logs misdiagnose as application code crashes.

**K8s Fault Analyzer (Lite Edition) v2.1.1** is a standalone, local diagnostic utility built on top of my **Pattern-Based Root Cause Isolation Model (PBRCIM)** framework. It processes raw Kubernetes telemetry—including `kubelet` streams, `containerd`/CRI-O runtime logs, CoreDNS queries, and API server event dumps—to deliver structured, post-mortem incident reports in **less than 10 milliseconds**.

Unlike generic LLM wrappers or cloud-dependent observability agents, this tool operates **100% offline in isolated local memory**. Your cluster logs, pod identities, internal endpoints, and sensitive credentials never leave your machine.

---

## 📽 Air-Gapped Execution Proof (120-Sec Demo)

Below is a demonstration of the engine evaluating an adversarial infrastructure crash log (*Node Ephemeral Storage Pressure Eviction disguised as a Node.js `ERR_UNHANDLED_REJECTION` application panic*). 

> **Verification:** Executed with **Wi-Fi completely disabled** and system monitor displaying zero network I/O activity.

https://github.com/user-attachments/assets/your-video-link-here.mp4

---

## 🛡 Key Engineering Capabilities

* **Deterministic Zero-Hallucination Engine:** Operates on explicit failure patterns and structural logic gates. Unrecognized logs safely trigger an `UNKNOWN VECTOR` fallback state rather than inventing false telemetry.
* **Air-Gapped Privacy & Zero API Dependency:** Runs locally with zero outbound HTTP requests, protecting cluster topology and PII data from public LLM models and third-party APIs.
* **Noise Filtering & Layer Separation:** Bypasses misleading application-layer stack traces to isolate the true root cause across Layer-4 transport (mTLS/SVID), Layer-7 proxy authorization, or infrastructure resource pressure.
* **Structured Incident Output:** Generates ready-to-use SRE post-mortem reports containing Incident Context, Intent vs. Outcome traces, Root Cause analysis, `kubectl`/`crictl` remediation commands, and Audit Protocols.

---

## 🧭 Supported Diagnostic Vectors (v2.1.1)

The v2.1.1 Lite release covers **10 core Kubernetes failure vectors**, accounting for approximately 80% of daily production crash scenarios:

| Rule Vector ID | Failure Category | Core Detection Mechanism |
| :--- | :--- | :--- |
| **`PBRCIM-K8S-002`** | **Admission & Policy Drop** | Mutating/Validating Webhook rejection and CVE admission blocks |
| **`PBRCIM-K8S-003`** | **Kernel Resource Eviction** | Linux OOMKiller limit breach (`Exit Code 137`) |
| **`PBRCIM-K8S-004`** | **Storage Volume Drift** | PV/PVC mount failures, stale attachments, and driver timeouts |
| **`PBRCIM-K8S-005`** | **Topology Starvation** | Scheduler constraint deadlocks, affinity mismatches, and node capacity exhaustion |
| **`PBRCIM-K8S-006`** | **Registry Auth Divergence** | ImagePullBackOff, 401/403 challenges, and proxy token mutation |
| **`PBRCIM-K8S-007`** | **Cluster RBAC Enforcement** | Identity permission denials, missing ServiceAccount tokens, and API 403s |
| **`PBRCIM-K8S-012`** | **Workload Config Resolution** | Missing ConfigMaps, unlinked Secrets, and subPath volume mount drops |
| **`PBRCIM-K8S-013`** | **Kubelet & Node Readiness** | Node `NotReady` states, PLEG health degradation, and runtime drops |
| **`PBRCIM-K8S-014`** | **Node Storage Eviction** | Ephemeral storage limit breaches, disk pressure, and `ENOSPC` writes |
| **`PBRCIM-K8S-021`** | **Control Plane TLS Expiration** | API server/etcd x509 certificate expiry and mTLS handshake rejection |

---

## 📋 Sample Diagnostic Output

```markdown
🧠 PBRCIM Failure Classification [PBRCIM-K8S-014]
Failure Family  → Node Ephemeral Storage Exhaustion & Disk Pressure
Failure Domain  → Node Storage Infrastructure

📌 Incident Context:
- Pod → payment-processor-78dfb9f-6kv2z
- Node → ip-10-0-42-11.ecore

🧭 PBRCIM Trace:
- Node ip-10-0-42-11.ecore experienced local storage pressure affecting pod payment-processor-78dfb9f-6kv2z in namespace default.
- Execution diverged when local disk usage exceeded available thresholds, forcing the kubelet eviction manager to terminate workload processes.
- Image garbage collection failed to reclaim sufficient space, leading to write failures (ENOSPC) and forced pod eviction.

🎯 Intent vs Outcome:
- Intended -> Worker node maintains adequate local disk headroom, containers operate within requested ephemeral storage limits, and disk writes complete normally.
- Outcome -> Local filesystem reaches capacity, pod payment-processor-78dfb9f-6kv2z is evicted due to ephemeral-storage exhaustion, and application write operations fail with ENOSPC.

🔬 Root Cause:
- Confirmed: Node ip-10-0-42-11.ecore evicted pod payment-processor-78dfb9f-6kv2z after local ephemeral storage utilization exceeded assigned resource requests/limits.
- Not Yet Established: Whether storage growth was driven by unrotated application logs, excessive local file caching, temporary directory writes, or accumulated container images.
- Contributing Config: Pod specification missing explicit ephemeral-storage limits or emptyDir.sizeLimit constraints.

🛠 Corrective Action:
- Step 1 — Verify Diagnosis:
  - kubectl describe node ip-10-0-42-11.ecore | grep -A10 Conditions
  - kubectl describe pod payment-processor-78dfb9f-6kv2z -n default | grep -A10 Evicted
- Step 2 — Mitigate:
  - Safely prune stale images on affected node: crictl rmi --prune
  - Clear evicted pods: kubectl delete pod payment-processor-78dfb9f-6kv2z -n default --force --grace-period=0
- Step 3 — Permanent Fix:
  - Define explicit ephemeral-storage requests and limits in the pod spec.
  - Configure sizeLimit on all emptyDir volumes and offload large persistent files to cloud storage or PVCs.

🛡 Prevention:
- Monitor kubelet_volume_stats_used_bytes, set alerts for node disk usage >80%, enforce ephemeral storage limits via admission control, and configure log rotation policies.

📋 Audit Protocol:
- Inspect node conditions for DiskPressure, pod eviction reasons, node filesystem utilization (df -h), container log sizes, and emptyDir volume consumption.

✅ Verification Check:
- Node ip-10-0-42-11.ecore clears DiskPressure conditions, pod payment-processor-78dfb9f-6kv2z reschedules and runs stably within assigned storage limits, and write operations complete without ENOSPC errors.



🌐 Portfolio & Case Studies: cursorlordsystems.com
📬 Diagnostic Consultation: contact@cursorlordsystems.com
(C) 2026 CursorLord Systems. Built for air-gapped, zero-trust infrastructure triage.

================================================================
