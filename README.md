# ⚡ K8s Fault Analyzer (Lite Edition) v2.1.1

> **An air-gapped, zero-hallucination, client-side diagnostic engine for rapid Kubernetes incident triage and root-cause isolation.**

[![Version](https://img.shields.io/badge/Version-2.1.1-380bbf?style=flat-square)](https://cursorlordsystems.com)
[![Architecture](https://img.shields.io/badge/Architecture-Deterministic_PBRCIM-f0b400?style=flat-square)](https://cursorlordsystems.com)
[![Execution](https://img.shields.io/badge/Execution-100%25_Offline_%2F_Air--Gapped-green?style=flat-square)](https://cursorlordsystems.com)
[![License](https://img.shields.io/badge/License-EVAL_%2F_Proprietary-blue?style=flat-square)](https://cursorlordsystems.com)

---

## 📌 Architectural Overview

Modern production failures across Kubernetes workloads are rarely simple syntax errors. They stem from broken state assumptions, silent networking deadlocks, mutating webhook proxies, and cascading infrastructure pressure that standard application logs misdiagnose as application code crashes.

**K8s Fault Analyzer (Lite Edition) v2.1.1** is a standalone, local diagnostic utility built on top of the **Pattern-Based Root Cause Isolation Model (PBRCIM)** framework. It processes raw Kubernetes telemetry-including `kubelet` streams, `containerd`/CRI-O runtime logs, CoreDNS queries, and API server event dumps-to deliver structured, post-mortem incident reports in **less than 10 milliseconds**.

Unlike generic LLM wrappers or cloud-dependent observability agents, this tool operates **100% offline in isolated local memory**. Your cluster logs, pod identities, internal endpoints, and sensitive credentials never leave your machine.

---

## 📽 Air-Gapped Execution Proof (3-Min Technical Demo)

Below is a complete demonstration of the engine evaluating an adversarial infrastructure crash log (*Node Ephemeral Storage Pressure Eviction disguised as a Node.js `ERR_UNHANDLED_REJECTION` application panic*). 

> **Verification:** Executed with **network adapters disabled live** (`ncpa.cpl`) and zero network I/O activity.

---

> 🌐 **Interactive Platform & Licensing:** Request trial access or explore more developer utilities at [cursorlordsystems.com](https://cursorlordsystems.com).


## 🛡 Key Engineering Capabilities

* **Deterministic Zero-Hallucination Engine:** Operates on explicit failure patterns and structural logic gates. Unrecognized logs safely trigger an `UNKNOWN VECTOR` fallback state rather than inventing false telemetry.
* **Air-Gapped Privacy & Zero API Dependency:** Runs locally with zero outbound HTTP requests, protecting cluster topology and PII data from public LLM models and third-party APIs.
* **Noise Filtering & Layer Separation:** Bypasses misleading application-layer stack traces to isolate the true root cause across Layer-4 transport (mTLS/SVID), Layer-7 proxy authorization, or infrastructure resource pressure.
* **Structured Incident Output:** Generates ready-to-use SRE post-mortem reports containing Incident Context, Intent vs. Outcome traces, Root Cause analysis, `kubectl`/`crictl` remediation commands, and Audit Protocols.

---

## 🧭 Supported Diagnostic Vectors (V2.1.1)

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

## 📋 Sample k8s incident from the wild:

2026-10-03T20:14:02Z  Warning  FreeDiskSpaceFailed   node/ip-10-0-42-11.ecore  failed to garbage collect required amount of images
2026-10-03T20:14:05Z  Warning  Evicted               pod/payment-processor-78dfb9f-6kv2z  The node was low on resource: ephemeral-storage. Container payment-container was using 12GiB, which exceeds its request of 2GiB.
2026-10-03T20:14:06Z  Normal   Killing               pod/payment-processor-78dfb9f-6kv2z  Stopping container payment-container

I1003 20:14:05.102145    2412 e_runtime.go:92] "Eviction manager: conserving resources, selecting pod payment-processor-78dfb9f-6kv2z for eviction"
I1003 20:14:05.214051    2412 event.go:282] Event(v1.ObjectReference{Kind:"Pod", Namespace:"prod", Name:"payment-processor-78dfb9f-6kv2z"}): Type: 'Warning' Reason: 'Evicted' The node was low on resource: ephemeral-storage.

[2026-10-03 20:14:01.451] [INFO] [OrderWorker] Processing batch sequence #90412...
[2026-10-03 20:14:03.112] [WARN] [StateCheckpoint] Local journal disk write latency spiked: 4210ms (Threshold: 500ms)
[2026-10-03 20:14:05.159] [FATAL] [Process] [node] [ERR_UNHANDLED_REJECTION] An unhandled rejected Promise occurred!
  Stack: Error: ENOSPC: no space left on device, write
      at Object.writeSync (node:fs:981:20)
      at StateCheckpoint.flushEmergencyBackup (/app/dist/workers/checkpoint.js:42:11)
      at EvictionHook.process.on('SIGTERM') (/app/dist/index.js:108:31)


Here is the post-mortem from the K8s Fault Analyzer (Lite Edition) v2.1.1 for the above incident log:

🧠 PBRCIM Failure Classification [PBRCIM-K8S-014]
Failure Family  → Node Ephemeral Storage Exhaustion & Disk Pressure
Failure Domain  → Node Storage Infrastructure

📌 Incident Context:
• Pod → payment-processor-78dfb9f-6kv2z
• Node → ip-10-0-42-11.ecore
• Container → payment-container
• StorageUsage → 12GiB
• StorageRequest → 2GiB
• Namespace → prod

🧭 PBRCIM Trace:
• Node ip-10-0-42-11.ecore experienced local storage pressure affecting pod payment-processor-78dfb9f-6kv2z in namespace prod.
• Execution diverged when local disk usage exceeded available thresholds, forcing the kubelet eviction manager to terminate workload processes.
• Image garbage collection failed to reclaim sufficient space, leading to write failures (ENOSPC) and forced pod eviction.

🎯 Intent vs Outcome:
• Intended -> Worker node maintains adequate local disk headroom, containers operate within requested ephemeral storage limits, and disk writes complete normally.
• Outcome -> Local filesystem reaches capacity, pod payment-processor-78dfb9f-6kv2z is evicted due to ephemeral-storage exhaustion, and application write operations fail with ENOSPC.

🔬 Root Cause:
• Confirmed: Node ip-10-0-42-11.ecore evicted pod payment-processor-78dfb9f-6kv2z after container payment-container using 12GiB exceeded request 2GiB - indicating ephemeral-storage pressure (requests affect scheduling/eviction ranking; limits, if configured, enforce separately).
• Not Yet Established: Whether storage growth was driven by unrotated application logs, excessive local file caching, temporary directory writes, or accumulated container images.
• Configuration to Inspect: Verify whether pod specifies explicit ephemeral-storage limits or `emptyDir.sizeLimit` constraints (not established from this log alone).

🛠 Corrective Action:
• Step 1 — Verify Diagnosis:
  - kubectl describe node ip-10-0-42-11.ecore | grep -A10 Conditions
  - kubectl describe pod payment-processor-78dfb9f-6kv2z -n prod | grep -A10 Evicted
  - du -sh /var/log/pods/prod_payment-processor-78dfb9f-6kv2z_*  # check log size
• Step 2 — Mitigate (with operational caution):
  - Assess node: crictl images | head -20; df -h; du -sh /var/lib/containerd/*
  - Only after assessment, prune stale images if safe: crictl rmi --prune (note: does not fix app-level log growth, use with caution on production nodes)
  - Clear evicted pods (does not fix underlying storage): kubectl delete pod payment-processor-78dfb9f-6kv2z -n prod --force --grace-period=0
• Step 3 — Permanent Fix:
  - Define explicit `ephemeral-storage` requests and limits in the pod spec.
  - Configure `sizeLimit` on all `emptyDir` volumes and offload large persistent files to cloud storage or PVCs.

🛡 Prevention:
• Monitor `kubelet_volume_stats_used_bytes` paired with node filesystem metrics (df -h) and kubelet eviction monitoring, set alerts for node disk usage >80%, enforce ephemeral storage limits via admission control, and configure log rotation policies.

📋 Audit Protocol:
• Inspect node conditions for DiskPressure, pod eviction reasons, node filesystem utilization (`df -h`), container log sizes, and `emptyDir` volume consumption.

✅ Verification Check:
• Node ip-10-0-42-11.ecore clears DiskPressure conditions, pod payment-processor-78dfb9f-6kv2z reschedules and runs stably within assigned storage limits, and write operations complete without ENOSPC errors.


🌐 Portfolio & Case Studies: cursorlordsystems.com
📬 Diagnostic Consultation: contact@cursorlordsystems.com
(C) 2026 CursorLord Systems. Built for air-gapped, zero-trust infrastructure triage.

================================================================

