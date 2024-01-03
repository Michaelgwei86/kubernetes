                             ** Author: Prince Chafah Forchu Sani **
                             ** https://www.linkedin.com/in/prince-chafah-forchu-sani-691534158/ **
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
 ## Node Components:
 
  + 1. **kubelet**: 
    The Kubelet is responsible for managing and ensuring the running of containers created by Kubernetes. communicate with control-plane 
  it registers nodes into the cluster. It monitors nodes and pods in the cluster every 10 minutes and 
  relates feedback to the API which is stored in the etcd.cluster
  + 2. **container runtime**:
    A fundamental component that empowers Kubernetes to run containers effectively. It is responsible for managing the execution and lifecycle of containers within the Kubernetes environment.
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

## PODS:
A Pod is a group of one or more containers, with shared storage and network resources, and a specification for how to run the containers
- The aim is to deploy applications as containers running on a set of machines. 
- Containers do not run directly on the node but on Pods. 
- Pod is the single instance of an application and the smallest object in k8s.
- If the user base increases, you can scale additional pods on the node, and if the node runs out of storage,
  you can spin up new nodes and assign new pods of the same or diff containers to it.
- Two containers of the same kind can not run in the same pod.
- There are multi-container pods which are helper containers running a process for the pod. they both live and die 
at the same time. they can refer to each other using localhost.

```bash
source <(kubectl completion bash) # set up autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
alias k=kubectl
complete -o default -F __start_kubectl k
```
```bash
kubectl run nginx --image nginx
```
it creates a pod call nginx and also pulls the image nginx from a public docker repo 

apiVersion: # this is the version of the k8s API, it is mandatory Pod: v1 , service: v1 
#replicaSet: apps/v1 , Deployment: apps/v1. It is also a string value 
kind: # This refers to the type of object to be created such as Pod, ReplicaSet, Deployment, etc string
metadata: # This is data about the object such as name and labels. Metadata is a dictionary, it is indented
   name: myapp #
   labels: # It is a dictionary and can take any kind of key-value pair such as
      app: myapp # It is a string
      type: front-end
note: you can only add names and labels under metadata or specifications from k8s 
spec: # This provides additional information about the object to create. it varies per object
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
    ports:
    - containerPort: 80
```
```sh
- kubectl apply/create -f <filename> #to create declaratively from a yml file
- kubectl get/describe pods <podname> #to get the pod spec
- kubectl create deployment redis-deployment --image=redis123 --dry-run=client -o yaml > deployment.yaml
- kubectl create deployment redis-deployment --image=redis123 -o yaml #print out output
- kubectl edit pod <podname>
```

# POD AND CONTROLLERS - KUBERNETES WORKLOAD:

## Deployments
+ When you want to deploy an application, you may want to deploy several pods of that application for high 
availability. When a newer version of that application is available in docker, you want to gradually update
to avoid downtime of the application.
suppose an update has issues, you will want to do a rollback to the previous working version   
+ You can also make changes such as resources and the number of pods.
+ Deployment provides the capability to upgrade the underlying instance such as rolling update, pause, upgrade, Rollback,
+ In creating a deployment, a ReplicaSet and Pod are created in the background

```sh
apiVersion: apps/v1
Kind: Deployment
metadata: 
  name: myapp-deployment
  labels:
    app: myapp 
    type: front-end
spec:
  template:  #Normal Pod template except ApiVersion and Kind
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
      type: front-end # must match .spec.template.metadata.labels, or it will be rejected by the API.
```
```sh
kubectl get deployment
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
kubectl edit deployment/nginx-deployment
kubectl rollout status deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment --to-revision=2
kubectl get rs
kubectl get pods
kubectl describe deployments
kubectl scale deployment/nginx-deployment --replicas=10
```
## REPLICASETS:
A ReplicaSet maintains a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods
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
#It helps the replica set define what pods fall under it although pod spec has already been mentioned in the spec
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
## AUTOSCALER
https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
You can use a Horizontal Pod AutoScaler (HPA)
```sh
#kubectl autoscale rs myapp-rc --cpu-percent=50 --min=1 --max=10
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-scaler
spec:
  scaleTargetRef:
    kind: ReplicaSet
    name: myapp-rc
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

You can generate load on your pods by running the command
```sh
# Run this in a separate terminal
# so that the load generation continues and you can carry on with the rest of the steps
kubectl run -i --tty load-generator --rm --image=nginx --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```
```sh
# type Ctrl+C to end the watch when you're ready
kubectl get hpa myapp-rc --watch
k get pods 
```
## STATEFULSETS:
+ StatefulSet is the workload API object used to manage stateful applications.
+ Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods
+ Though susceptible to failure, they provide persistence to storage volumes and can match existing volumes to Pods replaced from failures
+ Stable, unique network identifiers.
+ Stable, persistent storage.
+ Ordered, graceful deployment and scaling.
+ Ordered, automated rolling updates.

```sh
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  minReadySeconds: 10 # by default is 0
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

## DEAMONSETS:

Deamonsets are like replicasets which help you run one instance of pods, but it runs one copy of your pod on every  
node on the cluster.
the deamonset ensures that one copy of the pod is always running on every node in the cluster.
A use case is if you are deploying a log collecting or monitoring agent.
objects like the kube-proxy and network use deamonsets because they have to run on every node.
```sh
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # these tolerations are to have the daemonset runnable on control plane nodes
      # remove them if your control plane nodes should not run pods
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      # it may be desirable to set a high priority class to ensure that a DaemonSet Pod
      # preempts running Pods
      # priorityClassName: important
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```
```sh
k create -f <filename>
kubectl get deamonsets 
kubectl describe deamonsets
kubectl get daemonsets --all-namespaces 
```
- How do you get pods to be scheduled on every node?
- one approach is to use the node name to bypass the scheduler and place a pod on a desired node.

# Labels and Selectors:
+ Labels are used as filters for ReplicaSet. Labels allow the rs to know what pod in the cluster or nodes 
placed under its management since there could be multiple pods running in the cluster.
+ The template definition section is required in every rs, even for pods that were created before the rs 
+ This is because if the pod fails and is to be recreated, it will need the spec to recreate it

- if you want to scale from 3 to 6 replicas, update the replicas to 6 and run
```sh
 kubectl replace -f <filename>
 kubectl scale --replicas=6 <filename>
 kubectl scale -- replicas replicaset name 
 kubectl edit pod/rs/rc/deploy <podname>
