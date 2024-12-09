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

### **Step 4: Push the docker image into DockerHub

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