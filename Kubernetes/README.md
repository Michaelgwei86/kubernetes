                             #Author: Prince Chafah Forchu Sani
                             #https://www.linkedin.com/in/prince-chafah-forchu-sani-691534158/
  # KUBERNETES:
 

 ## K8S ARCHITECTURE:
   
  
### ControlPlane: MasterNode
     
  + 1. **ApiServer:**
    + Primary mgnt component/ exposes k8s API serving as frontend interface for all cluster operations
          handles restful Api from kubelet
  + When you run a kubectl command, it communicates with the kube API and then it gets the data from the ETCD
  + The request is first authenticated and then validated. 
  + 2. **ETCD:**
    + It is a key-value store, It stores the cluster's configuration data,
  + 2 **Scheduler:**
    + Responsible for making decisions about pod placement on worker nodes in the cluster.
    + It examines resource requirements, quality-of-service constraints, 
      affinity, anti-affinity, and other policies to determine the most suitable node for running a pod.
      it doesn't place resources on nodes but makes the decision
  + 3. **ControllerManagers:**
   + Manages Node lifecycle, desired pod number, and services it continuously monitors the state of resources in
       the cluster and ensures that they match the desired state.
   + Node Controler: monitors the status of the nodes every 5sec. it waits for 40 secs and if unreachable, it evicts
      the pods running on the node.
  + ## ReplicationController:
 + it monitors the status of the replica set and makes sure the desired states are maintained.

         ReplicaSet
         DaemonSet
         ReplicationController
 ## WorkerNodes:
 
  + 1. **kubelet**: 
     responsible for managing and operating containers. communicate with control-plane 
  it registers nodes into the cluster. It monitors nodes and pods in the cluster every 10 minutes and 
  relates feedback to the API which is stored in the etcd.cluster
  + 2. **container runtime**:
   [Container-d] docker pulling containered images
  + 3. **kube-proxy**: 
    enables network communication and load balancing between pods and services within the cluster.
  every pod in a cluster can communicate with other pods in the cluster by using IP address of the pod.
  to access each pod, a service has to be created and then you can access the pod by using the service name.
  the service is not an object, the kube-proxy creates rules that allow traffic routing within the cluster.

         ClusterIP
         NodePort
         LoadBalancer
+ kubernetes-client:
```bash
  kubectl  
      kubectl create/delete/get/describe/apply/run/expose 
``` 
      kubeconfig [.kube/config ] file will authenticate 
                                 the caller admin/Developer/Engineer 

In Kubernetes Quality of Service (QoS) refers to the priority and resource guarantees provided to different 
workloads or pods within a cluster. Kubernetes provides mechanisms to manage QoS to ensure that critical 
workloads receive the necessary resources and performance, while also allowing for efficient resource 
utilization.

## QUALITY OF SERVICE IN KUBERNETES:

Kubernetes offers three levels of Quality of Service:

+ 1. **BestEffort:**
   - Pods with BestEffort QoS are not guaranteed any specific amount of resources.
   - They are scheduled onto nodes based on availability, and they can use whatever resources are available at that time.
   - These pods are the first to be evicted if resources become scarce.

+ 2. **Burstable:**
   - Pods with Burstable QoS are guaranteed a minimum amount of CPU and memory.
   - These pods can burst beyond their guaranteed minimum if the resources are available.
   - If other pods on the node need resources, Burstable QoS pods might be limited in their burst capacity.

+ 3. **Guaranteed:**
   - Pods with Guaranteed QoS are guaranteed a specific amount of CPU and memory.
   - These pods are not allowed to exceed the resources they have been allocated.
   - Kubernetes tries to ensure that nodes have enough available resources to meet the guaranteed requirements.

Kubernetes determines the QoS level of pods based on the resource requests and limits specified in the pod's configuration:

- **Resource Requests:** The minimum amount of resources a pod requires to run. 
    These requests are used by the scheduler to make placement decisions.
- **Resource Limits:** The maximum amount of resources a pod is allowed to use.
    Exceeding these limits could lead to throttling or pod termination.

