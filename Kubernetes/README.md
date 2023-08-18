  # KUBERNETES:
 

 ## K8S ARCHITECTURE:
   
  
### 1.ControlPlane: MasterNode
     
  + ApiServer:  (Primary mgnt component/ exposes k8s API serving as frontend interface for all cluster operations
          handles restful Api from kubelet)

  + When you run a kubectl command, it communicates with the kube API and then it gets the data from the ETCD
  + The request is first authenticated and then validated. 
  ++ ETCD:  It is a key-value store, It stores the cluster's configuration data,
  ++ Scheduler: responsible for making decisions about pod placement on worker nodes in the cluster.
   It examines resource requirements, quality-of-service constraints, 
   affinity, anti-affinity, and other policies to determine the most suitable node for running a pod.
   it doesn't place resources on nodes but makes the decision
  ++ ControllerManagers: Manages Node lifecycle, desired pod number, and services 
  it continuously monitors the state of resources in the cluster and ensures that they match the desired state.
  ++ Node Controler: monitors the status of the nodes every 5sec. it waits for 40 secs and if unreachable, it evicts
  the pods running on the node.
  ++ ReplicationController: it monitors the status of the replica set and makes sure the desired states are maintained.

         ReplicaSet
         DaemonSet
         ReplicationController
 ## WorkerNodes:
    ====
  + kubelet: 
    responsible for managing and operating containers. communicate with control-plane 
  it registers nodes into the cluster. It monitors nodes and pods in the cluster every 10 minutes and 
  relates feedback to the API which is stored in the etcd.cluster
  + container runtime:
   [Container-d] docker pulling containered images
  + kube-proxy: 
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
   =================================

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
  ======

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
   ============
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
  =====================
+ Labels are used as filters for ReplicaSet. Labels allow the rs to know what pod in the cluster or nodes 
placed under its management since there could be multiple pods running in the cluster.
+ the template definition section is required in every rs, even for pods that were created before the rs 
+ this is due to the fact that if the pod fails and is to be recreated, it will need the spec to recreate it

- if you want to scale from 3 to 6 replicas, update the replicas to 6 and run
```sh
 kubectl replace -f <filename>
 kubectl scale --replicas=6 <filename>
 kubectl scale -- replicas replicaset name 
 kubectl edit pod/rs/rc/deploy <podname>
```
