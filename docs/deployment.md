StreamingApp AWS Deployment Guide

## Deployment

AWS Region: ap-south-1
EKS Cluster: streamingapp-eks
Worker Nodes: 2 x t3.medium

## CI/CD

Jenkins builds five Docker images and pushes them to Amazon ECR using Git commit SHA and latest tags.

ECR repositories:
- streamingapp-auth
- streamingapp-streaming
- streamingapp-admin
- streamingapp-chat
- streamingapp-frontend

Jenkins AWS credential ID: aws-ecr-credentials-2

## Helm

The Helm chart is stored in streamingapp/. It deploys the frontend, auth, streaming, admin, chat, MongoDB, services, and MongoDB persistent storage.

Deploy with:

helm upgrade --install streamingapp .\streamingapp

## Storage

MongoDB uses a 5 GiB gp2 PersistentVolumeClaim backed by the AWS EBS CSI driver.

## Security

JWT_SECRET is stored in the Kubernetes Secret streamingapp-secrets and is not committed to Git.

## Scaling

The streaming deployment was scaled from 2 to 3 replicas and validated at 3/3 Ready.

## Monitoring

Amazon CloudWatch Container Insights collects application, dataplane, host, and performance logs.

## Final Validation

admin: 2/2
auth: 2/2
chat: 2/2
frontend: 2/2
mongodb: 1/1
streaming: 3/3

All application pods were Running with 0 restarts during final validation.

Frontend is exposed through an AWS LoadBalancer Service.