```

# Services, Load balancing and Networking:
+ Pods in a cluster have a unique cluster-wide IP, acting like separate VMs
+ Pods can communicate with all other Pods without NAT
+ Kubernetes service enables communication between various components within and outside the application and between other applications and users.
+ Services expose groups of Pods over a network.

 A full-stack web app typically has several pods such as frontend pods hosting a web server,
  the backend hosting the app, and pods hosting a db. Kubernetes service can group all pod groups  
  and provide a single backend to access the pods. you can create another service for all pods running the DB 
  These pods for different applications can therefore be scaled like microservices without impacting the other.
  A separate SVC for frontend, for backend, and db. 

For external communication,

the Kubernetes node has an IP, the host OS which is in the same network has an IP (private), and the pod has an ip but on  
a separate network
To access the application externally, the k8s service enables communication from pods on nodes

## TYPES:

## 1. *ClusterIP*:
+ This default Service type assigns an IP address from a pool of IP addresses that your cluster has reserved for that purpose. CIDR
+ You can specify your cluster IP address as part of a Service creation request
  - This type of service that allows communication between pods in a cluster is called cluster IP service.
```sh
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
```
kubectl create -f <filename>
kubectl get svc 

the service can be accessed by other pods in the cluster using the service name or ClusterIP

### 2. *NodePort*:
+ The k8s service maps a port on the Node to a port on the Pod(target)
+ The NodePort is a port range on the Node that gives external access. it ranges from 30000-32767
+ If you set the type field to NodePort, the Kubernetes control plane allocates a port from a range specified by --service-node-port-range flag (default: 30000-32767)

apiVersion: v1
Kind: Service
metadata: 
  name: myapp-svc
spec:
  type: NodePort
  ports:
    - targetPort: 80  # (port on Pod). it will assume port if not specified
      port: 80 # port on service. This is a mandatory field
      nodePort: 30008  #external node port on service 30000-32767. a random port will be allocated if not specified
  selector:
    app: myapp # This is the label that was used in the deployment metadata section
    type: front-end

kubectl create -f <filename>
kubectl get svc 
curl IP:30008

- In the case of multiple pods running the same application, you need to maintain the labels and selector section with
the same values. the service uses a random algorithm to route traffic to all pods with that same label.
- If the pods are running on different nodes in the cluster, you can access it by calling the IP of any node in the  
cluster. Service is a cluster-wide resource in k8s.

### 3. *LoadBalancer*:
On cloud providers that support external load balancers, setting the type field to LoadBalancer provisions a load balancer for your Service. 
The actual creation of the load balancer happens asynchronously, and information about the provisioned balancer is published in the Service's
K8s have native support for cloud platforms 
```sh
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
```

# STORAGE:
## VOLUMES:
Files in a container are ephemeral and they will be lost once the container fails or is stopped
The container is recreated in a clean state and all volumes I lost
+ Therefore persisting volumes are essential in k8s as they persistent volumes exist beyond the lifetime of a pod.
+ These persistent volumes can be a directory that can be used by Pods or shared by containers running in a Pod

## Types of volumes:
ConfigMaps:
===========
+ A ConfigMap provides a way to inject configuration data into pods.
+ This is a way of managing environmental variables in k8s. you can manually inject this variable by passing them as env.
+ But with many def files that require this variable, then you need to create them as a separate object in k8s 
and simply reference them in your object definition file. This can be done using ConfigMaps and Secrets.

ConfigMaps are used to pass configuration data in the form of key-value pairs in k8s and then injected into pods.
```sh
kubectl create configmap <ConfigName> --from-literal=APP_COLOR=blue \
                                   --from-literal=APP_MODE=prod  
                   OR 
kubectl create configmap app-config --from-file=<pathtofile>
```
```sh
APP_COLOR: blue 
APP_MODE: prod

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue 
  APP_MODE: prod
```
kubectl get configmaps


to inject the  env to the running container, add the envFrom section under the spec section  
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
      - containerPort: 8080
    envFrom:
      - configMapRef:
        name: app-config
```
to create a resource, use kubectl create -f <filename>

You can also ref a single env from a configmap 

env:
  - name: APP_COLOR
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_COLOR

You can also inject it as a volume

volumes:
- name: app-config-volume
  ConfigMap:
    name: app-config

emptyDir:
=========
+ An emptyDir volume is created when the Pod is assigned to a node, the emptyDir volume is initially empty
+ All containers in the Pod can read and write the same files in the emptyDir volume, 
+ that volume can be mounted at the same or different paths in each container. 
+ When a Pod is removed from a node for any reason, the data in the emptyDir is deleted permanently.

```sh
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi
```

hostPath:
==========
+ A hostPath volume mounts a file or directory from the host node's filesystem into your Pod. #It is risky
+ This is not something that most Pods will need, but it offers a powerful escape hatch for some applications.

```sh
# This manifest mounts /data/foo on the host as /foo inside the
# single container that runs within the hostpath-example-linux Pod.
#
# The mount into the container is read-only.
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-example-linux
spec:
  os: { name: linux }
  nodeSelector:
    kubernetes.io/os: linux
  containers:
  - name: example-container
    image: registry.k8s.io/test-webserver
    volumeMounts:
    - mountPath: /foo
      name: example-volume
      readOnly: true
  volumes:
  - name: example-volume
    # mount /data/foo, but only if that directory already exists
    hostPath:
      path: /data/foo # directory location on host
      type: Directory # This field is optional
```
SECRETS:
========
Secrets just like configmaps are used to store configuration data which can be injected into an object in k8s.
Unlike configmaps, secrets store sensitive data such as passwords and keys in an encoded manner.
You can create a secret imperatively by using:
```sh
  kubectl create secret generic <secretName> app-secret --from-literal=DB_Host=mysql   \
                                                        --from-literal=DB_User=root
```
    You can ref the secret from a file using the --from-file=app-secret
    For a declarative approach
