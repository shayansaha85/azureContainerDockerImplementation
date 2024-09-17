
# Hosting a Python Script in an Azure Container

## Prerequisites

- **Azure Account**: Ensure you have an active Azure subscription.
- **Docker Installed**: You need Docker to create and manage the container.
- **Azure CLI**: Make sure the Azure CLI is installed on your local machine.
- **Azure Container Registry (optional)**: To store your Docker image.

---

## Steps to Host `main.py` in an Azure Container

### Step 1: Create a Dockerfile

In your project directory (where `main.py` is located), create a `Dockerfile` to package your Python app into a container.

```Dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY . /app
RUN pip install --no-cache-dir -r requirements.txt
EXPOSE 80
CMD ["python", "main.py"]
```

    Note: If your script doesn’t require dependencies, you can skip the RUN pip install part. If you have dependencies, add them to a requirements.txt file.

### Step 2: Build the Docker Image

Once the Dockerfile is ready, you can build the Docker image by running the following command:

```
docker build -t my-python-app .
```

### Step 3: Test the Docker Image Locally

You can test your container locally to make sure everything works as expected:

```
docker run -d my-python-app
```

### Step 4: Push the Docker Image to Azure Container Registry (Optional)

If you choose to store your Docker image in Azure Container Registry, follow these steps:

#### Login to Azure:

```
az login
```

#### Create Azure Container Registry:

```
az acr create --resource-group <resource-group-name> --name <registry-name> --sku Basic
```

#### Login to the Azure Container Registry:

```
az acr login --name <registry-name>
```

#### Tag the Docker Image:

```
docker tag my-python-app <registry-name>.azurecr.io/my-python-app:v1
```

#### Push the Docker Image:


```
docker push <registry-name>.azurecr.io/my-python-app:v1
```

#### Step 5: Create an Azure Container Instance (ACI)

Now, you can create an Azure Container Instance (ACI) to host your container.
##### Option 1: If You Have an Image in Azure Container Registry

```
az container create \
  --resource-group <resource-group-name> \
  --name my-python-container \
  --image <registry-name>.azurecr.io/my-python-app:v1 \
  --cpu 1 \
  --memory 1 \
  --registry-login-server <registry-name>.azurecr.io \
  --registry-username <username> \
  --registry-password <password> \
  --restart-policy Always
```
##### Option 2: If You’re Using a Local Image

```
az container create \
  --resource-group <resource-group-name> \
  --name my-python-container \
  --image my-python-app \
  --cpu 1 \
  --memory 1 \
  --restart-policy Always
```

#### Step 6: Schedule the Python Script to Run Every 30 Minutes

You have two options to schedule your script to run every 30 minutes:
##### Option 1: Modify the Python Code (Using APScheduler)

If you are already using APScheduler, you can modify your main.py script to run every 30 minutes.

```
from apscheduler.schedulers.blocking import BlockingScheduler

def job():
    print("Running task...")
    # Your task logic here

scheduler = BlockingScheduler()
scheduler.add_job(job, 'interval', minutes=30)
scheduler.start()
```
##### Option 2: Use Azure Logic Apps (External Trigger)

You can set up Azure Logic Apps to trigger the container every 30 minutes.

    Create a Logic App that runs every 30 minutes.
    Configure the Logic App to call the Azure Container Instance’s REST API to start the container.

#### Step 7: Monitor the Azure Container Instance

You can monitor the logs and the status of your Azure Container Instance using the Azure CLI:

```
az container logs --resource-group <resource-group-name> --name my-python-container
```
