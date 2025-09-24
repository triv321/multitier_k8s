# **Kubernetes Multi-Tier Microservice Communication**

This repo is a hands-on demonstration of a fundamental pattern in modern cloud-native architecture: secure, internal communication between microservices on a Kubernetes cluster. The project deploys a distinct frontend and backend application and uses a ClusterIP Service to enable them to communicate, showcasing the power of Kubernetes for building robust, multi-tier systems.

## **Architecture: The Secure Two-Tier Model**

This project provisions a classic two-tier application. The key principle is that the backend (e.g., a database or internal API) is completely isolated from external traffic and can only be reached by other trusted applications running inside the same cluster.

* **Backend Deployment**: A set of pods running a simple HTTP server that acts as our internal API.  
* **Frontend Deployment**: A set of pods running a basic client application. These pods need to communicate with the backend.  
* **ClusterIP Service**: This is the crucial networking component. It provides a stable, internal-only IP address and a DNS name (backend-service) for our backend pods. The frontend connects to this stable service name, and the service intelligently load balances the requests to a healthy backend pod. This service is **not** accessible from outside the Kubernetes cluster.

## **Core Concepts & Professional Practices Demonstrated**

* **Microservice Architecture**: The project is a practical implementation of a multi-tier system, a foundational pattern for building scalable and maintainable applications.  
* **Service Discovery**: Demonstrates how Kubernetes' internal DNS automatically allows services to find each other by name (http://backend-service), completely abstracting away the unstable, individual pod IP addresses.  
* **Secure Networking**: Proves the ability to use different Kubernetes Service types for different security needs. A ClusterIP service is used to ensure the backend is completely isolated and cannot be accessed from the public internet.  
* **Declarative Manifests**: The entire application is defined in clear, version-controllable YAML files, a core tenet of professional Kubernetes management.  
* **Debugging and Verification**: Uses kubectl exec to get a shell inside a running container to perform live, end-to-end connectivity tests, a critical skill for any infrastructure engineer.

## **Repository Contents**

* **backend-deployment.yaml**: Defines the Kubernetes Deployment for our backend http-echo server pods.  
* **backend-service.yaml**: Creates the ClusterIP Service that provides the stable internal endpoint for the backend.  
* **frontend-deployment.yaml**: Defines the Kubernetes Deployment for our frontend ubuntu client pods.

## **How to Use This Repository**

**Prerequisites**:

* A running Kubernetes cluster (e.g., minikube).  
* The kubectl command-line tool.

**Execution**:

1. **Apply the Manifests**: Apply all three manifest files to your cluster. The order does not matter.  
   kubectl apply \-f backend-deployment.yaml  
   kubectl apply \-f backend-service.yaml  
   kubectl apply \-f frontend-deployment.yaml

2. **Verify Pods are Running**: Wait for all pods to be in the Running state.  
   kubectl get pods

### **Verification: The Proof of Connection**

This is the final and most important step. We will get a shell inside one of our frontend pods and prove that it can communicate with our isolated backend.

1. **Get a Frontend Pod Name**:  
   \# Get the name of one of your frontend pods  
   kubectl get pods \-l app=my-frontend

2. **Exec into the Pod**: Use the name from the previous step to get an interactive shell.  
   \# Replace \<your-frontend-pod-name\> with the actual pod name  
   kubectl exec \-it \<your-frontend-pod-name\> \-- /bin/bash

3. **Install curl**: Inside the pod's shell, install the necessary tool.  
   \# This is a one-time setup inside the container  
   apt-get update && apt-get install \-y curl

4. **Test the Connection**: Send a request from the frontend to the backend using its stable service name.  
   curl http://backend-service

**Expected Outcome**: The command will successfully connect, and you will see the message from the backend server printed to your terminal:

\--- Hello from the Backend\! \---  