```sh
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: mysql
  DB_User: root 
  DB_Password: password
```
It is however not advisable to pass your secrets in plain text as if defeats the entire purpose.
To convert the data from plaintext to an encoded format, on a Linux system, use the  
```sh 
echo -n 'mysql' | base64
echo -n 'root' | base64
echo -n 'password' | base64
```
```sh
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: sjhdvdv=
  DB_User: fnvjsf== 
  DB_Password: sffvnhri
```
copy the corresponding encoded values and replace and inject them into the file.

kubectl get secrets app-secret
kubectl describe secrets
kubectl get secrets app-secret -o yaml

to decode encoded values use the  
```sh
echo -n 'djvfjdo=' | base64 --decode
```
to inject encoded values into the pod object, use the 
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
      - containerPort: 8080
    envFrom:
      - secretRef:
        name: app-secret
```
Secrets are not encrypted but rather encoded and can be decoded using the same method. Therefore, do not upload your
secret files to the GitHub repo.

You can enable encryption at rest:
```sh
kubectl get secrets --all-namespaces -o jason | kubectl replace -f -
```
https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
## PersistentVolume (PV)
+ A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator
+ PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV
```sh
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
## PersistentVolumeClaim (PVC)
+ A PersistentVolumeClaim (PVC) is a request for storage by a user. 
+ Pods consume node resources and PVCs consume PV resources.
+ While Pods can request specific levels of resources (CPU and Memory), Claims can request specific size and access modes (e.g., they can be mounted ReadWriteOnce, ReadOnlyMany, ReadWriteMany, or ReadWriteOncePod,)
```sh
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
## Volume Lifecycle:
The reclaim policy for a PersistentVolume tells the cluster what to do with the volume after it has been released of its claim
+ Recycle
+ Reclaim
+ Delete

 # SECURITY:
Security in Kubernetes is mainly in the area of 
* Securing the configurable cluster components
* Securing the applications that run in the cluster

![4c](https://github.com/CHAFAH/DevOps_Setup/assets/125821852/eee2926b-6e4a-4e4a-ba70-36b6c2cc4fff)

 ## Authentication (Login)
+ All Kubernetes clusters have two categories of users: service accounts managed by Kubernetes, and normal users.
+ You can authenticate into the cluster by either
```sh
kubectl create ns dev
kubectl create sa development --ns dev
kubectl create role dev --ns dev
kubectl create roleBinding dev --role=dev --serviceAccount=development:dev
```

# HOW TO ISSUE A CERTIFICATE TO A USER IN THE CLUSTER:
+ A few steps are required to get a normal user to be able to authenticate and invoke an API. 
+ First, this user must have a certificate issued by the Kubernetes cluster, and then present that certificate to the Kubernetes API.

## 1. Create private key
The following scripts show how to generate PKI private key and CSR. It is important to set CN and O attribute of the CSR.
 CN is the name of the user and O is the group that this user will belong to. You can refer to RBAC for standard groups.
```sh
openssl genrsa -out myuser.key 2048
openssl req -new -key myuser.key -out myuser.csr -subj "/CN=myuser"
```
## 2. Create a CertificateSigningRequest
+ Create a CertificateSigningRequest and submit it to a Kubernetes Cluster via kubectl. Below is a script to generate the CertificateSigningRequest.
```sh
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  request: #(base64 encoded value of the CSR file content)
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - client auth
EOF
```
#Some points to note:

+ usages has to be 'client auth'

+ expirationSeconds could be made longer (i.e. 864000 for ten days) or shorter (i.e. 3600 for one hour)

+ request is the base64 encoded value of the CSR file content. You can get the content using this command:
```sh
cat myuser.csr | base64 | tr -d "\n"
```
## 3. Approve the CertificateSigningRequest
+ Use kubectl to create a CSR and approve it.

+ Get the list of CSRs:
```sh
kubectl get csr
```
## 4. Approve the CSR:
```sh
kubectl certificate approve myuser
```
## 5. Get the certificate

+ Retrieve the certificate from the CSR:
```sh
kubectl get csr/myuser -o yaml
```
+ The certificate value is in Base64-encoded format under status.certificate.

## 6. Export the issued certificate from the CertificateSigningRequest.
```sh
kubectl get csr myuser -o jsonpath='{.status.certificate}'| base64 -d > myuser.crt
```
## 7. Create Role and RoleBinding
+ With the certificate created it is time to define the Role and RoleBinding for this user to access Kubernetes cluster resources.

+ This is a sample command to create a Role for this new user:
```sh
kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods
```
+ This is a sample command to create a RoleBinding for this new user:
```sh
kubectl create rolebinding developer-binding-myuser --role=developer --user=myuser
```
## 8. Add to kubeconfig
+ The last step is to add this user into the kubeconfig file.

+ First, you need to add new credentials:
```sh
kubectl config set-credentials myuser --client-key=myuser.key --client-certificate=myuser.crt --embed-certs=true
```
## 9. Then, you need to add the context:
```sh
kubectl config set-context myuser --cluster=kubernetes --user=myuser
```
+ To test it, change the context to myuser:
```sh
kubectl config use-context myuser
```

+ If you install Kubernetes with kubeadm, most certificates are stored in /etc/kubernetes/pki.
  https://kubernetes.io/docs/setup/best-practices/certificates/
+ You can manage Kubeadm certificates here
  https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/
  
 ## Authorization
 + After being authenticated into a k8s cluster, all API requests made by an authenticated user in the cluster must be authorized by the API server against all objects and processes
 + Permissions in the cluster to make API calls are denied by default
 + The API request attributes being authorized are either the user, group, resource, verb, namespace path etc
### Authorization Modes
### Node
+ Grant special permissions to the kubelet based on the pods they are scheduled to run
### ABAC
https://kubernetes.io/docs/reference/access-authn-authz/abac/
+ Rights are granted to users through the use of policies that combine attributes.
+ The policies can use any type of attribute (user, resource, object, environment, etc)
#Examples
+ Alice can do anything to all resources:
```sh
{"apiVersion": "abac.authorization.kubernetes.io/v1beta1", "kind": "Policy", "spec": {"user": "alice", "namespace": "*", "resource": "*", "apiGroup": "*"}}
```
+ Bob can just read pods in namespace "projectCaribou":
```sh
{"apiVersion": "abac.authorization.kubernetes.io/v1beta1", "kind": "Policy", "spec": {"user": "bob", "namespace": "projectCaribou", "resource": "pods", "readonly": true}}
```
### RBAC
https://kubernetes.io/docs/reference/access-authn-authz/rbac/
+ This is a way of authorization where users in the cluster are permitted to perform actions based on their role in the company
+ This generally follows the principles of least privilege
+ To enable RBAC, start the apiserver with --authorization-mode=RBAC.
+ The RBAC permissions are granted in the form of Roles and ClsuterRoles which are additive i.e, there is not deny
+ A Role always sets permissions within a particular namespace; when you create a Role, you have to specify the namespace it belongs in.
+ ClusterRole, by contrast, is a non-namespaced resource.
+ A RoleBinding or ClusterRoleBinding is used to attach the ClusterRole or Role to the new user.

## *Role and RoleBinding Example:*
```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: dev
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
````

