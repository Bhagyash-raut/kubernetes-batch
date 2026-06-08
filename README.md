# kubernetes-batch
## Follow the Installation file first to create cluster :

https://github.com/Bhagyash-raut/kubernetes-batch/blob/main/Installation%20of%20Cluster


## To create a Kubernetes Pod using the image bhagyash654/batch:v2, you can use either the kubectl run command or a YAML manifest.

# Method 1: Using kubectl run
kubectl run batch-pod --image=bhagyash654/batch:v2

Verify the pod: kubectl get pods

Get more details: kubectl describe pod batch-pod

Check logs: kubectl logs batch-pod

# Method 2: Using a YAML Manifest

Create a file named pod.yaml:  https://github.com/Bhagyash-raut/kubernetes-batch/blob/main/pod.yaml

Apply the manifest: kubectl apply -f pod.yaml

Check the pod status: kubectl get pods



kubectl describe pod batch-pod

