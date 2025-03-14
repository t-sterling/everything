# ☸️ Kubernetes Comprehensive Cheat Sheet

## **1. Installation & Version**
Kubernetes (k8s) is an open-source system for automating deployment, scaling, and management of containerized applications.
```sh
kubectl version --client           # Check kubectl version
kubectl cluster-info                # Get cluster information
kubectl config view                 # View Kubernetes configuration
```

---

## **2. Kubernetes Objects & Definitions**
Kubernetes objects are persistent entities that represent the state of the cluster.

### **Pods (Basic Deployable Unit)**
A Pod is the smallest deployable unit in Kubernetes that encapsulates one or more containers.
```sh
kubectl get pods                     # List all pods
kubectl get pods -o wide              # List pods with node information
kubectl describe pod mypod            # Show details of a specific pod
kubectl delete pod mypod              # Delete a pod
kubectl logs mypod                    # View logs of a pod
kubectl exec -it mypod -- sh          # Access a running pod interactively
```

### **Deployments (Manage Replica Pods)**
A Deployment manages ReplicaSets and ensures the desired state of application pods.
```sh
kubectl create deployment mydeploy --image=nginx  # Create a deployment
kubectl get deployments                           # List deployments
kubectl describe deployment mydeploy              # Show details of a deployment
kubectl scale deployment mydeploy --replicas=3    # Scale a deployment
kubectl rollout status deployment mydeploy        # Check rollout status
kubectl rollout history deployment mydeploy       # View rollout history
kubectl delete deployment mydeploy                # Delete a deployment
```

### **ReplicaSets (Ensure Availability)**
ReplicaSets ensure a specified number of identical pods are running.
```sh
kubectl get rs                     # List all ReplicaSets
kubectl describe rs myrs           # Show details of a ReplicaSet
kubectl delete rs myrs             # Delete a ReplicaSet
```

### **StatefulSets (For Stateful Applications)**
StatefulSets are used for stateful applications like databases.
```sh
kubectl get statefulsets           # List StatefulSets
kubectl describe statefulset mysts # Show details of a StatefulSet
kubectl delete statefulset mysts   # Delete a StatefulSet
```

### **DaemonSets (One Pod Per Node)**
DaemonSets ensure that a pod runs on every node in the cluster.
```sh
kubectl get daemonsets             # List DaemonSets
kubectl describe daemonset myds    # Show details of a DaemonSet
kubectl delete daemonset myds      # Delete a DaemonSet
```

### **Jobs & CronJobs (Batch Processing)**
Jobs run tasks that terminate after completion, while CronJobs run them on a schedule.
```sh
kubectl create job myjob --image=busybox -- echo "Hello Kubernetes"  # Create a job
kubectl get jobs                                                     # List jobs
kubectl delete job myjob                                             # Delete a job

kubectl create cronjob mycron --schedule="*/5 * * * *" --image=busybox -- echo "Hello"  # Create a cronjob
kubectl get cronjobs                                                 # List cronjobs
kubectl delete cronjob mycron                                        # Delete a cronjob
```

### **Services (Expose Applications)**
Services expose pods within or outside the cluster.
```sh
kubectl expose deployment mydeploy --type=ClusterIP --port=80  # Expose a deployment as a service
kubectl get services                                           # List all services
kubectl describe service myservice                            # Show details of a service
kubectl delete service myservice                              # Delete a service
```

### **Ingress (Routing External Traffic)**
Ingress manages HTTP and HTTPS routing to services inside the cluster.
```sh
kubectl get ingress                    # List all ingress rules
kubectl describe ingress myingress      # Show details of an ingress
kubectl delete ingress myingress        # Delete an ingress
```

### **ConfigMaps (Manage Configuration)**
ConfigMaps store non-sensitive configuration data as key-value pairs.
```sh
kubectl create configmap myconfig --from-literal=key=value  # Create a ConfigMap
kubectl get configmaps                                      # List ConfigMaps
kubectl describe configmap myconfig                         # Show details of a ConfigMap
kubectl delete configmap myconfig                           # Delete a ConfigMap
```

### **Secrets (Store Sensitive Data)**
Secrets store sensitive information like passwords and API keys.
```sh
kubectl create secret generic mysecret --from-literal=username=admin  # Create a secret
kubectl get secrets                                                   # List secrets
kubectl describe secret mysecret                                      # Show details of a secret
kubectl delete secret mysecret                                        # Delete a secret
```

### **Persistent Volumes & Claims (Storage Management)**
Persistent Volumes (PV) provide storage, while Persistent Volume Claims (PVC) request storage.
```sh
kubectl get pv                         # List persistent volumes
kubectl describe pv mypv                # Show details of a PV
kubectl delete pv mypv                   # Delete a PV

kubectl get pvc                         # List persistent volume claims
kubectl describe pvc mypvc               # Show details of a PVC
kubectl delete pvc mypvc                  # Delete a PVC
```

### **Namespaces (Isolate Resources)**
Namespaces logically separate cluster resources.
```sh
kubectl get namespaces                 # List all namespaces
kubectl create namespace mynamespace    # Create a new namespace
kubectl delete namespace mynamespace    # Delete a namespace
kubectl config set-context --current --namespace=mynamespace  # Switch to a namespace
```

---

## **3. Troubleshooting & Debugging**
### **Checking Cluster & Node Status**
```sh
kubectl get nodes                      # List nodes in the cluster
kubectl describe node mynode            # Show details of a node
kubectl top nodes                       # Show resource usage of nodes
kubectl top pods                        # Show resource usage of pods
```

### **Debugging Pods**
```sh
kubectl get events --sort-by=.metadata.creationTimestamp  # List recent events
kubectl logs mypod                                         # View pod logs
kubectl logs -f mypod                                      # Follow pod logs
kubectl describe pod mypod                                 # Show pod details
kubectl exec -it mypod -- sh                              # Open a shell in a pod
kubectl debug mypod --image=busybox --target=mycontainer  # Debug a container inside a pod
```

### **Checking Network Connectivity**
```sh
kubectl run testpod --image=busybox --rm -it -- ping myservice  # Test service connectivity
kubectl port-forward mypod 8080:80                               # Forward port from a pod to local machine
```

---

## **4. Managing Resources & Configuration**
### **Contexts & Configuration**
```sh
kubectl config get-contexts          # List available contexts
kubectl config current-context       # Show current context
kubectl config use-context mycontext # Switch context
```

### **Applying & Managing Resources**
```sh
kubectl apply -f myconfig.yaml       # Apply a configuration file
kubectl delete -f myconfig.yaml      # Delete resources from a file
kubectl get all                      # List all resources in the current namespace
```

### **Resource YAML Example (Deployment)**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp-container
        image: nginx
        ports:
        - containerPort: 80
```

---

## **5. Cleaning Up Resources**
```sh
kubectl delete all --all              # Delete all resources in a namespace
kubectl delete namespace mynamespace  # Delete an entire namespace
kubectl drain mynode --ignore-daemonsets --delete-local-data  # Prepare a node for maintenance
kubectl delete node mynode            # Delete a node from the cluster
```

---

### **Happy Kubernetes-ing! ☸️🚀**
