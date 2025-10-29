# 🧭 Rangkuman Belajar Kubernetes (K3s Single Node)

> Dari instalasi → Pod → Deployment → Service → Ingress  
> (disusun dari teori + praktik yang sudah dilakukan)

---

## 🧱 1️⃣ Instalasi K3s

**Perintah:**
```bash
curl -sfL https://get.k3s.io | sh -
```

- Menginstal semua komponen Kubernetes (API Server, Controller, Scheduler, Kubelet, Traefik, dsb).
- File konfigurasi cluster: `/etc/rancher/k3s/k3s.yaml`

**Cek apakah berhasil:**
```bash
sudo systemctl status k3s
kubectl get nodes
kubectl get pods -A
```

Jika status node `Ready`, berarti cluster aktif ✅

---

## 🌍 2️⃣ Node dan Namespace

### 🔹 Node
- Node = server tempat container dijalankan.  
- Dalam K3s single-node, 1 node = 1 master sekaligus worker.
```bash
kubectl get nodes -o wide
```

### 🔹 Namespace
Namespace = ruang logis untuk memisahkan resource.
```bash
kubectl get ns
```
| Namespace | Fungsi |
|------------|--------|
| `default` | tempat resource user |
| `kube-system` | komponen internal (traefik, coredns, dsb) |
| `kube-public` | data publik cluster |
| `kube-node-lease` | heartbeat status node |

---

## 🧩 3️⃣ Pod (Unit terkecil)

**Teori:**  
Pod = wadah yang menjalankan satu atau lebih container.  
Setiap Pod punya IP unik di dalam cluster.

**Praktik:**
```bash
kubectl run mynginx --image=nginx:alpine --port=80
kubectl get pods
kubectl logs mynginx
kubectl exec -it mynginx -- sh
```

> `--port=80` hanya metadata, tidak otomatis membuka akses ke luar.

---

## ⚙️ 4️⃣ Deployment

**Teori:**  
Deployment = pengatur ReplicaSet, memastikan jumlah Pod sesuai target (auto-healing, rolling update, rollback).

**Praktik:**
```bash
kubectl create deployment webapp --image=nginx:alpine
kubectl get deploy,rs,pods
```

Kalau Pod dihapus:
```bash
kubectl delete pod -l app=webapp
kubectl get pods -w
```

➡️ Kubernetes langsung membuat Pod baru otomatis.

---

## 📦 5️⃣ Service (Networking dasar)

**Teori:**  
Service memberi IP tetap dan load-balancing ke Pod.  
Tanpa Service, Pod tidak bisa diakses dari luar cluster.

| Jenis | Fungsi |
|--------|--------|
| ClusterIP | Hanya untuk komunikasi internal |
| NodePort | Membuka port di node host |
| LoadBalancer | Untuk cloud provider |

**Praktik:**
```bash
kubectl expose deployment webapp --type=NodePort --port=80
kubectl get svc
```

Output contoh:
```
webapp   NodePort   10.43.90.20   <none>   80:30729/TCP
```

Cek akses:
```bash
curl http://localhost:30729
# atau dari luar:
curl http://<IP_PUBLIC_VPS>:30729
```

> Rentang NodePort: 30000–32767, bisa diatur manual.

---

## 🔐 6️⃣ Firewall & Security Group

Aktifkan UFW dan buka port penting:
```bash
sudo ufw enable
sudo ufw allow 22/tcp       # SSH
sudo ufw allow 30729/tcp    # NodePort webapp
sudo ufw allow 80,443/tcp   # Untuk Ingress
sudo ufw status
```

Jangan lupa buka port sama di **Tencent Cloud Security Group**.

---

## ⚡ 7️⃣ Ingress & Traefik

**Teori:**  
Ingress = router HTTP di dalam cluster.  
Dikelola oleh Ingress Controller (K3s → Traefik).  
Mengatur routing berdasarkan *host* atau *path*.

**Cek Traefik aktif:**
```bash
kubectl get pods -n kube-system | grep traefik
```

**Praktik:**
Buat file `ingress-demo.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo-ingress
spec:
  ingressClassName: traefik
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp
            port:
              number: 80
      - path: /nginx
        pathType: Prefix
        backend:
          service:
            name: mynginx
            port:
              number: 80
```

Apply:
```bash
kubectl apply -f ingress-demo.yaml
kubectl get ingress -o wide
```

Cek akses:
```
http://<IP_PUBLIC_VPS>/        → webapp
http://<IP_PUBLIC_VPS>/nginx   → mynginx
```

---

## 🧠 8️⃣ Hubungan antar komponen

```
┌────────────┐     ┌─────────────┐     ┌────────────┐     ┌──────────┐
│  Ingress   │ ──> │   Service   │ ──> │   Pod(s)   │ ──> │ Container │
│ (Traefik)  │     │ (webapp)    │     │ (nginx)    │     │ (Image)  │
└────────────┘     └─────────────┘     └────────────┘     └──────────┘
```

---

## ✅ 9️⃣ Poin penting yang sudah dikuasai

✔ Instalasi dan verifikasi cluster K3s  
✔ Node dan Namespace  
✔ Pod (teori dan praktik)  
✔ Deployment dan auto-healing  
✔ Service (NodePort dan ClusterIP)  
✔ Firewall + Security Group  
✔ Ingress + Traefik untuk routing HTTP

---

## 🚀 🔟 Langkah Lanjut

1. **ConfigMap & Secret** → konfigurasi & environment variable  
2. **PersistentVolumeClaim (PVC)** → storage data yang persisten  
3. **Rolling update / rollback** → update image tanpa downtime  
4. **Monitoring & Metrics** → Prometheus + Grafana di k3s