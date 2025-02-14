# Istio Setup Guide

## Overview

Istio is a service mesh that provides traffic management, security, and observability for microservices. This guide covers setting up Istio and using Virtual Services, Gateways, and Destination Rules to manage traffic flow.

## Prerequisites

- A Kubernetes cluster (Minikube, GKE, EKS, or AKS)
- `kubectl` installed and configured
- Istio installed on the cluster
- An application deployed in the cluster

## Install Istio

### 1. Download Istio

Check out the official documentation for the latest version: [Istio Installation](https://istio.io/latest/docs/setup/install/istioctl/)

```sh
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.24.3
export PATH=$PWD/bin:$PATH
```

### 2. Install Istio on the cluster

```sh
istioctl install
```

### 3. Deploy Add-ons (Kiali, Jaeger, Prometheus, Grafana)

```sh
kubectl create -f samples/addons/
```

### 4. Create a Namespace and Enable Istio Injection

```sh
kubectl create ns dogbreed
kubectl label namespace dogbreed istio-injection=enabled
```

## Deploy the Application

Follow this repository to deploy the application with two versions (`v1` and `v2`). It's a Golang-based application that implements CRUD operations on a database to store dog breed data. The difference between the two versions is that `v2` introduces a new feature called **DeleteMany**, which is absent in `v1`.

This setup helps to observe how traffic is shifted between two versions using Istio.

### Repository:
[Dogbreed API](https://github.com/invisiblelad/dogbreed-api)

### Install Dependencies:

```sh
helm install mongo mongo
helm install dogbreed dogbreedv1
helm install dogbreed dogbreedv2
```

## Istio Configuration Files

### 1. **istio-virtualservice.yaml**

This file defines the **traffic shifting algorithm**, where we specify the percentage of traffic routed to each version of the application.

### 2. **istio-destination.yaml**

This file consists of **Destination Rules**, which define traffic routing policies for the `dogbreed-service` in the `sample` namespace. It creates subsets based on the version labels of the `dogbreed-service` pods, routing traffic accordingly to `v1` and `v2`.

### 3. **istio-gateway-ingress-https.yaml**

This Gateway resource configures **ingress traffic** for the `dogbreed-service` in the `sample` namespace.

Since Istio requires the TLS secret to be in the `istio-system` namespace, you should create it as follows:

```sh
kubectl create -n istio-system secret tls dogbreed-tls \
  --key=dogbreed.key \
  --cert=dogbreed.crt
```

## Monitor Traffic Shifting

Once the setup is complete, you can execute CRUD operations and monitor traffic shifting between `v1` and `v2` using Istio observability tools like **Kiali** and **Grafana**.

---

This guide provides a complete setup to deploy Istio and observe traffic management.🚀

