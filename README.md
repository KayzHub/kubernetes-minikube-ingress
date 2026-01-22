# Testing Kubernetes Dashboard Ingress on Minikube

This guide outlines how to expose the Kubernetes Dashboard via an Ingress resource using the custom domain `dashboard.com`.

---

## Prerequisites

Ensure the following are in place before proceeding:

- Minikube installed and running  
  ```bash
  minikube start
  ```

- Minikube Ingress addon is enabled
  ```bash
  minikube addons enable ingress
  ```
- Minikube dashboard command is called
  ```bash
  minikube dashboard
  ```
- Administrative / sudo privileges (for editing the hosts file)

---

## Steps to Test

### 1. Apply the Ingress Manifest

Save your YAML content into a file named `dashboard-ingress.yaml` and apply it:

```bash
kubectl apply -f dashboard-ingress.yaml
```

---

### 2. Identify the Minikube IP

Retrieve the IP address where the Ingress controller is listening:

```bash
minikube ip
```

> **Note:**  
> On macOS or Windows (especially when using the Docker driver), the Ingress may bind to `127.0.0.1`.  
> Verify using:
>
> ```bash
> kubectl get ingress -n kubernetes-dashboard
> ```

---

### 3. Update Your Local Hosts File

You must map the custom domain to the Minikube IP.

#### Linux / macOS

```bash
sudo nano /etc/hosts
```

#### Windows (run as Administrator)

Edit:

```
C:\Windows\System32\drivers\etc\hosts
```

Add the following line (replace `<MINIKUBE_IP>`):

```text
<MINIKUBE_IP> dashboard.com
```

Example:

```text
192.168.49.2 dashboard.com
```

Save and exit.

---

### 4. Start the Minikube Tunnel (Required for some drivers)

If you are on macOS, Windows, or using the Docker driver, run:

```bash
minikube tunnel
```

Keep this terminal open while testing.

---

### 5. Access the Dashboard

Open your browser and navigate to:

```
http://dashboard.com
```

---

## Expected Outcomes

### Browser Access

The Kubernetes Dashboard interface should load successfully at:

```
http://dashboard.com
```

### Ingress Verification

```bash
kubectl get ingress -n kubernetes-dashboard
```

You should see an assigned IP address (not `<pending>`).

### Backend Routing

The Ingress controller should correctly route:

```
Port 80 → kubernetes-dashboard Service → Dashboard Pods
```

in the `kubernetes-dashboard` namespace.

---

## Troubleshooting Tips

- Ensure the ingress controller pods are running:
  ```bash
  kubectl get pods -n ingress-nginx
  ```
- Confirm the dashboard service exists:
  ```bash
  kubectl get svc -n kubernetes-dashboard
  ```
- Re-check your `/etc/hosts` entry.
- Make sure `minikube tunnel` is still running if required.

---

## Notes

- `dashboard.com` is a local development domain only.
- No DNS changes are required outside your machine.
- No permanent system changes occur besides the hosts file entry.

---

Happy clustering 🚀