Kubernetes uses the relationship between requests and limits to categorize pods into different QoS classes. 
The actual QoS class assigned to a pod depends on how its requests and limits are set:

- **BestEffort:** Pods with no resource requests or limits.
- **Burstable:** Pods with resource requests, but without memory limits or with memory limits lower than their requests.
- **Guaranteed:** Pods with both CPU and memory limits set to be higher than or equal to their resource requests.

Setting appropriate resource requests and limits for pods is crucial for efficient resource allocation and QoS management 
within a Kubernetes cluster. Properly configured QoS levels help ensure that critical workloads are prioritized 
and that the cluster operates smoothly without resource contention issues.

## PODS:
- The aim is to deploy applications as containers running on a set of machines. 
- Containers do not run directly on the node but on Pods. 
- Pod is the single instance of an application and the smallest object in k8s/.
- If the user base increases, you can scale additional pods on the node, and if the node runs out of storage,
  you can spin up new nodes and assign new pods of the same or diff containers to it.
- Two containers of the same kind can not run in the same pod.
- There are multi-container pods which are helper containers running a process for the pod. they both live and die 
at the same time. they can refer to each other using localhost.
```bash
kubectl run nginx --image nginx
```
it creates a pod call nginx and also pulls the image nginx from a public docker repo 

apiVersion: # this is the version of the k8s API, it is mandatory Pod: v1 , service: v1 
#replicaSet: apps/v1 , Deployment: apps/v1. It is also a string value 
kind: # This refers to the type of object to be created such as Pod, ReplicaSet, Deployment, etc string
metadata: # This is data about the object such as name and labels. Metadata is a dictionary, it is indented
   name: myapp #
   labels: # it is a dictionary and can take any kind of key-value pair such as
      app: myapp # It is a string
      type: front-end
note: you can only add name and labels under metadata or specification from k8s 
spec: # this provides additional information about the object to create. it varries per object
  containers:  list/array
      - name: nginx-container  # first item in the list
        image: nginx
      - name:
        image:

+  example:
```sh
apiVersion: v1
Kind: Pod
metadata:
  name: myapp 
  labels:
    app: myapp
    type: front-end
spec:
  container:
  - name: nginx-container
    image: nginx
```
```sh
- kubectl apply/create -f <filename> #to create declaratively from a yml file
- kubectl get/describe pods <podname> #to get the pod spec
- kubectl create deployment redis-deployment --image=redis123 --dry-run=client -o yaml > deployment.yaml
- kubectl create deployment redis-deployment --image=redis123 -o yaml #print out output
- kubectl edit pod <podname>
```
## REPLICASETS:
Controllers are the brain behind k8s, they monitor k8s objects and respond accordingly.
- the replication controller helps increase the number of pods in the node for high availability.
- It also serves as recreating a pod if it fails.
- It creates pods across nodes to balance the load
- replication controller is replaced by replicasets
- it maintains the desired number of pods specified in your object definition file 
```sh
apiVersion: v1
kind: ReplicationController    #DEPRICATED
metadata:
  name: myapp-rc
      labels:
        app: myapp
        type: fe
spec:
  template: # Here you provide a pod template which is intended to be managed by the replicationcontroller
    metadata:
        name: myapp 
        labels:
          app: myapp
          type: front-end
    spec:
       container:
       - name: nginx-container
         image: nginx 
  replicas: 3
```
```sh
- kubectl create -f <filename>
- kubectl get rc 
```
#replicasets requires a selector field | it is not a must
#It helps the replica set defines what pods fall under it although pod spec has already been mentioned in the spec
#This is because it can manage pods that were not created to be managed by the rs
```sh
apiVersion: apps/v1
Kind: ReplicaSet
metadata: 
  name: myapp-rc
  labels:
    app: myapp 
    type: front-end
spec:
  template:
    container:
    - name: nginx-container
      image: nginx 
  replicas: 3
  selector:
    matchLabels:
      type: front-end # This must match the label that was input in the object metadata section
```
```sh
k get rs 
k get pods 
```
## Labels and Selectors:
+ Labels are used as filters for ReplicaSet. Labels allow the rs to know what pod in the cluster or nodes 
placed under its management since there could be multiple pods running in the cluster.
+ The template definition section is required in every rs, even for pods that were created before the rs 
+ This is due to the fact that if the pod fails and is to be recreated, it will need the spec to recreate it

