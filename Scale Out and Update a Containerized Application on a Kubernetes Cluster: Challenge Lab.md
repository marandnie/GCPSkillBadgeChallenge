# Challenge scenario
You are taking over ownership of a test environment and have been given an updated version of a containerized test application to deploy. Your systems' architecture team has started adopting a containerized microservice architecture. You are responsible for managing the containerized test web applications. You will first deploy the initial version of a test application, called echo-app to a Kubernetes cluster called echo-cluster in a deployment called echo-web. The cluster will be deployed in the ZONE zone.

1. Before you get started, in the Navigation menu, select Cloud Storage.

2. Verify the echo-web-v2.tar.gz file is in the bucket name bucket.

Next, you will check to make sure your GKE cluster has been created before continuing.

3. In the Navigation menu, select select Kuberntes Engine > Clusters.
Continue when you see a green checkmark next to echo-cluster:

4. To deploy your first version of the application, run the following commands in Cloud Shell to get up and running:
```
gcloud container clusters get-credentials echo-cluster --zone=ZONE
```
```
kubectl create deployment echo-web --image=gcr.io/qwiklabs-resources/echo-app:v1
```
```
kubectl expose deployment echo-web --type=LoadBalancer --port 80 --target-port 8000
```

Your challenge

You need to update the running echo-app application in the echo-web deployment from the v1 to the v2 code you have been provided. You must also scale out the application to 2 instances and confirm that they are all running.

```
gcloud auth list
gcloud config list project

export ZONE=zone
echo ${ZONE}

REGION="${ZONE%-*}"
echo ${REGION}

```

## Task 1. Build and deploy the updated application with a new tag
The updated sample application, including the Dockerfile and the application context files, are contained in an archive called echo-web-v2.tar.gz. The archive has been copied to a Cloud Storage bucket in your lab project called bucket name. V2 of the application adds a version number to the output of the application. In this task, you will download the archive, build the Docker image, and tag it with the v2 tag.

```
kubectl create deployment echo-web --image=gs://qwiklabs-gcp-00-97c2f1423a49:v2
```
```
kubectl expose deployment echo-web --type=LoadBalancer --port 80 --target-port 8000
```

## Task 2. Push the image to the Container Registry
Your organization uses the Container Registry to host Docker images for deployments, and uses the gcr.io Container Registry hostname for all projects. You must push the updated image to the Container Registry before deploying it.

Click Check my progress to verify the objective.
Check that there is a tagged image in gcr.io for echo-app:v2.


## Task 3. Deploy the updated application to the Kubernetes cluster
In this task, you will deploy the updated application to the Kubernetes cluster. The deployment should be named echo-web and the application should be exposed on port 80. The application should be accessible from outside the cluster.

Click Check my progress to verify the objective.
Deploy the updated application version (v2) to the Kubernetes cluster.


## Task 4. Scale out the application
In this task, you will need to scale out the application to 2 replicas.

Click Check my progress to verify the objective.
Scale out the kubernetes application so that it is running 2 replicas.


## Task 5. Confirm the application is running
In this task, you will need to confirm that the application is running and responding correctly. You can use the external IP address of the application to test it.

Click Check my progress to verify the objective.
Verify your deployed application service is responding correctly.

```
gsutil cp gs://sureskills-ql/challenge-labs/ch05-k8s-scale-and-update/echo-web-v2.tar.gz .

tar xvzf echo-web-v2.tar.gz

gcloud builds submit --tag gcr.io/$DEVSHELL_PROJECT_ID/echo-app:v2 .

gcloud container clusters get-credentials echo-cluster --zone=$ZONE

kubectl create deployment echo-web --image=gcr.io/qwiklabs-resources/echo-app:v1

kubectl expose deployment echo-web --type=LoadBalancer --port 80 --target-port 8000

kubectl edit deploy echo-web

Change image version 'v1' to 'v2' by pressing "i" Save by esc -> Esc - > ":wq" and press enter.

kubectl scale deploy echo-web --replicas=2
```

Troubleshooting
Receiving a 504, Gateway timeout error: This might just indicate that the application hasn't quite initialized yet, but it could also be caused by a mismatch between the default port that is set in the Dockerfile (TCP port 8000) and:

The choice of application port you configured when deploying the application image, or
When you configured external access.
