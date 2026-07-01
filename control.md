# Dokumentasi Setup Cluster K3s

## Control Plane / Admin

### 1. Install K3s Server

```bash
curl -sfL https://get.k3s.io | sh -
```

Menginstal K3s sebagai **control plane** yang bertugas mengelola seluruh cluster Kubernetes.

---

### 2. Menampilkan Node Token

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

Menampilkan token yang digunakan worker node untuk bergabung ke cluster.

---

### 3. Memastikan Worker Berhasil Join

```bash
kubectl get nodes
```

Menampilkan seluruh node yang terdaftar pada cluster. Apabila worker berhasil bergabung, statusnya akan **Ready**.

Contoh output:

```
NAME        STATUS   ROLES           VERSION
backlink    Ready    control-plane   v1.36.x
archlinux   Ready    <none>          v1.36.x
```

---

### 4. Install kubectl

```bash
pacman -S kubectl
```

Menginstal utilitas **kubectl** untuk mengelola resource Kubernetes.

---

### 5. Membuat Manifest Deployment

```bash
nvim slims-multi-node.yaml
```

Membuat file manifest Kubernetes yang berisi konfigurasi deployment dan service aplikasi SLiMS beserta database.

---

### 6. Deploy Manifest

```bash
kubectl apply -f slims-multi-node.yaml
```

Menerapkan seluruh resource yang terdapat pada file manifest ke dalam cluster Kubernetes.

---

### 7. Verifikasi Pod

```bash
kubectl get pods -o wide
```

Menampilkan status Pod beserta node tempat Pod dijalankan untuk memastikan deployment berhasil.

Contoh hasil:

```
slims-db-pod                 Running
slims-web-deployment-xxxxx   Running
slims-web-deployment-yyyyy   Running
```

Output juga menunjukkan salah satu Pod berjalan di node **archlinux**, yang menandakan worker telah berhasil digunakan oleh cluster.
