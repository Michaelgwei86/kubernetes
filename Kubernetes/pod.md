
### ClusterIP
A `ClusterIP` service exposes the service on an internal IP in the cluster. This type of service is only accessible from within the cluster.

#### Example: Exposing Nginx with ClusterIP

1. **Create the Nginx Pod**:

`nginx-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

Apply the pod definition:

```sh
kubectl apply -f nginx-pod.yaml
```

2. **Create the ClusterIP Service**:

`nginx-clusterip-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

Apply the service definition:

```sh
kubectl apply -f nginx-clusterip-service.yaml
```

#### Accessing the Service

To access the `ClusterIP` service, you need to be inside the cluster. For example, you can use `kubectl port-forward` to access the service from your local machine:

```sh
kubectl port-forward svc/nginx-clusterip-service 8080:80
```

Then, open your browser and go to `http://localhost:8080`.

### NodePort
A `NodePort` service exposes the service on each node’s IP at a static port (the NodePort). A `NodePort` service is accessible from outside the cluster using `<NodeIP>:<NodePort>`.

#### Example: Exposing Nginx with NodePort

1. **Create the Nginx Pod**:

`nginx-pod.yaml` (same as above):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

Apply the pod definition:

```sh
kubectl apply -f nginx-pod.yaml
```

2. **Create the NodePort Service**:

`nginx-nodeport-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
  type: NodePort
```

Apply the service definition:

```sh
kubectl apply -f nginx-nodeport-service.yaml
```

#### Accessing the Service

To access the `NodePort` service, use the IP address of any of the cluster nodes and the NodePort (30080 in this example):

```sh
http://<NodeIP>:30080
```

### LoadBalancer
A `LoadBalancer` service provisions an external load balancer and is the most straightforward way to expose a service to the internet.

#### Example: Exposing Nginx with LoadBalancer

1. **Create the Nginx Pod**:

`nginx-pod.yaml` (same as above):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

Apply the pod definition:

```sh
kubectl apply -f nginx-pod.yaml
```

2. **Create the LoadBalancer Service**:

`nginx-loadbalancer-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

Apply the service definition:

```sh
kubectl apply -f nginx-loadbalancer-service.yaml
```

#### Accessing the Service

Kubernetes will provision an external IP address for the `LoadBalancer` service. You can get the external IP address using:

```sh
kubectl get services nginx-loadbalancer-service
```

Once the external IP is assigned, you can access the service directly using that IP.

### Summary

- **ClusterIP**: Only accessible within the cluster. Use for internal services.
- **NodePort**: Exposes the service on a port on each node’s IP. Accessible externally but requires knowledge of the node’s IP.
- **LoadBalancer**: Provisions an external load balancer and is the easiest way to expose a service to the internet in cloud environments.