## *ClusterRole and ClusterRoleBinding Example:*
```sh
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]

---

apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
metadata:
  name: read-secrets-global
subjects:
- kind: Group
  name: manager # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

# NETWORKING:
  ===========
+ Linux Networking Basics:
  - Switching:
    to establish communication between two or more devices, an interface on each host is needed to connect them to the switch.
    Use the ip link command to see the interface.
    Assign an IP to the devices from the network 
ip addr add 192.168.1.10/24 dev eth0  

ping 192.168.1.10
- For a system in one network to communicate with another device in a different network, a router is needed.
- A router helps to connect networks.
- To add a route, check if a route exists as well as it must be configured in both networks if they intend to communicate 
route  
ip route add 192.168.2.0/24 via 192.168.1.1 

- The router gets two IPs  assigned to it to communicate between the two networks.
- A route
- For devices in these networks to communicate to the internet, add a new routing on the router for both networks
- You can also set a default router that routes r=traffic through that IP to the internet

DNS:
====
To communicate with different devices and different networks, instead of calling their IP addresses, you can assign a name   
to that IP which is called name resolution.

cat >> /etc/hosts
191.168.1.11   db

run the hostname command to see the name set to the device.
This DNS set on a system for another system is only known to the system and not to all other devices and networks  
Every run a command like an ssh db,   curl https//:db, it will look into the /etc/hosts file
because it grew complex and difficult to manage, it was moved to a DNS server where all entries can be added to and  
then when configured on the host, it will resolve the name from the entry in the DNS server.
- When an entry is added both on the host file and in the DNS server, the system refers first to the host file.
- the order can be changed to resolve the DNS server before the host file.

the .com   .org  .edu  .net .io 
These are top-level domains and they represent the intent of the websites.
www.google.com

www: is a subdomain # can have multiple subdomains
. : root
.com : top-level domain 

when a request is sent to apps.google.com
Org_DNS
root_DNS
.com_DNS
google_DNS
- Each time a request is sent, it goes through all these stages to get to the service. 
- the server will cache the server IP for some time to reduce the time it takes to reach the server
- you can also make an entry into the host file and use search and the domain name. 
 there are A name records that maps a name to an ipv4 address
- You can use nslookup OR dig to query a hostname

nslookup www.google.com 
it only queries names from the DNS.

NAMESPACES
==========
- when you run the ps aux in a container deployed in a namespace, it lists the PID of the container as isolated with PID=1 
- when u run the ps aux on the host, it lists several processes running in the system including the container process.
- You can create a container with a network NameSpace that shields the container from the network-related information of the host.

ip netns add red #to create a network namespace
ip netsn # to list the network ns
ip link  # to like the interfaces
ip netns exec red ip link  # to view interfaces within the namespace
ip -n red addr add 196.168.15.1 dev veth-red # to assign an ip to a namespace 

- to connect the red and the blue namespaces, run the 
ip link veth-red type veth peer name veth-blue
- to attach the two interfaces together, run the 
ip link set veth-red netns red  
- assign ip to the NameSpace
ip -n red addr add 196.168.15.1 dev veth-red
- bing up the interface using  
ip -n red lint set veth-red up  
- test connectivity using the 
ip netns exec red ping 192.168.15.2
- list the arp table on the red namespace
ip netns exec red arp  

To enable connectivity between multiple NameSpaces, you need a switch on the host server and
Use the Bridge network 
- Add a new interface and set it to bridge 
ip link add v-net-0 type bridge 
ip link # it will be status down so you need to bring it up
ip link set dev n-net-0 up
- to connect the NameSpaces to the network,  
ip link add veth-red type veth peer name veth-red-br
ip link add veth-blue type veth peer name  veth-blue-br

ip link set veth-red netns red 
ip link set veth-red-br master v-net-0
- assign ip addresses
 ip -n red addr add 192.168.15.2 dev veth-red
 ip -n red link set veth-red up

- Connection will not be established between the host and the NameSpace.
- to enable communication,  add the bridge network hosting the namespace to the host network
ip addr add 192.168.12.5/24 dev v-net-0

- These namespaces are not accessible over the internet
- A gateway needs to be added to the bridge network that allows connectivity. Since the local host has a gateway,
   we can add a route entry in the namespace to route traffic to the host ip.

DOCKER NETWORK:
==============
- When you create a docker container, you can specify a network for the container to run on.
docker run nginx --image=mginx --network none
- this container will not be accessible within or outside.
- with the host network, the container will be accessible by the host.
docker run --network host nginx.
if a port say 80 is open on the host, no other container can use that port.
- the third network is the bridge which has a default ip of 172.17.0.0
- when docker is installed, the bridge is created by default
docker network ls # ip link   # ip addr   # ip netns
docker creates an interface that attaches the container to the bridge network.
- the container is also assigned an IP  

Port Mapping:
-------------
  - Since the container is on a private network, it can only communicate to another container on the host and not be accessed externally.
  - But the host has an internet gateway that allows traffic to the internet.
  - Therefore, a port is opened on the host that allows traffic from the container to pass through the host.

  docker run -p 8080:80 nginx

  port 8080 is opened on the host, while port 80 is opened on the container
  - To access the container externally 
  curl http://192.168.1.10:8080
- Docker does this by creating a nat rule on the ip table.

iptables \
       -t nat \
       -A DOCKER \
       -j DNAT \
       --dport 8080 \
       --to-destination 172.17.0.3:80 
to list the rules, run  
iptables etcd-versionl -  t nat 


CONTAINER NETWORKING INTERFACE:
===============================

bridge add sf565f6s+6f /var/run/netns/sf565f6s+6f

this is an easier way to add a container to a container through a set standard of how containers should communicate.
CNI defines a plugin for how containers have to communicate.

# CLUSTER NETWORKING:
   ===================

- each node must have at least one interface connected to a network.
- Each network must have an address configured.
- the host must have a unique hostname and a unique mac address.
- some ports must be opened.
6443 : Kube-api
10250 : kubelet  # both master and worker nodes
10251 : kube-scheduler
10252 : kube-controller-manager
2379 : ETCD
30000-32767 : Services # on the worker nodes

ip address
ip address shows the type of bridge
ip route # lists all routes to the internet
netstat --help
netstat -npl | grep -i scheduler # to see the port the scheduler listens on| same for any k8s component

to see the number of established Connections,
netstat -npa | grep -i etcd | grep -i 2379 | we -1

# POD NETWORKING:
   ===============

- There is a network at the level of the nodes and also a network at the level of pods on the nodes to establish communication.
- It is not provided by default in k8s.
- K8s requires that every pod should have its IP addresses
- Every pod should be able to communicate with other pods on the same and other nodes without NAT.
- As long as this solution is implemented, the network will be established.
- there are many networking solutions such as WEAVE / FLANNERL / VMWARE NSX 
- https://www.weave.works/blog/weave-cloud-end-of-service
 https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
 Reference links: –

https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation
https://github.com/weaveworks/weave/releases

ls /etc/cni/net.d/  # to see the cni plugin used in the cluster

IPAM: Ip address management
the CNI (container network interface) is responsible for assigning IP addresses to pods in the k8s cluster

kubectl logs -n kube-system weave-net-mrks # to see the IP set to the pod 
kubectl exec <podname> -- ip route # to see the default route configured on the pod

# SERVICE NETWORKING:
   ===================
- Pods are rarely configured to communicate with each other directory. they make use of k8s services.
- When a service is created, it is accessible to all pods on the cluster. it is called ClusterIP.
- It is used for pods intended to only be accessed within the cluster.
- if the pod was intended to be accessed externally, a NodePort service will be used, it routes traffic to the internet
  through a port on the host node. it is also accessible within the cluster.
- the kubelet service on the worker nodes which is responsible for creating pods also monitors the changes in the cluster 
  kube-api server. Each time a new pod is to be created, the kubelet involves the CNI plugin to configure networking for the pod.
- the kube-proxy watched changes through the kube-apiserver, and every time a new service is created the kube-proxy is activated.
- Services are cluster-wide resources. A service is assigned an IP address from a predefined range which is used  
  by the kube-proxy. 
- the kube-proxy used the service IP address and created a forwarding rule in the cluster, any traffic coming to the IP of the  
  service is forwarded to the IP of the pod. 

k get pods -o wide 
k get svc 
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range  # to see the IP range for services in the cluster
iptables -L -t nat | grep <ServiceName>  # to see the routing rules
k logs -n kube-system <podname> # to see the proxy configured on the pod

# DNS RESOLUTION IN K8S:
  ======================

- k8s objects within a cluster can refer to each other by calling the ip addresses of the objects within thesame namespace.
  If an object is to communicate with another object on the dev namespace, the the name of the object withh be appended by  
  the name of the NameSpace.
  curl http://websetvice.dev # the last name of the service is the name of the namespace
- for each namespace, the dns sercer creates a subdomain for the cluster call dev.
- for all services in a cluster, the dns server creates another subdomain call svc 
- All services and pods are grouped for a root domain called local 

curl http://websetvice.dev.svc.cluster.local # fully qualified domain name for the service
- the same is done for pods, except that they are not given names but rather the . in the ip are changed to - 
Kubernetes implements a DNS server in the cluster called COREDNS,
- they are deployed as pods. they run the coreDNS executable.

## KUBERNETES INGRESS:
   ===================
Scenario:
  - You deploy a web app as a pod to host an online store called a store.
  - A database is also deployed to write data from the store app.
  - To make a database accessible to the web app, a ClusterIP service is destroyed for the db.
  - To make the webapp accessible to external users, a NodePort service is deployed say with port 38080
  - to access the app on the internet, http://<nodeIP>:38080
- Whenever traffic increases, we increase the number of pods and traffic is routed by the service
- With a pdt grade app, we do not want users to type the IP every time, so u configure a DNS to access using a name.
- The name will be something like http://my-store:38080.
- Since you don't want users to remember the nodePort either, you bring an additional layer btw dns and service and point the DNS  
  to the server.
- In the case of a cloud platform, create a service of type LoadBalancer. the cloud will deploy a LoadBalancer.
- the LB has an external IP that can be shared with users to access the app.

As the company grows you want to add a video app to the store but as a separate app on the same cluster.
- you create a service for the new app and assign a LoadBalancer. 
- To redirect traffic to a specific, you need to configure another proxy on the two existing LoadBalancers.
- each time a user types a specific URL, the proxy routes the traffic to the desired app.
- You also need to enable SSL for the app so that the user can access the app using https.
- it can be done at the app level or service level. It is a complex setup.
- It can all be managed on the k8s cluster just like an object.

- Ingress permits you to provide a single endpoint to access multiple applications as well as manage traffic routing between
  the defined URL path. It also configures ssl for http access. It is a layer 7 LoadBalancer in k8s.
- You still need to expose ingress either using NodePort or Cloud Native LoadBalancer.

- First, deploy a reverse proxy like nginx/haproxy   # INGRESS CONTROLLER
- specify a set of rules to specify ingress   # INGRESS RESOURCES

## Ingress Controller:
    ===================

They are created using a definition file. the cluster doesn't come with an ingress  controller.
- To deploy ingress, use any of  GCP HTTP LB  OR NGINX   # Supported and maintained by K8S
- They are not just LB, they have additional features 
- it is deployed using  

nginx-ingress.yaml
```sh
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args: 
            - /nginx-ingress-controller
            - --configmap=(POD_NAMESPACE) /nginx-configuration
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name 
            - name: POD_NAMESPACE 
              valueFrom:
                fieldRef: 
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
---
nginx-configmap.yaml

kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration

---
nginx-ingress-svc.yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: NodePort
  ports:
    - port:
      targetPort: 80
      protocol: TCP
      name: http 
    - port:
      targetPort: 443
      protocol: TCP
      name: https
    selector: nginx-ingress
---
nginx-service-account.yaml

kind: ServiceAccount
apiVersion: v1
metadata:
  name: nginx-ingress-serviceaccount
```

- the nginx program is stored within the image as /nginx-ingress-controller. it must be passed to start the nginx service
- to decouple this data from the nginx object, create a configmap object with no entries and pass it in
- Pass two env that carry the pod name and namespace to enable the nginx server to read the configuration data from the pod.
- specify the pods used by the ingress controller
- Create a service for the ingress controller
- Create a service account with roles and rolebinding for permissions for the ingress account to access and 
   manage the different app 

## Ingress Resources:
   ==================
- An ingress resource is a set of rules applied to the ingress controller.
- Here you can specify that if a user inputs a name, route them to the store etc.

ingress-wear.yaml
```sh
apiVersion: extensions/v1beta1
kind: Ingress 
metadata: nginx-wear
spec:
  backend:
    serviceName: wear-service   # the name of the service for the app
    servicePort: 80    # the service port.
```
k create ingress <IngressName> - <Namespace> --rule="/watch=ingress-service:8080 --rule"/wear=ingress-service:8080 "


k create -f <filename>
k get ingress  

- Rules are used when you want to route traffic based on different conditions, traafic originating for diff subdomains
- You can specify multiple rules depending on the number of subdomains existing on the app.

rule 1: www.my-online-store.com /waer   / watch 
rule2: www.wear.my-online-store.com     /returns  /support
rule3: www.watch.my-online-store.com 
rule4: anything els /movies  /tv

ingress resources for multiple rules:

  ingress-resource-rules.yaml
```sh
apiVersion: extensions/v1beta1
kind: Ingress 
metadata: nginx-wear
spec:
  rules:
  - http:
    paths:
    - path: /wear  
      backend:
        serviceName: wear-service
        servicePort: 80
    - path: /watch
      backend:
        serviceName: watch-service
        servicePort: 80