- if you want to scale from 3 to 6 replicas, update the replicas to 6 and run
```sh
 kubectl replace -f <filename>
 kubectl scale --replicas=6 <filename>
 kubectl scale -- replicas replicaset name 
 kubectl edit pod/rs/rc/deploy <podname>
```
## Deployments

+ When you want to deploy an application, you may want to deploy several pods of that application for high 
availability. When a newer version of that application is available in docker, you want to gradually update
to avoid downtime of the application.
suppose an update has issues, you will want to do a rollback to the previous working version   
+ You can also make changes such as resources and the number of pods.
+ Deployment provides the capability to upgrade the underlying instance such as rolling-update, pause, upgrade

```sh
apiVersion: apps/v1
Kind: Deployment
metadata: 
  name: myapp-deployment
  labels:
    app: myapp 
    type: front-end
spec:
  template:
    metadata:
      name: myapp
      labels:
        app: myapp
        type: front-end
    spec:
      container:
      - name: nginx-container
        image: nginx 
  replicas: 3
  selector:
    matchLabels:
      type: front-end # This must match the label that was input in the object metadata section
```
```sh
kubectl get deployment
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```

## Services
Kubernetes service enables communication between various components within and outside the application and between  
other applications and users. Services enable the frontend app to be available to users and btw frontend and backend

For external communication,

the Kubernetes node has an IP, the host OS which is in the same network has an IP (private), and the pod has an ip but on  
a separate network
To access the application externally, the k8s service enables that communication from pods on nodes

TYPES:

1. NodePort:
The k8s service maps a port on the Node to a port on the Pod(target)
the NodePort is a port range on the Node that gives external access. it ranges from 30000-32767

apiVersion: v1
Kind: Service
metadata: 
  name: myapp-svc
spec:
  type: NodePort
  ports:
    - targetPort: 80  # (port on Pod). it will assume port if not specified
      port: 80 # port on service. this is a mandatory field
      nodePort: 30008  #external node port on service 30000-32767. a random port will be allocated if not specified
  selector:
    app: myapp # This is the label that was used in the deployment metadata section
    type: front-end

kubectl create -f <filename>
kubectl get svc 
curl IP:30008

- In the case of multiple pods running the same application, you need to maintain the labels and selector section with
the same values. the service uses a random algorithm to route traffic to all pods with that same label.
- If the pods are running on different nodes in the cluster, you can access it by calling the ip of any node in the  
cluster. Service is a cluster-wide resource in k8s.

2. *ClusterIP*:

  A full-stack web app typically has a number of pods such as frontend pods hosting a web server,
  the backend hosting the app, and pods hosting a db. Kubernetes service can group all pod groups together 
  and provide a single backend to access the pods. you can create another service for all pods running the db 
  these pods for diff applications can therefore be scaled like microservices without impacting the other.
  A separate svc for frontend, for backend, and for db. 
  - This type of service that allows communication between pods in a cluster is called cluster ip service.

apiVersion: v1
Kind: Service
metadata: 
  name: backend
spec:
  type: ClusterIP
  ports:
    - targetPort: 80  # (port on Pod). it will assume port if not specified
      port: 80 # port on service. this is a mandatory field
  selector:
    app: myapp # This is the label that was used in the deployment metadata section
    type: backend

kubectl create -f <filename>
kubectl get svc 

the service can be accessed by other pods in the cluster using the service name or ClusterIP

3. LoadBalancer:
When multiple pods of an application are deployed, they can all be accessed by using the diff IPs of the nodes
mapped to the nodePort. 
But end users need to be provided with a single endpoint that can route traffic to all the pods.
K8s have native support for cloud platforms 

