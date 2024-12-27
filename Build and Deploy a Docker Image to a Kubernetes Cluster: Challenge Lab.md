# Challenge scenario
Your development team is interested in adopting a containerized microservices approach to application architecture. You need to test a sample application they have provided for you to make sure that it can be deployed to a Google Kubernetes container. The development group provided a simple Go application called echo-web with a Dockerfile and the associated context that allows you to build a Docker image immediately.

# Your challenge
To test the deployment, you need to download the sample application, then build the Docker container image using a tag that allows it to be stored on the Container Registry. Once the image has been built, you'll push it out to the Container Registry before you can deploy it.

With the image prepared you can then create a Kubernetes cluster, then deploy the sample application to the cluster.

Note: In order to ensure accurate lab activity tracking you must use echo-app as the container repository image name, call your Kubernetes cluster echo-cluster, create the Kubernetes cluster in ZONE zone and use echo-web for the deployment name.

```
gcloud auth list
gcloud config list project

export ZONE=zone
echo ${ZONE}

REGION="${ZONE%-*}"
echo ${REGION}

```

## Task 1. Create a Kubernetes cluster
Your test environment is limited in capacity, so you should limit the test Kubernetes cluster you are creating to just two e2-standard-2 instances. You must call your cluster echo-cluster.

```
gcloud container clusters create $CLUSTER_NAME \
  --num-nodes=2 \
  --machine-type=e2-standard-2 \
  --zone=$ZONE
```

## Task 2. Build a tagged Docker image
The sample application, including the Dockerfile and the application context files, are contained in an archive called echo-web.tar.gz. The archive has been copied to a Cloud Storage bucket belonging to your lab project called gs://[PROJECT_ID].

You must deploy this with a tag called v1.

```
kubectl create deployment echo-web --image=gs://$[PROJECT_ID]/echo-web.tar.gz:v1
```

## Task 3. Push the image to the Google Container Registry
Your organization has decided that it will always use the gcr.io Container Registry hostname for all projects. The sample application is a simple web application that reports some data describing the configuration of the system where the application is running. It is configured to use TCP port 8000 by default.

```
export PROJECT_ID=$(gcloud info --format='value(config.project)')
gsutil cp gs://${PROJECT_ID}/echo-web.tar.gz .
tar -xvzf echo-web.tar.gz
```
```
docker build -t echo-app:v1 .
docker tag echo-app:v1 gcr.io/${PROJECT_ID}/echo-app:v1
docker push gcr.io/${PROJECT_ID}/echo-app:v1
```

## Task 4. Deploy the application to the Kubernetes cluster
- Even though the application is configured to respond to HTTP requests on port 8000, you must configure the service to respond to normal web requests on port 80. When configuring the cluster for your sample application, call your deployment echo-web.
```
gcloud container clusters get-credentials echo-cluster
kubectl run echo-app --image=gcr.io/${PROJECT_ID}/echo-app:v1 --port 8000
```
```
kubectl expose deployment echo-app --name echo-web \
   --type LoadBalancer --port 80 --target-port 8000
   ```
```
kubectl get service echo-web
```

Troubleshooting
Receiving a 504, Gateway timeout error: This might just indicate that the application hasn't quite initialized yet, but it could also be caused by a mismatch between the default port that is set in the Dockerfile (TCP port 8000) and the choice of application port you configured when deploying the application image, or when you configured external access.

Not receiving assessment score for the last three objectives: This might just indicate that you have created your Kubernetes cluster in the different zone rather than ZONE zone which is expected in the lab.