```
kubectl create ingress ingress-wear -n <Namespace> --rule="/wear=wear-service:8080"


k describe ingress ingress-resource-rules

- To route traffic based on domain names, add the hsot filed in the spec.
```sh
apiVersion: extensions/v1beta1
kind: Ingress 
metadata: nginx-wear
spec:
  rules:
  - host: wear.my-online-store.com
    http:
      paths:  
      - backend:
          serviceName: wear-service
          servicePort: 80
  - host: watch.my-online-store.com
    http:
      paths:  
      - backend:
          serviceName: watch-service
          servicePort: 80
```
    
Find more information and examples in the below reference link:-

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em- 

References:-

https://kubernetes.io/docs/concepts/services-networking/ingress

https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types    




# NAMESPACES:

  A namespace is simply a distinct working area in k8s where a defined set of resources rules and users can  
  be assigned to a namespace. 
- By default, a k8s cluster comes with a default namespace. Here, a user can provision resources.
- the subsystem namespace is also created by default for a set of pods and services for its functioning.
- The kubepublic is also created to host resources that are made available to the public.
- Within a cluster, you can create different namespaces for different projects and allocate resources to that namespace.
- Resources from the same namespaces can refer to each other by their names,
- they can also communicate with resources from another namespace by their names and append their namespace.
eg msql.connect("db-service.dev.svc.cluster.local")

kubectl get pods > will list only pods in the default namespace
kubectl get pods --namespace=kubesystem

kubectl apply -f <filename>  ==> will create object in the default namespace
kubectl create -f <filename> --namespace=kubesystem  ==> will create object in the kubesystem namespace

To ensure that your resources are always created in a specific namespace, add the namespace block in the resources
definition file
```sh
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
```

```sh
apiVersion: v1
Kind: NameSpace
metadata:
  name: dev
