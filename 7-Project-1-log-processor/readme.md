
## 🚀 **Problem Statement: Log Processor App**

### 🧩 **Goal**

You need to deploy a **Log Processor application** inside Kubernetes that simulates a small real-world logging microservice system.
This system will:

* Continuously generate logs.
* Use a sidecar container to display them.
* Store logs in a shared volume.
* Load configuration from ConfigMap and Secret.
* Be resource-limited and monitored with probes.
* Run a CronJob every minute to clean up logs.

You must apply all your Kubernetes fundamentals — **architecture, scheduling, resources, lifecycle, probes, and volumes** — in this one small project.

---

## 🏗️ **System Architecture**

You’ll create the following resources:

| Type          | Name            | Purpose                                          |
| ------------- | --------------- | ------------------------------------------------ |
| **Namespace** | `demo`          | Isolate this environment                         |
| **ConfigMap** | `log-config`    | Store log message text                           |
| **Secret**    | `log-secret`    | Store mock API key                               |
| **Pod**       | `log-processor` | Main workload with 2 containers (main + sidecar) |
| **Volume**    | `emptyDir`      | Shared between both containers                   |
| **CronJob**   | `cleanup-logs`  | Cleans logs every minute                         |

---

## ⚙️ **Detailed Requirements**

### 🧱 1. Namespace

Create a namespace named **`demo`**.

---

### 🧾 2. ConfigMap

Create a ConfigMap named **`log-config`** in the `demo` namespace with the following data:

| Key         | Value                         |
| ----------- | ----------------------------- |
| LOG_MESSAGE | Processing logs from app v1.0 |

---

### 🔑 3. Secret

Create a Secret named **`log-secret`** in the same namespace with:

| Key     | Value       |
| ------- | ----------- |
| API_KEY | 12345-ABCDE |

---

### 📦 4. Pod — `log-processor`

The Pod runs two containers:

#### **Container 1: app**

| Field                 | Value                                                                 |
| --------------------- | --------------------------------------------------------------------- |
| Name                  | app                                                                   |
| Image                 | `busybox:latest`                                                      |
| Command               | `while true; do echo $LOG_MESSAGE >> /shared/logs.txt; sleep 5; done` |
| Environment Variables | From ConfigMap (`LOG_MESSAGE`) and Secret (`API_KEY`)                 |
| Volume Mount          | Mount `/shared` using `emptyDir` volume                               |
| CPU Request           | 100m                                                                  |
| Memory Request        | 64Mi                                                                  |
| CPU Limit             | 200m                                                                  |
| Memory Limit          | 128Mi                                                                 |
| Liveness Probe        | `cat /shared/logs.txt` (check every 10s after 5s delay)               |

#### **Container 2: sidecar-logger**

| Field        | Value                          |
| ------------ | ------------------------------ |
| Name         | sidecar-logger                 |
| Image        | `busybox:latest`               |
| Command      | `tail -f /shared/logs.txt`     |
| Volume Mount | Same `/shared` emptyDir volume |

#### **Pod-Level Specs**

| Field         | Value                                                                         |
| ------------- | ----------------------------------------------------------------------------- |
| Namespace     | demo                                                                          |
| restartPolicy | Always                                                                        |
| nodeSelector  | `kubernetes.io/hostname: node1` (you can replace `node1` with your node name) |

---

### 🧹 5. CronJob — `cleanup-logs`

This CronJob will run every **1 minute** to delete the logs file.

| Field                   | Value                                                  |   |       |
| ----------------------- | ------------------------------------------------------ | - | ----- |
| Schedule                | `*/1 * * * *`                                          |   |       |
| Container Image         | `busybox:latest`                                       |   |       |
| Command                 | `echo 'Cleaning old logs...' && rm -f /shared/logs.txt |   | true` |
| restartPolicy           | Never                                                  |   |       |
| backoffLimit            | 1                                                      |   |       |
| Volume                  | Same `emptyDir` (for simulation)                       |   |       |
| ttlSecondsAfterFinished | 30                                                     |   |       |

---
