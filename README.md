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