```
OR 
 kubectl create namespace dev
 kubectl get ns 

to set a namespace as the default namespace so that you don't always have to pass the NameSpace command, you need to set
set the namespace in the current context

kubectl config set-context $(kubectl config current-context) --namespace=dev
contexts are used to manage all resources in clusters from a single system 

To view resources in all NameSpaces use the 
kubectl get pods --all-namespaces

### Resource Quota:
To set a limit for resources in a namespace, create a resource-quota object in
```sh
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
```
kubectl apply -f <filename>

### Declarative and Imperative :
  these are different approaches to creating and managing infrastructure in IaC.
- The Imperative approach is giving just the required infrastructure and the resource figures out the steps involved.
eg kubectl create deployment nginx --image nginx
    kubectl set image deployment nginx nginx=nginx:1.18
    kubectl replace -f nginx.yaml

    it creates resources quickly but is difficult to edit or understand what is involved.

- in the Declarative approach, a step-by-step approach through writing configuration files
   Here we can write configuration files 
   Here, changes can be made as well as the resources can be versioned.
   resources can also be edited in the running state using the kubectl edit command
   The best practice is always to edit the configuration file and run the replace command rather than using the edit command.

   when you use the kubectl apply command, it will create objects that do not exist, and when you want to update the object,
   edit the yml file and run kubectl apply again and the object will pick up the latest changes in the file.
```sh
kubectl run --image=nginx nginx 
kubectl create deployment --image=nginx nginx 
kubectl edit deployment nginx 
kubectl scale deployment nginx --replicas=5
kubectl set image deployment nginx nginx=nginxv
```
```sh
kubectl run nginx --image=nginx --dry-run=client -o yaml    > nginx-deployment.yaml
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
k create deploy redis-deploy --image=redis --replicas=2 --namespace=dev-ns
kubectl run httpd --image=httpd:alpine --port=80 --expose   #will create pod and service
```
when you run a kubectl apply command, if the object stated in the file does not exist, it is created.
another live object configuration is created with additional fields and can be viewed using the  
- kubectl edit/describe object <objectName>
there is the last applied file that provides details about the last image of the live configuration.


SCHEDULING:
============

- there is a builtin scheduler in the cluster control-plane, that scans through nodes in the cluster and schedules
  pods on nodes based on several factors such as resources,
- But if you want to override and schedule your pods on specific nodes for some reason, you can do that by
  specifying the nodeName in the pod definition file.
- If a scheduler does not exist in the cluster, the pod will continually be pending.
- If you need a pod to run on a specific node, declare it at the time of creation.
- Kubernetes does not allow node modification after the pod has already been created.
- It can only be modified by creating a binding object setting the target to the NodeName and then sending a post 
 request to the pod's binding API.
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  -  image: nginx
     name: nginx
  nodeName: controlplane
```
RESOURCE REQUIREMENTS:
=======================
- every pod requires a set of resources to run.
when a pod is placed on a node, it consumes the resources on that node.
the scheduler determines the node a pod will be scheduled on based on resource availability.
if nodes have insufficient resources, the scheduler keeps the pod in a pending state.
- You can specify the resource requested by  a pod to run.
the scheduler will look for a node that has that resource specification and place that pod on it.
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources:
      requests:
        memory: "4Gi"
        cpu: 2
      limits:
        memory: 
        cpu: 
  nodeName: controlplane
```

- When a pod tries to exceed resources out of its limits, the system throttles the containers so that it doesn't
  use more than its limit,
- as for memory, a container can use more memory resources than its limit. but a pod cannot
- By default, k8s does not have a request and limit set, therefore resources can consume as much as they need.
- One pod can consume more and prevent others from running.
- When a CPU limit is set without request, k8s sets the request to the same as the limit
- When CPU requests and limits are set, then they stay within the range. But if one pod isn't consuming resources,
  then it is securing resources that other pods could use.
- When requests are set without limits, any pods can consume as many CPUs are required and when a pod needs more  
  resources, it has a guaranteed resource but without a limit. Make sure all pods have requests set.

LimitRanges as objects can be used to ensure that every pod created has some default values at the namespace level.
You can set it for both CPU and memory at the ns level. all pods will assume that standard.
ResourceQuota can also be used to set resource limits at the level of the NameSpace.

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



STATIC PODS:
============
- Without the controlplane which contains the API server, you can store your configuration files at the path 
  /etc/kubernetes/manifest,.
- kubernetes frequently visit this directory and any manifest file here to create pods will create the pod.
- it can create only pods, another object will need the controlplane
- The static pod folder can be any directory, but the path is set in the kubelet.service file in the kubeconfig.yaml
- Static pods can be viewed by running the docker ps command.
- The kubelet can create resources in k8s either by using configuration files or by listening to the kube API endpoints
  from the control controlplane.
- If you run the k get pods command, it will also list the static pods. This is blc a mirror of the static pods that are  
- created in the kubeapi but like other pods, can not be edited, except in the pod definition files.
- A use case for static pods is when you want to install components of the k8s control plane on every node, then you start 
  by installing the kubelet service and then create pod definition files that use docker images of all other 
  components of the controlplane and place them in the etc/kubernetes/manifest dir

kubectl get pods -n kube-system

Multiple Schedulers:
====================
You can deploy an optional scheduler added to the custom scheduler and configure it to schedule specific pods.
you can use a pod definition file or wget the scheduler binary and remane the scheduler to a different name.
to make sure that your object to be created is managed by that scheduler, you can add a scheduler option under
the spec section and pass the name of the scheduler.

kubectl logs object objectname

scheduling queu ===> priority sort
filtering       ===> NodeName/NodeUnschedulable/NodeReourceFit
scoring         ===> NodeReourceFit/ImageLocality
binding         ===> Defaultbinding

LOGGING AND MONITORING:
=======================
We can monitor the applications deployed in k8s as well as the Kubernetes cluster.
to monitor resources in the cluster, we can monitor
- node level metrics  #number of nodes, healthy, memory performance, CPU utilization
- pod level metric # number of pods and their CPU utilization

Kubernetes by default does not come with any monitoring agent but metric servers can be used and other resources  
such as Prometheus.
- You can have one metric server per cluster, the metric server retrieves metrics from pods and nodes  
 and stores them in memory. It does not store the metric in the disk so you can not store see historical metric.

You can clone the metric server from the github repo and run it. 
cluster performance can be seen by running  

kubectl top node 
kubectl top pods

managing application logs: 
  When you run a container in a pod, you can stream the logs by running the  
   kubectl logs -f <podname>
in the case of multi-container pods, you need to specify the name of the container individually.

APPLICATION LIFECYCLE MANAGEMENT:
=================================
## 1. Rolling Update and rollback:
  when you first create a deployment, it triggers a rollout which can be rev 1
  later when the image is updated, a new rollout is made called rev2
  to see the status of the rollout, run the  
  kubectl rollout status deployment/<deploymentname>

  to see the revision and the rollout history, run the
   kubectl rollout history deployment/myapp-deployment

there are two types of deployment strategies.

## 2. RECREATE:
- You can delete the existing deployment and then make a new deployment with the newer version
- This will lead to app downtime. this is not the k8s default strategy.
  
## 3. ROLLING UPDATE:
- Here, pods replicas are progressively destroyed and replaced with newer pods to ensure there is no downtime 
- This is the k8s default update strategy.

update can be done by changing the version of the image, replicas  
kubectl apply -f <filename>

it is advisable to manually edit the definition file than using the imperative approach because with an Imperative,
changes will not be saved in the definition file.

to rollback run the kubectl rollout undo deployment/myapp-deployment
```sh
kubectl apply -f deployment
kubectl get deployment
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/app 
```

Multi Container PODS:
=====================
the idea of decoupling a large mom=nolithic application into small components called microservices,
allows us to deploy a set of small independent and reusable code. This setup allows us to manage, or update only,
small portions of the app instead of the entire app. It might require running two apps or components in the same  
container. An example is a web server and a log agent deployed in the same container. They share the same lifecycle,
they are created and destroyed together, and they share the same network and the same volume of resources.
```sh
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: webapp
spec:
  containers:
  - image: nginx     # container1
    name: nginx
    ports:
      - containerPort: 8080
  - image: log-agent     #container2
    name: log-agent
