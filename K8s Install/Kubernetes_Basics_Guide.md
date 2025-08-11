# Kubernetes Basics Guide

## Table of Contents
- [What is Kubernetes](#what-is-kubernetes)
- [Kubernetes Core Concepts](#kubernetes-core-concepts)
- [Kubernetes Architecture](#kubernetes-architecture)
- [Kubernetes Object Types](#kubernetes-object-types)
- [Kubernetes Command Line Tool kubectl](#kubernetes-command-line-tool-kubectl)
- [Deploying Applications](#deploying-applications)
- [Service Discovery and Load Balancing](#service-discovery-and-load-balancing)
- [Storage Management](#storage-management)
- [Configuration Management](#configuration-management)
- [Kubernetes Networking](#kubernetes-networking)
- [Kubernetes Security](#kubernetes-security)
- [Monitoring and Logging](#monitoring-and-logging)
- [Extending Kubernetes](#extending-kubernetes)
- [Best Practices](#best-practices)

## What is Kubernetes

Kubernetes (often abbreviated as K8s) is an open-source container orchestration platform designed for automating the deployment, scaling, and management of containerized applications. It was originally designed by Google and is now maintained by the Cloud Native Computing Foundation (CNCF).

The primary goals of Kubernetes include:
- **Automated scheduling**: Automatically deploying containers to nodes in the cluster
- **Self-healing**: Automatically replacing and rescheduling failed containers
- **Horizontal scaling**: Automatically scaling applications up or down based on load
- **Service discovery and load balancing**: Automatically providing service discovery and load balancing for containers
- **Automated rollouts and rollbacks**: Gradually deploying applications with the ability to quickly roll back
- **Secret and configuration management**: Managing sensitive information and application configurations

## Kubernetes Core Concepts

### 1. Nodes
Nodes are the worker machines in a Kubernetes cluster, which can be either physical or virtual machines. Nodes are responsible for running containers and managing the workload of the cluster.

### 2. Pods
A Pod is the smallest deployable unit in Kubernetes, containing one or more containers. Containers within a Pod share network namespaces and storage volumes.

### 3. Control Plane
The control plane is the brain of Kubernetes, responsible for managing the cluster state. Key components include:
- **kube-apiserver**: Provides REST API interfaces
- **etcd**: Key-value store that saves cluster state
- **kube-scheduler**: Responsible for node scheduling
- **kube-controller-manager**: Runs controllers
- **cloud-controller-manager**: Interacts with cloud service providers

### 4. Worker Nodes
Worker nodes are the nodes that run the actual workload. Key components include:
- **kubelet**: Communicates with the control plane and manages containers
- **kube-proxy**: Maintains network rules
- **Container runtime**: Such as Docker, containerd, etc.

### 5. Namespaces
Namespaces are used to isolate groups of resources within a single cluster, typically used to separate different environments (e.g., development, staging, production).

### 6. Controllers
Controllers are control loops that continuously monitor the cluster state and ensure the actual state matches the desired state.

## Kubernetes Architecture

Kubernetes architecture consists of a control plane and worker nodes:

```
+----------------+      +-----------------+      +-----------------+
|                |      |                 |      |                 |
|   API Server   |<---->|   etcd          |      |                 |
|                |      |                 |      |                 |
+----------------+      +-----------------+      +-----------------+
       ^                                           ^
       |                                           |
       v                                           v
+----------------+      +-----------------+      +-----------------+
|                |      |                 |      |                 |
| Controller Mgr |      |    Scheduler    |      |   Cloud Mgr    |
|                |      |                 |      |                 |
+----------------+      +-----------------+      +-----------------+
       |                                           |
       |                                           |
       v                                           v
+-----------------------------------------------------------+
|                                                           |
|                    Worker Nodes                           |
|                                                           |
+-----------------------------------------------------------+
       |                   |                   |
       |                   |                   |
       v                   v                   v
+----------------+  +----------------+  +----------------+
|                |  |                |  |                |
|   Node 1       |  |   Node 2       |  |   Node 3       |
|                |  |                |  |                |
+----------------+  +----------------+  +----------------+
```

## Kubernetes Object Types

### 1. Deployment
Deployments are used to manage stateless applications, ensuring a specified number of Pod replicas are running.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

### 2. StatefulSet
StatefulSets are used to manage stateful applications, providing stable, unique identifiers and persistent storage for each Pod.

### 3. DaemonSet
DaemonSets ensure that a copy of a Pod runs on all (or some) nodes, commonly used for cluster storage, log collection, and monitoring.

### 4. Job and CronJob
Jobs create one or more Pods and ensure they successfully complete a task. CronJobs are used to schedule jobs to run at specific times.

### 5. Service
Services provide network abstraction for a set of Pods, enabling service discovery and load balancing.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

### 6. Ingress
Ingress manages HTTP/HTTPS routing rules, forwarding external requests to services within the cluster.

### 7. ConfigMap and Secret
ConfigMaps store non-sensitive configuration data, while Secrets store sensitive information like passwords and API keys.

### 8. PersistentVolume and PersistentVolumeClaim
PersistentVolume (PV) is a piece of storage in the cluster, while PersistentVolumeClaim (PVC) is a request for storage resources.

## Kubernetes Command Line Tool kubectl

kubectl is the command-line tool for interacting with the Kubernetes API. Here are some common commands:

### Basic Commands
- `kubectl get <resource>` - Get a list of resources
- `kubectl describe <resource> <name>` - Get detailed information about a resource
- `kubectl create -f <file>` - Create resources from a file
- `kubectl apply -f <file>` - Configure resources using a file
- `kubectl delete <resource> <name>` - Delete a resource
- `kubectl edit <resource> <name>` - Edit a resource

### Debugging Commands
- `kubectl logs <pod>` - View Pod logs
- `kubectl exec -it <pod> -- <command>` - Execute a command in a container
- `kubectl port-forward <pod> <local-port>:<remote-port>` - Forward ports
- `kubectl proxy` - Create a proxy server

### Configuration Commands
- `kubectl config get-contexts` - Get a list of contexts
- `kubectl config use-context <context-name>` - Switch contexts
- `kubectl cluster-info` - Display cluster information

## Deploying Applications

### Create a Deployment
```bash
kubectl create deployment nginx --image=nginx
```

### Check Deployment Status
```bash
kubectl get deployments
```

### Scale an Application
```bash
kubectl scale deployment/nginx --replicas=4
```

### Update an Application
```bash
kubectl set image deployment/nginx nginx=nginx:1.16.1
```

### Rollback a Deployment
```bash
kubectl rollout undo deployment/nginx
```

## Service Discovery and Load Balancing

### Create a ClusterIP Service (for internal cluster access)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

### Create a NodePort Service (access via node IP)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007
```

### Create a LoadBalancer Service (via cloud provider's load balancer)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

## Storage Management

### Using PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### Using PersistentVolumeClaim in a Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      volumeMounts:
        - name: my-volume
          mountPath: /usr/share/nginx/html
  volumes:
    - name: my-volume
      persistentVolumeClaim:
        claimName: my-pvc
```

## Configuration Management

### Using ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "INFO"
  API_ENDPOINT: "https://api.example.com"
```

### Using ConfigMap in a Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: my-image
      env:
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL
        - name: API_ENDPOINT
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: API_ENDPOINT
```

## Kubernetes Networking

### Pod Network
Pods have unique IP addresses in the cluster and can communicate directly with each other using these IP addresses.

### Service Network
Services have their own virtual IP addresses that can be used to access the service from within the cluster.

### CNI Plugins
Kubernetes uses Container Network Interface (CNI) plugins to configure Pod networks. Common CNI plugins include:
- Flannel
- Calico
- Weave Net

### Ingress Controllers
Ingress controllers (such as Nginx Ingress Controller, Traefik) implement the routing rules defined by Ingress resources.

## Kubernetes Security

### RBAC (Role-Based Access Control)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

### Pod Security Policies
Limit Pod behavior to enhance security, such as restricting privileged containers or access to the host network.

### Network Policies
Control network traffic between Pods to implement micro-segmentation.

### Secrets Management
Use Kubernetes Secrets or external tools (such as HashiCorp Vault) to manage sensitive information.

## Monitoring and Logging

### Prometheus Monitoring
Deploy a Prometheus server to collect and monitor application metrics, and use Grafana for visualization.

### EFK Logging Stack
- Elasticsearch: Stores and searches logs
- Fluentd: Collects and forwards logs
- Kibana: Visualizes logs

### Metrics Server
Deploy Metrics Server to provide container resource usage metrics.

## Extending Kubernetes

### Custom Resource Definitions (CRDs)
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: myresources.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                size:
                  type: string
  scope: Namespaced
  names:
    plural: myresources
    singular: myresource
    kind: MyResource
    shortNames:
    - myres
```

### Operator Pattern
Use Operator frameworks (such as KubeBuilder, Ansible Operator) to create automated operational tools.

### Custom Controllers
Develop custom controllers to manage custom resources.

## Best Practices

### Application Design
- Design stateless applications to make them easier to scale and migrate
- Use health checks and readiness probes
- Implement graceful startup and shutdown logic

### Cluster Management
- Use namespaces for environment isolation
- Set resource quota limits
- Regularly update and patch cluster components

### Security Best Practices
- Follow the principle of least privilege
- Use network policies to restrict Pod-to-Pod communication
- Regularly rotate keys and certificates

### Performance Optimization
- Properly configure resource requests and limits
- Use Horizontal Pod Autoscaler (HPA)
- Optimize image size and startup time

### Cost Control
- Use Cluster Autoscaler
- Optimize resource utilization
- Use Spot instances (low-cost instances provided by cloud providers)

## References

- [Kubernetes Official Documentation](https://kubernetes.io/docs/home/)
- [Kubernetes GitHub Repository](https://github.com/kubernetes/kubernetes)
- [CNCF Kubernetes Project](https://www.cncf.io/projects/)
