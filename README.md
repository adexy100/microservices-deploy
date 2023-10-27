# Containerized deployment with AKS

## Project Summary 
Implementing a project for containerized deployment with Azure Kubernetes Service (AKS) for a Django application that can automatically scale based on real-time resource usage 

### Step 1: Understand the Project Elements

- Ensure you have a working Django application that you want to containerize and deploy. Let's call this application "MyDjangoApp."
- Azure Subscription: You need access to a Microsoft Azure subscription.

### Step 2: Create the Azure Container Registry

1. **Login to Azure**: Use the Azure CLI to log in to your Azure account.
   ```shell
   az login
   ```

2. **Create a Resource Group**: You can create a new resource group or use an existing one.
   ```shell
   az group create --name MyResourceGroup --location EastUS
   ```

3. **Create an Azure Container Registry (ACR)**:
   ```shell
   az acr create --resource-group MyResourceGroup --name myacrname --sku Basic
   ```

### Step 3: Build an Image and Run a Container before Push to Azure Container Registry

1. **Dockerize Your Django App**:

   Create a `Dockerfile` in your Django project directory:

   ```Dockerfile
   FROM python:3.8

   WORKDIR /app

   COPY requirements.txt requirements.txt

   RUN pip install -r requirements.txt

   COPY . .

   CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
   ```

2. **Build the Docker Image**:
   ```shell
   docker build -t myacrname.azurecr.io/mydjangoapp:v1 .
   ```

3. **Run the Docker Container Locally**:
   ```shell
   docker run -d -p 8000:8000 myacrname.azurecr.io/mydjangoapp:v1
   ```

### Step 4: Create an Azure Kubernetes Cluster

1. **Create AKS Cluster**:
   ```shell
   az aks create --resource-group MyResourceGroup --name myakscluster --node-count 1 --enable-addons monitoring --generate-ssh-keys
   ```

2. **Configure kubectl** to use the AKS cluster:
   ```shell
   az aks get-credentials --resource-group MyResourceGroup --name myakscluster
   ```

### Step 5: Deploy an Application in Azure Kubernetes Cluster

1. **Create Kubernetes Deployment YAML**:

   Create a file named `deployment.yaml`:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: mydjangoapp
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: mydjangoapp
     template:
       metadata:
         labels:
           app: mydjangoapp
       spec:
         containers:
         - name: mydjangoapp
           image: myacrname.azurecr.io/mydjangoapp:v1
           ports:
           - containerPort: 8000
   ```

2. **Deploy to AKS**:
   ```shell
   kubectl apply -f deployment.yaml
   ```

### Step 6: Scale Out the Azure Kubernetes Cluster

1. **Auto Scaling**:

   Update your `deployment.yaml` to include resource requests and limits for CPU and memory. For example:

   ```yaml
   resources:
     requests:
       memory: "64Mi"
       cpu: "250m"
     limits:
       memory: "128Mi"
       cpu: "500m"
   ```

### Step 7: Monitoring Azure Kubernetes Cluster

1. **Azure Monitor**:

   Azure Monitor collects telemetry data, including performance and resource usage metrics, automatically. You can access this data through Azure Portal.

2. **Kubernetes Dashboard**:

   Install the Kubernetes Dashboard for monitoring your cluster:
   ```shell
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
   ```

   Access the dashboard using `kubectl proxy` and a token.

### Step 8: Clean Up the Project Environment

1. **Delete Kubernetes Resources**:
   ```shell
   kubectl delete -f deployment.yaml
   ```

2. **Delete the AKS Cluster**:
   ```shell
   az aks delete --resource-group MyResourceGroup --name myakscluster --yes --no-wait
   ```

3. **Delete the Azure Container Registry**:
   ```shell
   az acr delete --name myacrname --resource-group MyResourceGroup --yes --no-wait
   ```

This detailed walkthrough includes examples for creating an Azure Container Registry, building a Docker image, deploying to an Azure Kubernetes Cluster, and cleaning up the environment. Make sure to replace <placeholders> with your actual resource names and configurations.
  
