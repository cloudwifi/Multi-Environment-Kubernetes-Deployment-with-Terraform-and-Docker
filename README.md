# **Project: Multi-Environment Kubernetes Deployment with Terraform and Docker**

This project demonstrates how to deploy a **Python-based Flask application** in a **Kubernetes (EKS)** cluster using **Terraform** for infrastructure as code and **Docker** for containerization. The infrastructure is created for three environments: **Dev**, **Stage**, and **Prod**, using **Terraform workspaces**. The application is deployed in Kubernetes and accessed via a **LoadBalancer**.

---

    Prerequisites

    Steps to Deploy the Application

        Step 1: Set Up Terraform Workspaces

        Step 2: Create Infrastructure Using Terraform

        Step 3: Build and Push Docker Image

        Step 4: Deploy Application in Kubernetes

        Step 5: Access the Application

    Folder Structure

    Terraform Code Overview

    Kubernetes Manifests

    Conclusion
---

## **Prerequisites**
Before starting, ensure you have the following installed:
- **Terraform**: [Install Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli)
- **AWS CLI**: [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- **kubectl**: [Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- **Docker**: [Install Docker](https://docs.docker.com/get-docker/)
- **Helm** (optional, for Prometheus/Grafana): [Install Helm](https://helm.sh/docs/intro/install/)

---

## **Steps to Deploy the Application**

### **Step 1: Set Up Terraform Workspaces**
Terraform workspaces allow you to manage multiple environments (Dev, Stage, Prod) using the same codebase.

1. **Initialize Terraform**:
   ```bash
   terraform init
   ```

2. **Create Workspaces**:
   ```bash
   terraform workspace new dev
   terraform workspace new stage
   terraform workspace new prod
   ```

3. **Switch to the Desired Workspace**:
   ```bash
   terraform workspace select dev
   ```

---

### **Step 2: Create Infrastructure Using Terraform**
The Terraform code creates a **VPC**, **EKS cluster**, and other resources for each environment.

1. **Review Terraform Code**:
   - `main.tf`: Defines the infrastructure (VPC, EKS, etc.).
   - `variables.tf`: Contains environment-specific variables.
   - `outputs.tf`: Outputs important details (e.g., cluster name, VPC ID).

2. **Apply Terraform Configuration**:
   ```bash
   terraform plan
   terraform apply
   ```

   This will create:
   - A **VPC** with public and private subnets.
   - An **EKS cluster** with worker nodes.
   - Other necessary resources (e.g., IAM roles, security groups).

---

### **Step 3: Build and Push Docker Image**
Containerize the Flask app and push it to Docker Hub.

1. **Create a Dockerfile**:
   ```dockerfile
   FROM python:3.9-slim
   WORKDIR /app
   COPY . .
   RUN pip install -r requirements.txt
   EXPOSE 5000
   CMD ["flask", "run", "--host=0.0.0.0"]
   ```

2. **Build the Docker Image**:
   ```bash
   docker build -t your-dockerhub-username/flask-app:latest .
   ```

3. **Push the Docker Image**:
   ```bash
   docker push your-dockerhub-username/flask-app:latest
   ```

---

### **Step 4: Deploy Application in Kubernetes**
Deploy the Flask app to the EKS cluster using Kubernetes manifests.

1. **Create Kubernetes Deployment** (`deployment.yaml`):
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: flask-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: flask-app
     template:
       metadata:
         labels:
           app: flask-app
       spec:
         containers:
         - name: flask-app
           image: your-dockerhub-username/flask-app:latest
           ports:
           - containerPort: 5000
   ```

2. **Create Kubernetes Service** (`service.yaml`):
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: flask-app-service
   spec:
     type: LoadBalancer
     selector:
       app: flask-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 5000
   ```

3. **Apply the Manifests**:
   ```bash
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   ```

---

### **Step 5: Access the Application**
The Flask app is exposed via a **LoadBalancer**.

1. **Get the LoadBalancer URL**:
   ```bash
   kubectl get services
   ```

   Look for the `EXTERNAL-IP` of the `flask-app-service`.

2. **Access the Application**:
   Open a browser and navigate to:
   ```
   http://<EXTERNAL-IP>
   ```

   You should see the message: **"Hello, Kubernetes!"**.

---

## **Folder Structure**
```
terraform/
├── dev/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
├── stage/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
├── prod/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
├── modules/
│   ├── vpc/
│   ├── eks/
│   ├── ec2/
k8s/
├── deployment.yaml
├── service.yaml
Dockerfile
requirements.txt
app.py
```

---

## **Terraform Code Overview**
- **VPC Module**: Creates a VPC with public and private subnets.
- **EKS Module**: Creates an EKS cluster and worker nodes.
- **EC2 Module**: Creates EC2 instances for Dev and Stage environments.

---

## **Kubernetes Manifests**
- **Deployment**: Manages the Flask app pods.
- **Service**: Exposes the Flask app via a LoadBalancer.

---

## **Conclusion**
This project demonstrates how to:
- Use **Terraform** to create infrastructure for multiple environments.
- Containerize a Flask app using **Docker**.
- Deploy the app to **Kubernetes (EKS)**.
- Access the app via a **LoadBalancer**.

By following these steps, you can easily deploy and manage applications in a multi-environment setup using modern DevOps tools.

---

Let me know if you need further assistance! 🚀
