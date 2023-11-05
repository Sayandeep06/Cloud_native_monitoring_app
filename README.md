# **Cloud Native Resource Monitoring Python App on Kubernetes!**

**Steps**

1. Created a Monitoring Application in Python using Flask and psutil ![Screenshot from 2023-11-03 23-29-18](https://github.com/Sayandeep06/Cloud_native_monitoring_app/assets/100061797/12c8736e-a425-421e-a4e6-86e5a96b2e2f)
 
2. Run the Python App locally. ![Screenshot from 2023-11-04 00-29-31](https://github.com/Sayandeep06/Cloud_native_monitoring_app/assets/100061797/3eb020e4-eace-475f-918d-d28ba704b4dc)

3. Containerize a Python application using Docker
    1. Creating Dockerfile ![Screenshot from 2023-11-04 00-55-36](https://github.com/Sayandeep06/Cloud_native_monitoring_app/assets/100061797/ad47c339-db60-4e5a-bfdf-24d25a2d575d)

    2. Building DockerImage ![Screenshot from 2023-11-04 00-55-36](https://github.com/Sayandeep06/Cloud_native_monitoring_app/assets/100061797/d78bc415-780d-4418-8439-745805d734af)

    3. Running Docker Container 
    4. Docker Commands
4. Create ECR repository using Python Boto3 and pushing Docker Image to ECR ![Screenshot from 2023-11-04 01-33-23](https://github.com/Sayandeep06/Cloud_native_monitoring_app/assets/100061797/1a69390f-2209-432e-b118-6774412c137f)
![Screenshot from 2023-11-04 01-34-36](https://github.com/Sayandeep06/Cloud_native_monitoring_app/assets/100061797/ca739f36-f127-4ec0-90f4-d68227605281)

5. Create EKS cluster and Nodegroups  ![Screenshot from 2023-11-05 01-57-45](https://github.com/Sayandeep06/Cloud_native_monitoring_app/assets/100061797/4174b228-bb37-4a19-8546-d8074a59cce9)

6. Create Kubernetes Deployments and Services using Python!


## **Prerequisites** !


- [x]  AWS Account.
- [x]  Programmatic access and AWS configured with CLI.
- [x]  Python3 Installed.
- [x]  Docker and Kubectl installed.
- [x]  Code editor (Vscode)

# Start the Project 

## **Part 1: Deploying the Flask application locally**

### **Step 1: Clone the code**

Clone the code from the repository:

```
git clone <repository_url>
```

### **Step 2: Install dependencies**

The application uses the **`psutil`** and **`Flask`, Plotly, boto3** libraries. Install them using pip:

```
pip3 install -r requirements.txt
```

### **Step 3: Run the application**

To run the application, navigate to the root directory of the project and execute the following command:

```
python3 app.py
```

This will start the Flask server on **`localhost:5000`**. Navigate to [http://localhost:5000/](http://localhost:5000/) on your browser to access the application.

## **Part 2: Dockerizing the Flask application**

### **Step 1: Create a Dockerfile**

Create a **`Dockerfile`** in the root directory of the project with the following contents:

```
# Use the official Python image as the base image
FROM python:3.9-slim-buster

# Set the working directory in the container
WORKDIR /app

# Copy the requirements file to the working directory
COPY requirements.txt .

RUN pip3 install --no-cache-dir -r requirements.txt

# Copy the application code to the working directory
COPY . .

# Set the environment variables for the Flask app
ENV FLASK_RUN_HOST=0.0.0.0

# Expose the port on which the Flask app will run
EXPOSE 5000

# Start the Flask app when the container is run
CMD ["flask", "run"]
```

### **Step 2: Build the Docker image**

To build the Docker image, execute the following command:

```
docker build -t <image_name> .
```

### **Step 3: Run the Docker container**

To run the Docker container, execute the following command:

```
docker run -p 5000:5000 <image_name>
```

This will start the Flask server in a Docker container on **`localhost:5000`**. Navigate to [http://localhost:5000/](http://localhost:5000/) on your browser to access the application.

## **Part 3: Pushing the Docker image to ECR**

### **Step 1: Create an ECR repository**

Create an ECR repository using Python:

```
import boto3

# Create an ECR client
ecr_client = boto3.client('ecr')

# Create a new ECR repository
repository_name = 'my-ecr-repo'
response = ecr_client.create_repository(repositoryName=repository_name)

# Print the repository URI
repository_uri = response['repository']['repositoryUri']
print(repository_uri)
```

### **Step 2: Push the Docker image to ECR**

Push the Docker image to ECR using the push commands on the console:

```
docker push <ecr_repo_uri>:<tag>
```

## **Part 4: Creating an EKS cluster and deploying the app using Python**

### **Step 1: Create an EKS cluster**

Create an EKS cluster and add node group

### **Step 2: Create a node group**

Create a node group in the EKS cluster.

### **Step 3: Create deployment and service**

```jsx
from kubernetes import client, config

# Load Kubernetes configuration
config.load_kube_config()

# Create a Kubernetes API client
api_client = client.ApiClient()

# Define the deployment
deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name="my-flask-app"),
    spec=client.V1DeploymentSpec(
        replicas=1,
        selector=client.V1LabelSelector(
            match_labels={"app": "my-flask-app"}
        ),
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(
                labels={"app": "my-flask-app"}
            ),
            spec=client.V1PodSpec(
                containers=[
                    client.V1Container(
                        name="my-flask-container",
                        image="568373317874.dkr.ecr.us-east-1.amazonaws.com/my-cloud-native-repo:latest",
                        ports=[client.V1ContainerPort(container_port=5000)]
                    )
                ]
            )
        )
    )
)

# Create the deployment
api_instance = client.AppsV1Api(api_client)
api_instance.create_namespaced_deployment(
    namespace="default",
    body=deployment
)

# Define the service
service = client.V1Service(
    metadata=client.V1ObjectMeta(name="my-flask-service"),
    spec=client.V1ServiceSpec(
        selector={"app": "my-flask-app"},
        ports=[client.V1ServicePort(port=5000)]
    )
)

# Create the service
api_instance = client.CoreV1Api(api_client)
api_instance.create_namespaced_service(
    namespace="default",
    body=service
)
```

make sure to edit the name of the image on line 25 with your image Uri.

- Once you run this file by running “python3 eks.py” deployment and service will be created.
- Check by running following commands:

```jsx
kubectl get deployment -n default (check deployments)
kubectl get service -n default (check service)
kubectl get pods -n default (to check the pods)
```

Once your pod is up and running, run the port-forward to expose the service

```bash
kubectl port-forward service/<service_name> 5000:5000
```
![Screenshot from 2023-11-04 01-16-48](https://github.com/Sayandeep06/Cloud_native_monitoring_app/assets/100061797/499fd77d-9e8c-48f3-9769-a22d000c9567)
