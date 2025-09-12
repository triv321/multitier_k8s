# **Kubernetes Segment 4: Learnings & Debugging Playbook**

This document is a summary of the core concepts and a detailed incident log from our session on managing configuration and data in Kubernetes. It's designed to be a personal reference guide for solving common problems when deploying stateful applications.

## **1\. Core Concepts Learned**

This segment focused on the data an application needs to run. We moved beyond simple stateless web servers and learned to manage a complete, stateful application stack.

* **ConfigMaps**: The standard way to manage **non-sensitive** configuration data (like URLs, welcome messages, etc.). They decouple configuration from the container image, allowing you to update settings without rebuilding.  
  * **Analogy**: A shared bulletin board for your application.  
* **Secrets**: The secure way to manage **sensitive** data (passwords, API keys). The data is stored Base64 encoded, but its real security comes from Kubernetes' strict, role-based access control.  
  * **Analogy**: A secure safe deposit box.  
* **Persistent Storage (PVs & PVCs)**: The two-part model for providing stateful applications with a stable place to store data.  
  * **PersistentVolumeClaim (PVC)**: A **request for storage** made by your application. Analogy: The request form.  
  * **PersistentVolume (PV)**: The **actual storage** in the cluster that fulfills the request. Analogy: The storage locker.  
  * This model ensures that if a Pod dies, its replacement can reconnect to the same storage, preserving the application's state.

## **2\. The Debugging Playbook: A Real-World Incident Log**

During our hands-on project, we encountered a cascading series of failures. This is a realistic scenario and a masterclass in the systematic debugging workflow.

### **Incident \#1: The Service is Unreachable**

* **Symptom**: After deploying all our WordPress and MySQL files, the minikube service wordpress command failed with a SVC\_UNREACHABLE error.  
* **Diagnosis Tool**: kubectl get pods followed by kubectl describe pod \<pod-name\>.  
* **Investigation**: We saw that the wordpress-mysql pod was stuck in a Pending state. We then used describe on that pod.  
* **Root Cause**: The Events log showed the critical error: didn't find available persistent volumes to bind. Our PersistentVolumeClaim (the request) had nothing to connect to.  
* **Learning**: A default minikube cluster does not have an automatic storage provisioner enabled.  
* **Solution**: We enabled the required addon with minikube addons enable storage-provisioner and then did a clean redeployment of all our resources.

### **Incident \#2: The Application Error**

* **Symptom**: After fixing the storage issue, the website loaded but showed an application-specific error: Error establishing a database connection.  
* **Diagnosis Tool**: Logical deduction. The error was from WordPress, not Kubernetes, meaning the networking was working, but the application's configuration was wrong.  
* **Investigation**: We reviewed our wordpress-deployment.yaml and realized we had provided the database host and password, but not the username.  
* **Root Cause**: The WordPress container was trying to connect to the MySQL database with the wrong username (wordpress instead of root).  
* **Learning**: Application configuration is just as critical as infrastructure configuration. Always verify all required environment variables for a container image.  
* **Solution**: We added the WORDPRESS\_DB\_USER environment variable to the WordPress deployment and re-applied the file.

### **Incident \#3: The Image Pull Failure (DNS Error)**

* **Symptom**: During the Redis project, the redis pod was stuck in an ImagePullBackOff state.  
* **Diagnosis Tool**: kubectl describe pod \<pod-name\>.  
* **Investigation**: We used describe and checked the Events log.  
* **Root Cause**: The error was not a missing image, but a DNS failure: lookup docker-images-prod... no such host. This indicated a network connectivity problem *inside* the minikube cluster itself, preventing it from reaching the internet.  
* **Learning**: Local development clusters can sometimes get into a bad networking state, especially after a computer sleeps or changes networks.  
* **Solution**: We performed a clean restart of the cluster with minikube stop followed by minikube start. We also learned the alternative solution of pre-pulling the image directly onto the node with minikube ssh \-- docker pull \<image-name\>.

## **3\. Key Things to Remember**

* **The Debugging Hierarchy**: When a Pod is unhealthy, follow this order:  
  1. kubectl get pods (to see the high-level status).  
  2. kubectl describe pod (to see cluster-level events causing the issue).  
  3. kubectl logs (to see application-level errors inside the container).  
* **YAML Indentation is King**: The structure of a YAML file is defined by its indentation. containers and volumes are **siblings** under the spec of a Pod template.  
* **Labels and Selectors are the Glue**: The connection between a Service and its Pods is made by matching labels, not by names. A mismatch will cause the Service to have no endpoints.  
* **Bare Pods vs. Deployments**: A bare Pod is not self-healing. A Deployment (or other controller) is required to automatically replace a failed pod.