apiVersion: v1
Kind: Service
metadata: 
  name: backend
spec:
  type: LoadBalancer
  ports:
    - targetPort: 80  # (port on Pod). it will assume port if not specified
      port: 80 # port on service. this is a mandatory field
  selector:
    app: myapp # This is the label that was used in the deployment metadata section
    type: backend

NAMESPACES:

  A namespace is simply a distinct working area in k8s where a defined set of resources and rules and users can  
  be assigned to a namespace. 
- By default, a k8s cluster comes with a default namespace. Here, a user can provision resources.
- the subsystem namespace is also created by default for a set of pods and services for its functioning.
- The kubepublic is also created to host resources that are made available to the public.
- Within a cluster, you can create diff namespaces for different projects and allocate resources to that namespace.
- Resources from the same namespaces can refer to each other by their names,
- they can also communicate with resources from another namespace by their names and append their namespace.
eg msql.connect("db-service.dev.svc.cluster.local")

kubectl get pods > will list only pods in the default namespace
kubectl get pods --namespace=kubesystem

kubectl apply -f <filename>  ==> will create object in the default namespace
kubectl create -f <filename> --namespace=kubesystem  ==> will create object in the kubesystem namespace

to ensure that your resources are always created in a specific namespace, add the namespace block in the resources
definition file

apiVersion: v1
Kind: Service
metadata: 
  name: backend
  namespace: dev  # This resource will always be created in the dev namespace, and will create the ns if it didn't exist
spec:
  type: LoadBalancer
  ports:
    - targetPort: 80  # (port on Pod). it will assume port if not specified
      port: 80 # port on service. this is a mandatory field
  selector:
    app: myapp # This is the label that was used in the deployment metadata section
    type: backend

apiVersion: v1
Kind: NameSpace
metadata:
  name: dev
OR 
 kubectl create namespace dev
 kubectl get ns 

to set a namespace as the default namespace so that you don't always have to pass the NameSpace command, you need to set
set the namespace in the current context

kubectl config set-context $(kubectl config current-context) --namespace=dev
contexts are used to manage all resources in clusters from a single system 

to view resources in all NameSpaces use the 
kubectl get pods --all-namespaces

Resource Quota:
  to set a limit for resources in a namespace, create a resource-quota object in

apiVersion: v1
Kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi

kubectl apply -f <filename>



Declarative and Imperative :
  these are different approaches to creating and managing infrastructure in IaC.
- The Imperative approach is giving just the required infrastructure and the resource figures out the steps involved.
eg kubectl create deployment nginx --image nginx
    kubectl set image deployment nginx nginx=nginx:1.18
    kubectl replace -f nginx.yaml

    it creates resources quickly but is difficult to edit or understand what was involved.

- in the Declarative approach, a step-by-step approach through writing configuration files
   Here we can write configuration files 
   Here, changes can be made as well as the resources can be versioned.
   resources can also be edited in the running state using the kubectl edit command
   The best practice is always to edit the configuration file and run the replace command rather than using the edit command.

   when you use the kubectl apply command, it will create objects that do not exist, and when you want to update the object,
   edit the yml file and run kubectl apply again and the object will pick up the latest changes in the file.

- kubectl run --image=nginx nginx 
- kubectl create deployment --image=nginx nginx 
- kubectl edit deployment nginx 
- kubectl scale deployment nginx --replicas=5
- kubectl set image deployment nginx nginx=nginxv

kubectl run nginx --image=nginx --dry-run=client -o yaml    > nginx-deployment.yaml
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
k create deploy redis-deploy --image=redis --replicas=2 --namespace=dev-ns


kubectl run httpd --image=httpd:alpine --port=80 --expose   #will create pod and service


when you run a kubectl apply command, if the object stated in the file does not exist, it is created.
another live object configuration is created with additional fields and can be viewed using the  
- kubectl edit/describe object <objectName>
there is the last applied file that provides details about the last image of the live configuration.
