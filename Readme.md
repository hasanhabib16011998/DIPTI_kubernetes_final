# Task 1
---

### **Step 1: Create the `index.html` File**
Create a simple `index.html` file to serve via the Nginx server.

**File: `index.html`**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome</title>
</head>
<body>
    <h1>Welcome to My Nginx Server</h1>
    <p>This page is being served by an Nginx container.</p>
</body>
</html>
```

---

### **Step 2: Create the `Dockerfile`**
This `Dockerfile` uses the Nginx base image and serves the `index.html` file by copying it into the Nginx HTML directory.

**File: `Dockerfile`**
```dockerfile
# Use the Nginx base image
FROM nginx:latest

# Copy the index.html file to the default Nginx HTML directory
COPY index.html /usr/share/nginx/html/

# Expose port 80 for web traffic
EXPOSE 80
```

---

### **Step 3: Build and Run the Docker Image**
1. Save the `index.html` and `Dockerfile` in the same directory.
2. Open a terminal, navigate to this directory, and build the Docker image:
   ```bash
   docker build -t my-nginx-server .
   ```
3. Run the Docker container:
   ```bash
   docker run -d -p 8080:80 my-nginx-server
   ```
4. Access the webpage by opening a browser and visiting:
   ```
   http://localhost:8080
   ```

We can see the content from the `index.html` file served by the Nginx container.

### **Step 4: Push the docker image into DockerHub**

```bash
# Step 1: Build the Docker image
docker build -t my-nginx-server .

# Step 2: Tag the image for Docker Hub
docker tag my-nginx-server hasanhabib16011998/dipti_kubernetes_final:latest

# Step 3: Log in to Docker Hub
docker login

# Step 4: Push the image to Docker Hub
docker push hasanhabib16011998/dipti_kubernetes_final:latest
```

To deploy your container to a Kubernetes cluster using a NodePort service, follow these steps:

### **Step 5: Create a Kubernetes Deployment**
The Deployment will ensure your container is running and can manage updates. Save the following YAML as `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dipti-kubernetes-final
  labels:
    app: dipti-kubernetes-final
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dipti-kubernetes-final
  template:
    metadata:
      labels:
        app: dipti-kubernetes-final
    spec:
      containers:
      - name: dipti-container
        image: hasanhabib16011998/dipti_kubernetes_final:latest
        ports:
        - containerPort: 80
```

### **Step 6: Create a NodePort Service**
The NodePort service will expose your container to a port on the cluster nodes. Save this YAML as `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: dipti-kubernetes-final-service
spec:
  type: NodePort
  selector:
    app: dipti-kubernetes-final
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30007 # Custom port
```

### **Step 7: Apply the YAML Files**
Run the following commands to deploy the container:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### **Step 8: Access the Application**
Access the application using the node's IP and the NodePort:

   ```
   http://<node-ip>:30007
   ```


# Task 2
---


# Setting up NFS Server and Kubernetes NFS Provisioner

### **Step 1: Prepare the NFS Server**

#### Install NFS Server
On the host machine, install the NFS server:
```bash
sudo apt update
sudo apt install nfs-kernel-server -y
```

#### Create a Directory for NFS
Create a directory where the NFS server will serve the files:
```bash
sudo mkdir -p /var/k8-nfs/data
sudo chown -R nobody:nogroup /var/k8-nfs/data
sudo chmod 2770 /var/k8-nfs/data
```

#### Configure NFS Exports
Add the export options to the NFS configuration:
```bash
sudo vi /etc/exports
```
Add the following line:
```
/var/k8-nfs/data 192.168.0.0/24(rw,sync,no_subtree_check,no_root_squash,no_all_squash)
```

#### Apply NFS Export Configuration
Make the specified directories available for NFS clients to access and restart the NFS service:
```bash
sudo exportfs -avr
sudo systemctl restart nfs-kernel-server
sudo systemctl status nfs-kernel-server
```

### **Step 2: Install NFS Common on Worker and Master Nodes**
Install the `nfs-common` package on all worker and master nodes:
```bash
sudo apt install nfs-common -y
```

### **Step 3: Install Helm**
Helm is the best way to find, share, and use software built for Kubernetes.

#### Install Helm
Follow the official Helm installation instructions: [Helm Installation Guide](https://helm.sh/docs/intro/install/)

#### Add NFS Provisioner Helm Repository
After installing Helm on the master node, add the NFS provisioner repository:
```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
```

#### Install NFS Subdir External Provisioner
Install the NFS provisioner using Helm:
```bash
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --set nfs.server=192.168.0.169 \
  --set nfs.path=/var/k8-nfs/data
```

### **Step 4: Create a Persistent Volume Claim**
Use the following YAML to create a PersistentVolumeClaim:
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: demo-claim
  # namespace: nfs-provisioning
spec:
  storageClassName: nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```

Apply the YAML using `kubectl apply -f`:
```bash
kubectl apply -f demo-claim.yaml
```

### **Step 5: Deploy Two Containers Sharing the NFS Volume**

#### Deployment YAML
Add the following deployment file to deploy two containers sharing the same NFS volume:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: task-2
  labels:
    app: dual-alpine
spec:
  replicas: 1
  selector:
    matchLabels:
      app: dual-alpine
  template:
    metadata:
      labels:
        app: dual-alpine
    spec:
      containers:
      - name: container-1
        image: alpine:latest
        command: ["sh", "-c", "i=1; while true; do echo \"Container-1 $(date)\" > /mount/log$i.txt; i=$((i+1)); sleep 60; done"]
        volumeMounts:
        - name: shared-nfs-volume
          mountPath: /mount
      - name: container-2
        image: alpine:latest
        command: ["sh", "-c", "i=1; while true; do echo \"Container-2 $(date)\" > /mount/file$i.txt; i=$((i+1)); sleep 60; done"]
        volumeMounts:
        - name: shared-nfs-volume
          mountPath: /mount
      volumes:
      - name: shared-nfs-volume
        persistentVolumeClaim:
          claimName: nfs-pvc

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
  labels:
    app: dual-alpine
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs-client
```

#### Breakdown of What Happens
1. **Deployment Name**: The deployment is named `task-2` and deploys a single pod with two containers.
2. **Container-1**: Writes logs (`log1.txt`, `log2.txt`, etc.) to the `/mount` directory every minute. The logs include the current date and time.
3. **Container-2**: Writes files (`file1.txt`, `file2.txt`, etc.) to the same `/mount` directory every minute. The files include the current date and time.
4. **Shared Volume**: Both containers share the same NFS-backed volume mounted at `/mount`, allowing them to write files to the same directory.
5. **NFS PVC**: The `nfs-pvc` PersistentVolumeClaim is used to connect the deployment to the NFS server, which enables ReadWriteMany (RWX) access across containers.

Apply the deployment using:
```bash
kubectl apply -f deployment.yaml
```