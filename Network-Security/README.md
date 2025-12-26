# cert-manager, Let’s Encrypt & CNI Network Policies – Production Guide


* What cert-manager is and why it is used
* What Let’s Encrypt is
* How cert-manager works internally
* Certificate lifecycle in Kubernetes
* Integration with Ingress
* What CNI Network Policies are
* Types of CNI / Kubernetes / Cilium policies
* How policies are used in production
* Best practices & common mistakes

---

## 1. Why TLS & Network Security Matter in Kubernetes

In production Kubernetes clusters:

* Traffic must be **encrypted (HTTPS / mTLS)**
* Services must be **isolated (zero trust)**
* Manual certificate & firewall management **does not scale**

This is where **cert-manager** and **CNI network policies** come in.

---

## 2. What is cert-manager?

**cert-manager** is a Kubernetes controller that **automates the creation, renewal, and management of TLS certificates** inside Kubernetes.

It removes the need to:

* Manually generate certificates
* Manually renew certificates
* Manually update secrets

cert-manager works by:

* Watching Kubernetes resources
* Talking to Certificate Authorities (CAs)
* Storing certificates as Kubernetes Secrets

---

## 3. What is Let’s Encrypt?

**Let’s Encrypt** is a **free, automated public Certificate Authority (CA)**.

It provides:

* Trusted TLS certificates
* Automatic issuance via ACME protocol
* Short-lived certificates (90 days)

In Kubernetes:

* cert-manager uses Let’s Encrypt as the CA

---

## 4. cert-manager Architecture (Internal Working)

Key components:

1. **cert-manager controller**
2. **Webhook** – validation
3. **CA injector** – injects CA bundles
4. **Issuers / ClusterIssuers**
5. **Certificates (CRD)**

cert-manager runs as Pods in the cluster.

---

## 5. Issuer vs ClusterIssuer

### Issuer

* Namespace-scoped
* Used by apps in one namespace

### ClusterIssuer

* Cluster-wide
* Preferred in production

Production clusters almost always use **ClusterIssuer**.

---

## 6. How Certificate Issuance Works (Step-by-Step)

Example: HTTPS for `app.example.com`

1. Ingress references a TLS secret
2. cert-manager sees missing certificate
3. cert-manager creates a Certificate CR
4. ACME challenge is created
5. Let’s Encrypt validates domain
6. Certificate is issued
7. TLS Secret is created/updated
8. Ingress starts serving HTTPS

Certificates are **auto-renewed** before expiry.

---

## 7. ACME Challenge Types

### HTTP-01

* Uses HTTP endpoint
* Most common with Ingress

### DNS-01

* Uses DNS records
* Required for wildcard certificates
* Common in production

---

## 8. cert-manager + Ingress (Typical Production Setup)

Production pattern:

* Ingress Controller (NGINX / Traefik / Istio)
* cert-manager for certificates
* Let’s Encrypt as CA

Ingress only **references TLS secrets**.

cert-manager handles everything else.

---

## 9. Certificate Lifecycle in Kubernetes

1. Request created
2. Certificate issued
3. Stored as Secret
4. Used by Ingress / Gateway
5. Renewed automatically
6. Old certificate replaced

No downtime if configured correctly.

---

## 10. What are CNI Network Policies?

CNI Network Policies control **network traffic between Pods**.

They define:

* Who can talk to whom
* On which ports
* In which direction

They are enforced by the **CNI plugin**, not Kubernetes itself.

---

## 11. Types of Network Policies

### 11.1 Kubernetes NetworkPolicy

* Standard Kubernetes API
* L3/L4 only (IP + Port)
* Enforced by CNI (Calico, Cilium, etc.)

---

### 11.2 Cilium Network Policy (Advanced)

* L3–L7 support
* Identity-based (labels)
* DNS & HTTP-aware
* Enforced using eBPF

---

## 12. Ingress vs Egress Policies

### Ingress

* Controls traffic **coming into a Pod**

### Egress

* Controls traffic **leaving a Pod**

Production clusters use **both**.

---

## 13. How Network Policies Work Internally

1. Pod sends packet
2. Packet intercepted by CNI
3. Policy rules evaluated
4. Traffic allowed or dropped

If dropped:

* Connection timeout
* Visible in logs / observability tools

---

## 14. Production Use of Network Policies

### Common patterns

* Default deny all traffic
* Explicit allow rules
* Namespace isolation
* Service-to-service allowlists
* Restricted egress to internet

---

## 15. cert-manager + Network Policies Together

Important production consideration:

* cert-manager needs:

  * DNS access
  * HTTP/HTTPS access

Network policies must allow:

* cert-manager → DNS
* cert-manager → Let’s Encrypt endpoints

Otherwise certificate issuance fails.

---

## 16. Production Best Practices

### TLS / cert-manager

* Use ClusterIssuer
* Prefer DNS-01 for wildcards
* Monitor certificate expiry
* Rate-limit Let’s Encrypt usage

### Network Policies

* Start with default deny
* Always allow DNS
* Use labels, not IPs
* Test in staging first
* Use observability tools

---

## 17. Common Mistakes

❌ Manual certificate management
❌ Using NodePort without TLS
❌ Blocking DNS via policies
❌ Overly broad allow rules
❌ No policy documentation

---

## 18. Interview One-Liners (VERY IMPORTANT)

### cert-manager

> “cert-manager automates TLS certificate issuance and renewal in Kubernetes by integrating with Certificate Authorities like Let’s Encrypt.”

### Let’s Encrypt

> “Let’s Encrypt is a free public CA that issues trusted TLS certificates using the ACME protocol.”

### Network Policies

> “CNI network policies enforce pod-level traffic control in Kubernetes, enabling zero-trust security inside the cluster.”


#

