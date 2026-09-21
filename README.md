# Udagram Microservices Project

Udagram is a cloud-based image sharing application developed as part of the Udacity Cloud Developer Nanodegree.

This project refactors the original monolithic Udagram backend into a microservices architecture. The application components are containerized using Docker and configured for deployment to Kubernetes on AWS EKS.

## Architecture

The application is divided into four main components:

- `udagram-api-feed` - Handles feed and image-related API operations
- `udagram-api-user` - Handles user registration and authentication
- `udagram-frontend` - Angular/Ionic frontend application
- `udagram-reverseproxy` - Nginx reverse proxy that routes requests to the backend microservices

The application uses:

- **AWS RDS PostgreSQL** for application data
- **AWS S3** for image storage
- **Docker** for containerization
- **Kubernetes / AWS EKS** for container orchestration
- **Nginx** as the reverse proxy
- **Travis CI configuration** for the CI/CD pipeline
- **Docker Hub** for container image storage

## Microservices

The original backend was separated into two independent backend services:

### Feed API

`udagram-api-feed`

Handles feed-related operations and communication with the image storage and database.

### User API

`udagram-api-user`

Handles user-related operations such as registration and authentication.

The two backend services can be built, deployed, and scaled independently.

## Docker

Each application component contains its own Dockerfile and can be built as an independent Docker image.

The following Docker images were created:

- `anabytesdocker/udagram-api-feed`
- `anabytesdocker/udagram-api-user`
- `anabytesdocker/udagram-frontend`
- `anabytesdocker/udagram-reverseproxy`

Example build commands:

```bash
docker build -t anabytesdocker/udagram-api-feed ./udagram-api-feed
docker build -t anabytesdocker/udagram-api-user ./udagram-api-user
docker build -t anabytesdocker/udagram-frontend ./udagram-frontend
docker build -t anabytesdocker/udagram-reverseproxy ./udagram-reverseproxy
```

The images are stored in Docker Hub.

Docker Hub evidence is available in:

```text
screenshots/DockerHub.png
```

## CI/CD - Travis CI

The project includes a `.travis.yml` configuration for the CI/CD pipeline.

The Travis CI pipeline is configured to:

1. Build the Feed API Docker image
2. Build the User API Docker image
3. Build the Frontend Docker image
4. Build the Reverse Proxy Docker image
5. Authenticate to Docker Hub using environment variables
6. Push the successfully built Docker images to Docker Hub

The Docker Hub username and password are supplied through environment variables rather than being hard-coded into `.travis.yml`.

### Travis CI Build Limitation

The Travis CI pipeline configuration has been implemented in `.travis.yml`.

However, an actual Travis CI build could not be executed because repository/build access was blocked by the Travis CI OSS/pricing request process.

A screenshot documenting the Travis CI access/request limitation is included as:

```text
screenshots/travis-ci-oss-request.png
```

The `.travis.yml` file remains included in the project to demonstrate the configured CI/CD build and Docker image publishing process.

## Kubernetes

Kubernetes configuration files are located in the `k8s/` directory.

The project contains Kubernetes deployments and services for:

- Feed API
- User API
- Frontend
- Reverse Proxy

### Deployments

The following deployment manifests are included:

```text
k8s/feed-deployment.yml
k8s/user-deployment.yml
k8s/frontend-deployment.yml
k8s/reverseproxy-deployment.yml
```

Each application deployment is configured with two replicas:

```yaml
replicas: 2
```

This allows multiple pods of each application component to run in the Kubernetes cluster.

### Services

The following Kubernetes service manifests are included:

```text
k8s/feed-service.yml
k8s/user-service.yml
k8s/frontend-service.yml
k8s/reverseproxy-service.yml
```

The backend microservices use Kubernetes `ClusterIP` services for internal communication.

The frontend and reverse proxy use `LoadBalancer` services to provide external access.

## AWS EKS Deployment

The Kubernetes application was deployed to an AWS EKS cluster.

Example commands used to deploy and inspect the application:

```bash
kubectl apply -f ./k8s/
kubectl get nodes
kubectl get pods
kubectl get services
kubectl get deployments
```

During the successful deployment, the application components were running as Kubernetes pods with two replicas configured for each deployment.

Deployment evidence is included in the `screenshots/` directory.

## Reverse Proxy

Nginx is used as the reverse proxy for the backend APIs.

Requests are routed to the appropriate Kubernetes service.

Example routing configuration:

```nginx
location /api/v0/feed {
    proxy_pass http://udagram-api-feed:8080;
}

location /api/v0/users {
    proxy_pass http://udagram-api-user:8080;
}
```

This allows the Feed and User APIs to operate as separate microservices while being accessible through the reverse proxy.

## Database

The application uses PostgreSQL hosted on AWS RDS.

The Kubernetes backend workloads connect to the RDS PostgreSQL database using port:

```text
5432
```

Network access between EKS and RDS is controlled using AWS security groups.

The EKS workloads and RDS database were configured to allow the required database communication.

Supporting security group evidence is included in:

```text
screenshots/RDS-EKS_SG.png
```

## S3 Storage

AWS S3 is used for image storage.

The S3 configuration is provided to the backend through environment configuration rather than being hard-coded directly into the application.

## Environment Configuration

Non-sensitive application configuration is supplied to Kubernetes using a ConfigMap:

```text
k8s/env-configmap.yml
```

This includes configuration such as:

- AWS region
- S3 bucket
- PostgreSQL host
- PostgreSQL database

Sensitive configuration is supplied separately using Kubernetes Secrets.

The real Kubernetes secret manifest is intentionally excluded from Git.

## Security

Sensitive credentials are not committed to the repository.

The Kubernetes secret file:

```text
k8s/env-secret.yml
```

is excluded using `.gitignore`.

Sensitive information such as:

- PostgreSQL username/password
- AWS credentials
- Docker Hub credentials

should be supplied using environment variables or Kubernetes Secrets.

No real credentials should be committed to source control.

## Project Structure

```text
.
├── udagram-api-feed/
│   └── Dockerfile
│
├── udagram-api-user/
│   └── Dockerfile
│
├── udagram-frontend/
│   └── Dockerfile
│
├── udagram-reverseproxy/
│   └── Dockerfile
│
├── k8s/
│   ├── env-configmap.yml
│   ├── feed-deployment.yml
│   ├── feed-service.yml
│   ├── user-deployment.yml
│   ├── user-service.yml
│   ├── frontend-deployment.yml
│   ├── frontend-service.yml
│   ├── reverseproxy-deployment.yml
│   └── reverseproxy-service.yml
│
├── screenshots/
│   ├── DockerHub.png
│   ├── kubectl get nodes.png
│   ├── kubernetes-pod.png
│   ├── kubernetes-pod-redeploy.png
│   ├── kubernetes-services.png
│   ├── kubernetes-describe_services.png
│   ├── kubernetes-describe_services-1.png
│   ├── kubernetes-describe_services-2.png
│   ├── kubernetes-deployments-config.png
│   ├── RDS-EKS_SG.png
│   └── travis-ci-oss-request.png
│
├── .travis.yml
├── .gitignore
└── README.md
```

## Deployment Evidence

The `screenshots/` directory contains evidence from the project implementation and deployment, including:

- Docker images available in Docker Hub
- AWS EKS Kubernetes nodes
- Running Kubernetes application pods
- Kubernetes services
- Kubernetes service configuration
- Kubernetes deployment configuration
- Pod redeployment
- RDS/EKS security group configuration
- Travis CI OSS/pricing access limitation

These screenshots document the successful Docker and Kubernetes portions of the project as well as the external limitation encountered when attempting to execute the Travis CI pipeline.

## Notes

The AWS environment used for this project is a temporary Udacity/VocLabs environment. AWS resources and credentials may become unavailable or be reset when the lab session expires.

The Kubernetes and AWS screenshots included in this repository were captured while the required resources were active.