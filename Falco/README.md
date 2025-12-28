# Falco – Runtime Security for Kubernetes (Production Guide)
---

## 1. What is Falco

**Falco** is an **open-source runtime security tool** used to **detect suspicious and malicious behavior** in:

* Kubernetes clusters
* Containers
* Hosts

Falco focuses on **runtime threats**, meaning:

> It detects attacks **while the application is running**, not before deployment.

Examples of things Falco can detect:

* Shell access inside containers
* Privilege escalation attempts
* Unexpected file access (e.g. /etc/passwd)
* Network activity from unexpected processes

---

## 2. Why Falco is Needed

Traditional Kubernetes security tools:

* Scan images
* Validate manifests
* Enforce policies at admission time

But once a pod is running:

* Attacker may exploit the app
* Gain shell access
* Run malicious commands

Falco solves this by:

* Monitoring **system calls**
* Detecting **abnormal runtime behavior**

---

## 3. Falco Architecture (High-Level)

Falco architecture has **four main layers**:

### 1. Event Source

* Linux kernel events
* System calls (open, execve, chmod, etc.)

### 2. Kernel Instrumentation

Falco collects events using:

* eBPF (modern, preferred)
* Kernel module (legacy)

### 3. Falco Engine

* Evaluates events against Falco rules
* Decides whether behavior is suspicious

### 4. Outputs

* Logs
* Alerts
* Webhooks
* SIEM / Slack / PagerDuty

---

## 4. How Falco Works (Step-by-Step)

1. A container performs an action (example: bash execution)
2. Linux kernel generates a system call
3. Falco captures the syscall using eBPF
4. Falco rule engine evaluates the event
5. If rule matches → alert is generated

Important:

> Falco does **not block** by default, it **detects and alerts**.

---

## 5. Falco Rules (Core Concept)

Falco uses **rules written in YAML**.

Rules define:

* What behavior is allowed
* What behavior is suspicious

Example rule logic (conceptual):

* If a shell is spawned inside a container
* AND container is not expected to run a shell
* THEN trigger alert

Rules are:

* Highly customizable
* Environment-specific

---

## 6. Falco Deployment in Kubernetes

### Common Deployment Model

* Deployed as a **DaemonSet**
* Runs on every node
* Monitors all containers on that node

Why DaemonSet:

* Needs kernel access
* One Falco per node

---

## 7. Falco in Production – How We Use It

### Typical Production Flow

1. Falco runs as DaemonSet
2. Custom rules added for environment
3. Alerts sent to monitoring system
4. Security team investigates incidents

### Integration Examples

* Prometheus + Alertmanager
* Slack alerts
* SIEM tools
* Incident response pipelines

---

## 8. Falco vs Admission Controllers

Falco:

* Runtime security
* Detects active attacks

Admission Controllers (Kyverno, Gatekeeper):

* Pre-runtime enforcement
* Prevent misconfigurations

Production Best Practice:

> Use **both together**.

---

## 9. Common Use Cases (Interview Important)

### Detect Shell in Container

* exec into running pod
* Reverse shell attacks

### Detect Privilege Escalation

* sudo usage
* setuid binaries

### Detect File Tampering

* /etc/passwd access
* SSH key modifications

### Detect Network Anomalies

* Unexpected outbound connections

---

## 10. Performance and Overhead

* Uses eBPF (low overhead)
* Minimal performance impact
* Safe for production clusters

---

## 11. Falco Advantages

### 1. Runtime Visibility

* Sees what is actually happening

### 2. Kubernetes-Native

* Understands pods, namespaces, containers

### 3. Real-Time Detection

* Alerts immediately

### 4. Open Source

* No vendor lock-in

### 5. Highly Customizable

* Custom rules per app/team

---

## 12. Falco Limitations (Be Honest in Interviews)

* Detects, does not block by default
* Requires tuning to reduce noise
* Not a replacement for image scanning

---

## 13. Falco in HA and Production Security Strategy

Falco fits into:

* Defense-in-depth
* Zero trust runtime security
* Compliance monitoring

Used along with:

* Image scanning
* Admission control
* Network policies

---


This README provides a **production-level, interview-ready understanding of Falco** and how it is used for **runtime security in Kubernetes**.

