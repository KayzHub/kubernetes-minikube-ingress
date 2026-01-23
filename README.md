# Testing Kubernetes Dashboard Ingress on Minikube

This guide explains how to expose the Kubernetes Dashboard using an Ingress resource with the custom domain `dashboard.com`.

It includes **driver-specific instructions** so it works correctly on:

- macOS / Windows (Docker driver)
- Linux / VM drivers

---

## Prerequisites

- Minikube installed and running:
  ```bash
  minikube start
  ```

- Ingress addon enabled:
  ```bash
  minikube addons enable ingress
  ```

- Kubernetes Dashboard addon enabled:
  ```bash
  minikube dashboard
  ```

- Admin / sudo privileges (for editing hosts file)

---

## Step 1 – Apply the Ingress Manifest

```bash
kubectl apply -f dashboard-ingress.yaml
```

Verify:

```bash
kubectl get ingress -n kubernetes-dashboard
```

---

## Step 2 – Determine your Minikube driver

Run:

```bash
minikube profile list
```

Look at the **Driver** column.

---

## Step 3 – Configure domain mapping (IMPORTANT)

### If you are using Docker driver (most macOS & Windows users)

You MUST:

1. Start tunnel:

```bash
minikube tunnel
```

(leave this running)

2. Edit hosts file:

```bash
sudo nano /etc/hosts
```

Add:

```text
127.0.0.1 dashboard.com
```

Why:
- Docker driver exposes services via localhost forwarding
- Kubernetes ingress will NOT show 127.0.0.1 in `kubectl get ingress`
- Local tunnel handles routing

---

### If you are using VM drivers (Linux, VirtualBox, HyperKit)

1. Get Minikube IP:

```bash
minikube ip
```

2. Edit hosts file:

```bash
sudo nano /etc/hosts
```

Add:

```text
<MINIKUBE_IP> dashboard.com
```

Example:

```text
192.168.49.2 dashboard.com
```

Tunnel is NOT required in this case.

---

## Step 4 – Access the Dashboard

Open browser:

```
http://dashboard.com
```

---

## Validation Checks

### Ingress exists

```bash
kubectl get ingress -n kubernetes-dashboard
```

### Ingress controller running

```bash
kubectl get pods -n ingress-nginx
```

### Ingress controller service

```bash
kubectl get svc -n ingress-nginx
```

If using Docker driver, you should see:

```
EXTERNAL-IP   127.0.0.1
```

---

## Common Pitfalls

| Problem | Cause | Fix |
|--------|--------|------|
| dashboard.com not loading | No tunnel running (Docker driver) | Run `minikube tunnel` |
| Ingress shows no IP | Normal on Docker driver | Ignore |
| 404 error | Wrong ingress rules | Check backend service name |
| Connection refused | Wrong hosts entry | Use 127.0.0.1 |
| Service pending | Tunnel not running | Start tunnel |

---

## Important Notes

- `kubectl get ingress` will NOT show `127.0.0.1`
- 127.0.0.1 is provided by Minikube tunnel (host networking)
- Kubernetes itself never assigns localhost addresses
- `/etc/hosts` change is local-only and safe

---
---

## Cleanup (Remove Everything After Testing)

When you are done testing, you can clean up safely using the steps below.

### 1. Delete the Ingress resource

```bash
kubectl delete -f dashboard-ingress.yaml
```

Or if you created it manually:

```bash
kubectl delete ingress dashboard-ingress -n kubernetes-dashboard
```

---

### 2. Stop the Minikube tunnel (Docker driver users)

In the terminal where `minikube tunnel` is running:

Press:

```
CTRL + C
```

---

### 3. Remove hosts file entry

Edit your hosts file:

```bash
sudo nano /etc/hosts
```

Remove this line:

```text
127.0.0.1 dashboard.com
```

(or the Minikube IP if you used a VM driver)

Save and exit.

---

### 4. (Optional) Disable addons

```bash
minikube addons disable ingress
minikube addons disable dashboard
```

---

### 5. (Optional) Stop Minikube completely

```bash
minikube stop
```

Or delete the cluster:

```bash
minikube delete
```

---

Cleanup complete ✅
---

Happy clustering 🚀
