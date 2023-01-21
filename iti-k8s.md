### Volumes
#### emptyDir
- Used as temporary storage

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
        - name: data 
          emptyDir: {}

      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: data 
          mountPath: /data
```

#### hostPath
- Relies on storing data on the kubernetes node
- Data isn't replicated, it's only available on that single node

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
        - name: kube-data  
          hostPath:
            path: /tmp/kubernetes

      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: kube-data 
          mountPath: /kube-data
```

---

### NFS Server Setup

1. NFS Server - Virtual Machine (EC2) => nfs-server
2. Worker Node - nfs-client

```bash
sudo apt install nfs-kernel-server -y

# Create main directory
sudo mkdir -pv /mnt/nfs

# Set directory permissions
sudo chown -R nobody:nogroup /mnt/nfs
sudo chmod 777 /mnt/nfs

# Modify exports file to allow any client to access the server mount 
echo '/mnt/nfs *(rw,no_root_squash,no_subtree_check)' | sudo tee -a /etc/exports

# Export NFS shared directory
sudo exportfs -a

# Restart nfs server
sudo systemctl restart nfs-kernel-server
```

### NFS Client Setup
```bash
sudo apt install nfs-common -y
```

---

### Create PV Via NFS
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs 
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /mnt/nfs/nginx
    server: 192.168.100.12
```

### Create PVC to bind to the PV
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs
  namespace: nginx
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: nfs
  volumeName: pv-nfs
  resources:
    requests:
      storage: 1Gi
```

### Create Nginx Deployment with PVC used as mount
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: nginx-volume
        persistentVolumeClaim:
          claimName: pvc-nfs
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-volume
          mountPath: /usr/share/nginx/html
```

---

### Dynamic Provisioning
- PVCs result in a PVC being created

---

### Probes
#### Startup Probes

#### Readiness Probes

#### Liveness Probes