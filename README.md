# Example microservice application with DAPR and Kubernetes using Rancher K3D

## Technologies

This section describes the used technologies for this project.

### Services

| Service          	| Technology            	|
|------------------	|-----------------------	|
| AnalyticsService 	| Golang w/ Gorilla Mux 	|
| PostService      	| ASP.NET Core 6        	|

### Deployment

All of the listed services are being dockerized with their respective Dockerfiles and then deployed in a local Rancher K3D Kubernetes cluster. By using the `annotations`-section of the deployment manifests, Dapr then injects a sidecar container into each pod of the deployments, that handles all microservice-relevant communication (pubsub, state, etc.). For pubsub-communication, an instance of Redis is being used in development, which can easily be switched out by editing the `dapr-redis.yaml`-manifest.

## Getting Started

Create a K3D Cluster

```sh
k3d registry create registry.localhost --port 5000
k3d cluster create -c k3d.yaml --registry-use k3d-registry.localhost:5000
```

Install Dapr

```sh
# Install Dapr CLI
wget -q https://raw.githubusercontent.com/dapr/cli/master/install/install.sh -O - | /bin/bash
# Initialize in K8s
dapr init -k
```

Add the local registry domain name to your local hosts file.

```sh
# /etc/hosts
127.0.0.1       k3d-registry.localhost
```

Now build the necessary docker images, tag them and deploy to the local registry.

```sh
# Build
docker build -t posts-service ./PostsService
docker build -t analytics-service ./AnalyticsService

# Tag
docker tag posts-service k3d-registry.localhost:5000/posts-service
docker tag analytics-service k3d-registry.localhost:5000/analytics-service

# Push
docker push k3d-registry.localhost:5000/posts-service
docker push k3d-registry.localhost:5000/analytics-service
```

Now run the application by creating all resources in K8s.

```sh
kubectl apply -f ./k8s
```

## Cleanup

To clean up all resources, use the following commands.

```sh
k3d cluster delete dapper-cluster
k3d registry delete registry.localhost
```