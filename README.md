# kubernetes-batch
## Follow the Installation file first to create cluster :

https://github.com/Bhagyash-raut/kubernetes-batch/blob/main/Installation%20of%20Cluster


## To create a Kubernetes Pod using the image bhagyash654/batch:v2, you can use either the kubectl run command or a YAML manifest.

## Method 1: Using kubectl run
kubectl run batch-pod --image=bhagyash654/batch:v2

Verify the pod: kubectl get pods

Get more details: kubectl describe pod batch-pod

Check logs: kubectl logs batch-pod

## Method 2: Using a YAML Manifest

Create a file named pod.yaml:  https://github.com/Bhagyash-raut/kubernetes-batch/blob/main/pod.yaml

Apply the manifest: kubectl apply -f pod.yaml

Check the pod status: kubectl get pods

kubectl describe pod batch-pod

## In Kubernetes, different controllers manage specific workloads depending on the requirements of applications. Among the most commonly used are Deployments, StatefulSets, and DaemonSets. Each serves unique purposes in orchestrating containerized applications.

Deployments
A Deployment ensures a specified number of pod replicas are running at any given time. Deployments are best suited for stateless applications.

StatefulSets
A StatefulSet is used for applications requiring unique identities and persistent storage for each pod. These are suited for stateful applications.

DaemonSets
A DaemonSet ensures that a copy of a specific pod runs on all or selected nodes within a cluster. DaemonSets are typically used for system-level applications.

Replication Controllers and ReplicaSets ensure that the specified number of Pod replicas are running at all times.

### List all pods
kubectl get pods

### List all deployments
kubectl get deployments

### List all ReplicaSets
kubectl get rs

### List all StatefulSets
kubectl get statefulsets

### List all DaemonSets
kubectl get daemonsets

### Show all resources
kubectl get all

### Describe a pod
kubectl describe pod <pod-name>

### View pod logs
kubectl logs <pod-name>

### Delete a pod
kubectl delete pod <pod-name>

### Delete all resources in the current namespace
kubectl delete all --all


## Kubernetes Networking: Intra-Pod and Inter-Pod Communication

Kubernetes networking is fundamental for ensuring smooth communication between various components, including pods, services, and external clients. It provides flexible networking configurations for intra-pod and inter-pod communication.

Intra-Pod Communication

	• Definition: Intra-pod communication refers to the communication between containers within the same pod.
	• Mechanism: Containers in a pod share the same network namespace, which means they: 
		
		 Share the same IP address.
		 Can communicate directly using localhost and exposed container ports.
Inter-Pod Communication
	
	• Definition: Inter-pod communication refers to the communication between pods.
	• Mechanism: 
	
		Kubernetes assigns each pod a unique IP address.
		pods communicate directly using these IP addresses or via Kubernetes services.
		Kubernetes ensures a flat network model where all pods can communicate without NAT.

## Kubernetes Service Types for Networking

1. Pod IP
	
	• Each pod gets a unique IP address within the cluster.
	• Enables direct communication between pods without port conflicts.

2. Container Port
	• The port exposed by the container inside the pod.
	• Used for intra-pod communication.

3. Node IP
	• IP address of the Kubernetes node.
	• Used when accessing services exposed via NodePort or LoadBalancer.

4. Node Port
	• A static port on the node that forwards traffic to the service.
	• Example: Node IP + Node Port allows access to services from outside the cluster.

5. LoadBalancer
	• Integrates with cloud provider load balancers to expose services externally.
	• Automatically assigns external IPs for access.

## Kubernetes Namespace

Namespaces provide a way to divide cluster resources between multiple users or teams.

Default Namespaces

	• default: The default namespace for resources without a namespace.
	• kube-system: For Kubernetes system resources.
	• kube-public: For publicly accessible resources.

## Kubernetes Deployment Strategies

Recreate Strategy
	
	• Terminates all existing pods before creating new ones.

	• Suitable for non-critical updates.

Rolling Update Strategy

	• Updates pods incrementally.
	• Ensures minimal downtime and availability during updates.

Canary Deployment
	
	• Deploys a small subset of new pods alongside existing ones to test the update.

Blue-Green Deployment
	
	• Creates a new set of pods ("blue") while the old set ("green") remains active, enabling a smooth transition.


## Kubernetes Type of Containers

Main Container
The primary container that serves the main purpose of the application. Examples include application servers or web servers.

Sidecar Container
An auxiliary container that provides supporting functionalities, such as logging, monitoring, or proxying.
	
	Examples:
		A logging container (e.g., Fluentd) that collects logs from the main container.
        A proxy container (e.g., Envoy) that manages network traffic.


## Kubernetes ConfigMap and Secret

A ConfigMap is a Kubernetes resource used to store non-confidential data as key-value pairs. It helps decouple configuration data from application code.
A Secret is a Kubernetes resource designed to store confidential data, such as passwords or API keys, in a secure and encoded format. Secrets are encoded using Base64 , providing a layer of obfuscation but not encryption.
Why Use ConfigMap and Secret?
	Separation of Concerns: Decouples configuration from application logic.
	Ease of Updates: Configuration changes do not require rebuilding or redeploying applications.
	Security: Secrets ensure sensitive data is handled securely.

Practical Steps
Step 1: Create the ConfigMap
Manifest File
Save the following content in a file named configmap.yaml:  https://github.com/Bhagyash-raut/kubernetes-batch/blob/main/ConfigMap.yaml

Run the following command to create the ConfigMap in your cluster: kubectl apply -f configmap.yaml
Verify the ConfigMap
View the created ConfigMap: kubectl get configmap  my-vars  -o yaml

Step 2: Create the Secret
Manifest File
Save the following content in a file named secret.yaml:  
Apply the Secret
Run the following command to create the Secret in your cluster: kubectl apply -f secret.yaml

Verify the Secret
To check the Secret:
kubectl get secret my-sec -o yaml
Note: The values will appear base64-encoded. To decode, use:
echo "QWRtaW4=" | base64 --decode

Step 3: Using ConfigMap and Secret in a Pod
Example Pod Manifest
Create a file named pod-with-config-and-secret.yaml with the following content: https://github.com/Bhagyash-raut/kubernetes-batch/blob/main/ConfigMap-and-Secret-ina-Pod.yaml

Apply the Pod Manifest
kubectl apply -f pod-with-config-and-secret.yaml

Verify the Pod Environment Variables
kubectl exec -it  pod-with-config-secret -- printenv | grep DB_
##Notes and Best Practices
ConfigMap Notes
	
	Non-Confidential Data: ConfigMaps should not store sensitive data.
	Dynamic Updates: ConfigMaps can be updated dynamically, and changes can reflect in running Pods if the configuration is mounted as a volume.
	Avoid Overloading: Use ConfigMaps for lightweight configurations to prevent complexity.
Secret Notes
	
	Secure Handling: Avoid storing Secrets in plain text files. Use tools like kubectl to manage them.
	Encryption: Enable encryption at rest for Secrets in your cluster for additional security.
	Access Control: Use RBAC to restrict access to Secrets.