```
https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/
    
to exec into a container in kubernetes, run the 
```sh
kubectl -n <Namespace> exec -it <ContainerName> -- cat /log/app.log
```
### InitContainers:

  these are also sidecar containers just like in a multicontainer pod. But they do not run constantly like the multi-containers
  Init containers are designed to run a particular process and then once the process is complete, they are exited.
  they may provide a service or run a script that starts a process in the main container and then exits.
```sh
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ;']
```
CLUSTER MAINTENANCE:
====================
This is important to know how and when to upgrade a cluster, and how to understand disaster recovery.
### 1. OS UPGRADE:
  To take down nodes in the cluster for updates or security patches on the node. When a node hosting the pods goes down,
  all the pods will not be accessible to users. Except there were replicas of that pod on another node.
  If the node lasts less than 5 minutes, the pods can be rescheduled, but if it exceeds 5 minutes, the controller will  
  consider it dead. If the pods were part of a ReplicaSet, they are recreated on other nodes depending on the nodeAffinity policies

  A quick update can be done when you are sure it will last less than 5 minutes, and if the pods on that node are part of a ReplicaSet.
  this will ensure that the application remains accessible.
  To safely do an upgrade on the nodes, you can drain the node for
```sh
  kubectl drain node-1 # --ignore-daemonsets 
  kubectl cordon node-2  # to make node unschedulable
  kubectl uncordon node-2 
```
  The node will be marked as unschedulable and pods will be gracefully terminated and recreated on other nodes.
  You can, path the nodes and make them available. You need to manually uncordon the node to make it schedulable.

  After a node is uncordon, it will require that pods are scheduled on it afresh, pods that were evicted during the drain 
  will not be automatically rescheduled.

  If a pod is not part of a ReplicaSet on a node, k8s internal security will not permit that node to be drained unless
  you manually delete the pod.
  nevertheless, you can use  --force flag to force delete the pod. This will permanently delete the pod on that node  
  and will not recreate it on another node because it was not part of a ReplicaSet.

### 2. Cluster Upgrade:
  Kubernetes is released in version and there are minor versions such as the alpha and beta versions before a more stable 
  release is made.
  None of the cluster components can be of a version higher than the API Server, except for the kubelet service.
  You can upgrade component by component.
  At any time, k8s supports only the latest three minor versions. It is good to upgrade your cluster before the version is unsupported.
  You should upgrade only one version higher at a time and not to the latest version if you were not using the previous one.

the upgrade depends on how the cluster is set up. between managed and self-managed clusters. managed clusters provided by the cloud is easier.
the cluster is being upgraded component by component and while the master node is being upgraded, the Kube API, and controllers,  
go down briefly. this does not affect the worker nodes.

To upgrade the worker nodes, if you do all of them at once, users will not be able to access the app. 
You can also upgrade one node at a time, which guarantees your pods are not all down. Or u use new nodes with newer
software and then move all pods?


https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
```sh
cat /etc/*release*  # to see the OS the nodes are using
kubeadm upgrade plan # to all the latest and stable versions
k drain controlplane --ignore-daemonsets
apt update
apt-cache madison kubeadm  # select the kubeadm version
apt-get upgrade -y kubeadm=1.12.0-00  # It has to be upgraded before the cluster components
```

to upgrade the cluster, use the 
```sh
kubeadm upgrade apply v1.12.0
```
the next step is to upgrade the kubelet.
the kubelet service must be upgraded manually
```sh
apt-get upgrade kubelete=v1.12.0-00
systemctl restart kubelet.service
```
### 3. Node Upgrade:

  https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linux-nodes/

  You need to first move all the workloads from the node to other nodes using the 
  ```sh
  kubectl drain node01 # This makes the node unschedulable
  apt-get upgrade -y kubeadm=v1.12.0-00
  apt-get upgrade -y kubelete=1.12.0-00
  kubeadm upgrade node config --kubelet-version v1.12.0
  systemctl restart kubelet
  kubectl uncordon node01
```

  perform the same steps to upgrade all other worker nodes.

  ssh <nodename>

BACKUP AND RESTORE:
===================
With respect to resources, declarative approaches can be used to save your configuration files. A good practice is to save
these codes in a source code repo like GitHub. But if an object is created imperatively, it will be difficult to keep track.
therefore, the KubeAPI server is a place to get all created resources.
All resource configurations are saved in the kube-apiserver

1. kubectl get all --all-namespaces -o yaml > all-deploy.yaml

there are tools like VELERO that can help in taking backups of the cluster.

The ETCD stores information about the state of the cluster.
So you can choose to backup the etcd cluster itself. it is found in the controlplane
data is stored in the data directory.
it also comes with the built-in snapshot utility

etcdctl snapshot save snapshot.db
etcdctl snapshot status snapshot.db


A snapshot directory is created. To restore the cluster from this backup, stop the kubeapi server and run the  

etcdctl snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup
systemctl daemon-reload
service kube-apiserver start

k logs etcd-controlplane -n kube-system | grep -i 'etcd-version'


ls /etc/kubernetes/manifest
k edit etcd.yaml
export ETCDCTL_API=3

ETCDCTL_API=3 etcdctl snapshot save --endpoint= \
--cacert= \
--cert= \
--key= \
/opt/snapshot-pre-boot.db  #location to save backup

to restore the original state of the cluster using the backup file, you can use the etcd restore <filename>


etcdctl snapshot restore --data-dir /var/lib/etcd-from-backup /opt/snapshot-pre-boot.db 

ls /var/lib/etcd-from-backup

vi /etc/kubernetes/manifest/etcd.yaml

edit the Hostpath and add /var/etcd-from-backup

