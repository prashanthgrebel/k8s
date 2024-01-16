  # k8s

1) Master Node: Manage, plan, schedule, and monitor the nodes

2) Worker Node: Host application as a container

3) ETCD Cluster: ETCD is a distributed reliable key-value store that is Simple, Secure & Fast

      1) ETCD - Commands 

      (Optional) Additional information about ETCDCTL Utility

        ETCDCTL is the CLI tool used to interact with ETCD.

      ETCDCTL can interact with ETCD Server using 2 API versions - Version 2 and Version 3.  By default its set to use Version 2. Each version has different sets of commands.

For example ETCDCTL version 2 supports the following commands:

    etcdctl backup
    etcdctl cluster-health
    etcdctl mk
    etcdctl mkdir
    etcdctl set


Whereas the commands are different in version 3

    etcdctl snapshot save 
    etcdctl endpoint health
    etcdctl get
    etcdctl put


To set the right version of API set the environment variable ETCDCTL_API command

export ETCDCTL_API=3


When API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don't work. When API version is set to version 3, version 2 commands listed above don't work.


Apart from that, you must also specify path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don't worry if this looks complex:

    --cacert /etc/kubernetes/pki/etcd/ca.crt     
    --cert /etc/kubernetes/pki/etcd/server.crt     
    --key /etc/kubernetes/pki/etcd/server.key


So for the commands I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form:


    kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key" 

4) Kube-scheduler: identifying  the right node to place a container,  based on node resources and capacity.

5) Kube-Controller Manager :
     1) Node controller: Responsible for onboarding New nodes to cluster, handling situations when a node failure 
     2) Replication controller: Responsible for  a number of containers running all the time
  

6) Kubep-apiserver: Responsible for orchestrating all operations within the cluster,  and exposing API to the external  to perform management operations monitor the cluster changes, and communicate with worker node and master.


7) kubelet: The kubelet is in charge of facilitating communication between the control plane and the node. The container runtime is in charge of pulling the relevant container image from a registry, unpacking containers, running them on the node, and communicating with the operating system kernel

8) Kube-proxy: enables the communication between the worker nodes containers


# Pod :-
   Kubernetes pod is a collection of one or more Linux® containers and is the smallest unit of a Kubernetes application
      
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end

spec: 
  containers:
    - name: nginx-myapp-pod
      image: nginx
```
# ReplicaSet :- 
  A ReplicaSet (RS) is a Kubernetes object used to maintain a stable set of replicated pods running within a cluster at any given time. A Kubernetes pod is a cluster deployment unit that typically contains one or more containers.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp_r
    type: front-end_r

spec:
  template:
    metadata:
      name: myapp-replicaset
      labels:
        app: myapp_r
        type: front-end_r
    spec:
      containers:
      - name: nginx-myapp-replicaset
        image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end_r
```
* # Labels and Selector:-
They are used to connect kubernetes services with a kubernetes pod. Labels are any key-value pairs that are used to identify that pod. The pod gets its label through the deployment which is like a blueprint for the pod before the pod is created. The Selector matches the label.
 ==> get pods that are connected with labels and selectors
 
``` # kubectl get pod --selector type=manual-sche-deployment-frontend```
 

# Deployments: 
A Kubernetes Deployment tells Kubernetes how to create or modify instances of the pods that hold a containerized application. Deployments can help to efficiently scale the number of replica pods, enable the rollout of updated code in a controlled manner, or roll back to an earlier deployment version if necessary

   # * Features: 
  * Rolling Update:
        Rolling updates allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones. The new Pods will be scheduled on Nodes with available resources.
  * Roll Back: 

![image](https://github.com/prashanthgrebel/k8s/assets/92351464/7c9fb08a-86ef-40c6-9d7d-3dd20429a957)


* Create an NGINX Pod

 ```kubectl run nginx --image=nginx```

* Generate POD Manifest YAML file (-o yaml). Don’t create it(–dry-run)

 ```kubectl run nginx --image=nginx --dry-run=client -o yaml```

* Create a deployment

``` kubectl create deployment --image=nginx nginx```

* Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run)

``` kubectl create deployment --image=nginx nginx --dry-run=client -o yaml```

* Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.

 ```kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml```

* Make necessary changes to the file (for example, adding more replicas) and then create the deployment.

 ```kubectl create -f nginx-deployment.yaml```

OR

* In k8s version 1.19+, we can specify the –replicas option to create a deployment with 4 replicas.

 ```kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml```




```
apiVersion: apps/v1
kind: Deployment 
metadata:
  name: myapp-replicaset
  labels:
    app: myapp_r
    type: front-end_r

spec:
  template:
    metadata:
      name: myapp-replicaset
      labels:
        app: myapp_r
        type: front-end_r
    spec:
      containers:
      - name: nginx-myapp-replicaset
        image: nginx
  replicas: 4
  selector:
    matchLabels:
      type: front-end_r



---
apiVersion: v1
kind: Service
metadata:
  name: deploy-service

spec:
  selector: 
      type: front-end_r
  type: NodePort
  ports:
    - nodePort: 30124
      port: 80
      targetPort: 80
```


# Service: 
  a Service is a method for exposing a network application that is running as one or more Pods in your cluster.

![image](https://github.com/prashanthgrebel/k8s/assets/92351464/9bba6256-bb12-479a-ac39-b24cb3df385a)


 *  ClusterIP
    Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default that is used if you don't explicitly specify a type for a Service. You can expose the Service to the public internet using an Ingress or a Gateway.
 *  NodePort
    Exposes the Service on each Node's IP at a static port (the NodePort). To make the node port available, Kubernetes sets up a cluster IP address, the same as if you had requested a Service of type: ClusterIP.
 *  LoadBalancer
    Exposes the Service externally using an external load balancer. Kubernetes does not directly offer a load balancing component; you must provide one, or you can integrate your Kubernetes cluster with a cloud provider.
  
# Scheduling: ![image](https://github.com/prashanthgrebel/k8s/assets/92351464/b4d7f2b9-45c1-4228-a055-61766099cbb3)


  # * Manual Scheduing: 
  We can manually assign the PODs to nodes yourself. Without a scheduler the easiest way to schedule a POD is to simply set the nodeName field to the name of the node in your POD specification file while creating the POD. The POD then gets assigned to the specified node.
  
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: manual-sche-deployment
  labels:
    app: manual-sche-deployment
    type: manual-sche-deployment-frontend 
spec:
  template:
    metadata:
      name: manual-sche-deployment
      labels:
        app: manual-sche-deployment
        type: manual-sche-deployment-frontend
    spec:
      containers:
      -  name: manual-sche-deployment-frontend-app
         image: nginx:latest
      nodeName: 1-117kworker
  replicas: 2
  selector:
    matchLabels:
      type: manual-sche-deployment-frontend
```

  # * Taints and tolerations: --Pending
  
  # * Node Selector: 
  The nodeSelector allows a Pod to schedule only on nodes that have a label(s) identical to the label(s) in the nodeSelector. These are key-value pairs that can be defined within the PodSpec.

  * Step: 1
    1) Label the nodes
    ``` # kubectl label nodes 1-117kworker size=Large ```
    ``` # kubectl get nodes --show-labels ```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-selector
  labels:
    app: node-selector-app
    type: node-selector-app-frontend


spec:
  template:
    metadata:
      name: node-selector
      labels:
        app: node-selector-app
        type: node-selector-app-frontend
    spec:
      nodeSelector:
        size: Large
      containers:
      - name: node-selector-app-frontend
        image: registry.prashanthgr.private:5000/nginx_custom:latest
  replicas: 3
  selector:
    matchLabels:
      type: node-selector-app-frontend
```
 # * Node Affinity: --- Pending
 
 # * Resource Limits:-
   Kubernetes defines limits as a maximum amount of a resource to be used by a container. This means that the container can never consume more than the memory amount or CPU amount indicated. resources: limits: cpu: 0.5 memory: 100Mi. 
```
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: app
    image: registry.prashanthgr.private:5000/nginx_custom:latest
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```
# DeamonSets:

A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.

Some typical uses of a DaemonSet are:

 * running a cluster storage daemon on every node
 * running a logs collection daemon on every node
 * running a node monitoring daemon on every node
