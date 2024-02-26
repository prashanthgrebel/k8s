  # k8s

[Kubernetes-CKA-0100-Core-Concepts-1.pdf](https://github.com/prashanthgrebel/k8s/files/14151675/Kubernetes-CKA-0100-Core-Concepts-1.pdf)


   * Helm: https://phoenixnap.com/kb/install-helm

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

#  Imperative Commands with Kubectl: - 


While you would be working mostly the declarative way – using definition files, imperative commands can help in getting one time tasks done quickly, as well as generate a definition template easily. This would help save considerable amount of time during your exams.

Before we begin, familiarize with the two options that can come in handy while working with the below commands:

--dry-run: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the --dry-run=client option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.

-o yaml: This will output the resource definition in YAML format on screen.

 

Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.

 
POD

Create an NGINX Pod

```kubectl run nginx --image=nginx```

 

Generate POD Manifest YAML file (-o yaml). Don’t create it(–dry-run)

```kubectl run nginx --image=nginx --dry-run=client -o yaml```

 
Deployment

Create a deployment

```kubectl create deployment --image=nginx nginx```

 

Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run)

```kubectl create deployment --image=nginx nginx --dry-run=client -o yaml```

 

Generate Deployment with 4 Replicas

```kubectl create deployment nginx --image=nginx --replicas=4```

 

You can also scale a deployment using the kubectl scale command.

```kubectl scale deployment nginx --replicas=4 ```

Another way to do this is to save the YAML definition to a file and modify

```kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml```

 

You can then update the YAML file with the replicas or any other field before creating the deployment.

 
Service

Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

```kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml```

(This will automatically use the pod’s labels as selectors)

Or

```kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml``` (This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)

 

Create a Service named nginx of type NodePort to expose pod nginx’s port 80 on port 30080 on the nodes:

```kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml```

(This will automatically use the pod’s labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

```kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml```

(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.
Reference:

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands

https://kubernetes.io/docs/reference/kubectl/conventions/



# Kind and apiVersions of all components

Kind  | apiVersion
------------- | -------------
Pod           | v1
ReplicaSet    | apps/v1
Deployment    | apps/v1
StatefulSet   | apps/v1
Service       | v1
DaemonSet     | apps/v1
Job           | batch/v1
CronJob       | batch/v1
ReplicationController | v1
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
  
  
[Kubernetes-CKA-0200-Scheduling-1.pdf](https://github.com/prashanthgrebel/k8s/files/14256914/Kubernetes-CKA-0200-Scheduling-1.pdf)
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

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-app
  labels:
    name: daemonset-app
    type: daemonset-app-frontend

spec:
  template:
    metadata:
      name: daemonset-app
      labels:
        name: daemonset-app
        type: daemonset-app-frontend
    spec:
      containers:
      - name: daemonset-app-fe
        image: registry.prashanthgr.private:5000/nginx_custom:latest
  selector:
    matchLabels:
      type: daemonset-app-frontend

```

# Static Pods:
  Static Pods are managed directly by the kubelet daemon on a specific node, without the API server observing them. Unlike Pods that are managed by the control plane (for example, a Deployment); instead, the kubelet watches each static Pod (and restarts it if it fails).




# #############################################################################################################################################################################################################################################



# Application life cycle management:- 
[Kubernetes-CKA-0400-Application-Lifecycle-Management-1.pdf](https://github.com/prashanthgrebel/k8s/files/14163119/Kubernetes-CKA-0400-Application-Lifecycle-Management-1.pdf)


 

# * Rolling Updates and Rollbacks  - Deployments

   * Rollout and varsioning
   ``` Summarize Commands
> kubectl rollout status deployment/myapp-deployment
> kubectl rollout history deployment/myapp-deployment
> kubectl create –f deployment-definition.yml
> kubectl get deployments
> kubectl apply –f deployment-definition.yml
> kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
> kubectl rollout undo deployment/myapp-deployment
```
# Commands and Arguments ConfigMaps and Secrets:-

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-redisdev

data:
  redis.conf: |
    bind 0.0.0.0 
    port 6380
    pidfile /var/run/redis_6380.pid
    logfile /var/log/redis.log
    tcp-keepalive 0
    loglevel notice
    dbfilename dump.rdb
    dir /var/lib/redis-data
    maxmemory 1gb
    maxclients 10000
    requirepass Rebel@9902
```
```
apiVersion: apps/v1
kind: Deployment
metadata: 
  name: redisdev
  labels:
    tier: redisdev-db
spec:
  template:
    metadata:
      name: redisdev
      labels:
        tier: redisdev-db
    spec:
      containers:
      - name: redisdev
        image: redis
        command: ["/bin/bash","-c"]
        args:
          - echo creating data dir '/var/lib/redis-data';
            mkdir /var/lib/redis-data;
            echo running redis server with port 6380;
            redis-server /data/redis.conf;
        volumeMounts:
        - name: redis-conf
          mountPath: "./data"
      volumes: 
      - name: redis-conf
        configMap:
          name: configmap-redisdev

  replicas: 2
  selector:
    matchLabels: 
      tier: redisdev-db
```
# * A note on Secrets

Remember that secrets encode data in base64 format. Anyone with the base64 encoded secret can easily decode it. As such the secrets can be considered not very safe.

The concept of safety of the Secrets is a bit confusing in Kubernetes. The kubernetes documentation page and a lot of blogs out there refer to secrets as a “safer option” to store sensitive data. They are safer than storing in plain text as they reduce the risk of accidentally exposing passwords and other sensitive data. In my opinion it’s not the secret itself that is safe, it is the practices around it.

Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:

    Not checking-in secret object definition files to source code repositories.
    Enabling Encryption at Rest for Secrets so they are stored encrypted in ETCD.

Also the way kubernetes handles secrets. Such as:

    A secret is only sent to a node if a pod on that node requires it.
    Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
    Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.

Read about the protections and risks of using secrets here

Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, HashiCorp Vault. I hope to make a lecture on these in the future.



  # * Managing Secrets using kubectl
  * Use raw data
```
kubectl create secret generic db-user-pass \
    --from-literal=username=admin \
    --from-literal=password='S!B\*d$zDsb='
```

 * Use source files

    Store the credentials in files:
```
    echo -n 'admin' > ./username.txt
    echo -n 'S!B\*d$zDsb=' > ./password.txt
```
   The -n flag ensures that the generated files do not have an extra newline character at the end of the text. This is important because when kubectl reads a file and encodes the content into a base64 string, the extra newline character gets encoded too. You do not need to escape special characters in strings that you include in a file.

   Pass the file paths in the kubectl command:
```
    kubectl create secret generic db-user-pass \
        --from-file=./username.txt \
        --from-file=./password.txt
```
   The default key name is the file name. You can optionally set the key name using --from-file=[key=]source. For example:
```
    kubectl create secret generic db-user-pass \
        --from-file=username=./username.txt \
        --from-file=password=./password.txt
```
  # * Managing Secrets using Configuration File
  1. Convert the strings to base64:
     ```
     echo -n 'admin' | base64
     echo -n '1f2d1e2e67df' | base64
     ```

   The output is similar to:
   ```
    YWRtaW4=
    MWYyZDFlMmU2N2Rm
  ```
   2. Create the manifest:

```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

# Cluster Maintenance : --

  # * Node Maintainance
  1. Drain Node safely from the cluster so pods are moved to other nodes with an issue

```
     kubectl drain <node name>
 ```
 2. After maintainance make node schedulable
```
     kubectl uncordon <node name>
```

== ``` kubectl cordon <node name>``` -- allows you to mark a node as “unschedulable,” which means that — until told otherwise — the existing pods will continue to run, but no new ones will be created on that node

 # * Cluster maintainance / upgrade clusters versions

   Follow  official kubernetes docs
  
  https://v1-27.docs.kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

# * Backup and Restore Methods :-
  * 1. Backup resource configs.
     ```
     kubectl get all --all-namespaces -o yaml > all-spaces-configs.yml
     ```
  * 2. Backup-ETCD Cluster.
       1. Built-in snapshot
          ```
          ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT snapshot save snapshot.db
          ```
          OR Important
        ```
        ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
          --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
          snapshot save <backup-file-location>
        ```
          Verify the snapshot:
          ```
          ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db
          ```

  * 3. Restoring an etcd cluster :
        1. stop kubeapi service
        ```
        systemctl stop kube-apiserver
        ```
        2. Restore
        ```
        ETCDCTL_API=3 etcdctl --data-dir <data-dir-location> snapshot restore snapshot.db
        ```

        3. make changes in etcd manifest file of etcd volume host path  with newly backedup dir
        4. restart etcd server
           ```
           systemctl daemon-reload
           systemctl restart etcd
           ```
        5. start kube-apiserver
           ```
           systemctl start kube-apiserver
           ````
       # * WORKING WITH ETCDCTL

etcdctl is a command line client for etcd.

 

In all our Kubernetes Hands-on labs, the ETCD key-value database is deployed as a static pod on the master. The version used is v3.

To make use of etcdctl for tasks such as back up and restore, make sure that you set the ETCDCTL_API to 3.

 

You can do this by exporting the variable ETCDCTL_API prior to using the etcdctl client. This can be done as follows:
```
export ETCDCTL_API=3
```
On the Master Node:

 

To see all the options for a specific sub-command, make use of the -h or –help flag.

 

For example, if you want to take a snapshot of etcd, use:

etcdctl snapshot save -h and keep a note of the mandatory global options.

Since our ETCD database is TLS-Enabled, the following options are mandatory:

–cacert                verify certificates of TLS-enabled secure servers using this CA bundle

–cert                    identify secure client using this TLS certificate file

–endpoints=[127.0.0.1:2379] This is the default as ETCD is running on master node and exposed on localhost 2379.

–key                  identify secure client using this TLS key file


# Storage:= 


# * Persistentvolume:
  A persistent volume is a piece of storage in a cluster that an administrator has provisioned. It is a resource in the cluster, just as a node is a cluster resource. A persistent volume is a volume plug-in that has a lifecycle independent of any individual pod that uses the persistent volume.

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-ngix
  labels:
    name: pv-ngix
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Mi
  hostPath:
    path: /tmp/pv-ngix

  
```
* Persistent volume claim:-
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-ngix-claim

spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 900Ki
  selector:
    matchLabels:
       name: pv-ngix
```


# Networking

* # Ingress

  ![image](https://github.com/prashanthgrebel/k8s/assets/92351464/3c19a476-fb93-43a4-8f6e-0166059cb3a1)
![image](https://github.com/prashanthgrebel/k8s/assets/92351464/1a3c5409-0ef0-448c-894c-09aff8c89477)
![image](https://github.com/prashanthgrebel/k8s/assets/92351464/8aeed843-a5d7-4fd4-8ce0-d85289fb0576)


* # Ingress-controller

```
apiVersion: v1
kind: Namespace
metadata:
  labels:
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  name: ingress-nginx
---
apiVersion: v1
automountServiceAccountToken: true
kind: ServiceAccount
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx
  namespace: ingress-nginx
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx-admission
  namespace: ingress-nginx
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx
  namespace: ingress-nginx
rules:
- apiGroups:
  - ""
  resources:
  - namespaces
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - configmaps
  - pods
  - secrets
  - endpoints
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - networking.k8s.io
  resources:
  - ingresses
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - networking.k8s.io
  resources:
  - ingresses/status
  verbs:
  - update
- apiGroups:
  - networking.k8s.io
  resources:
  - ingressclasses
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resourceNames:
  - ingress-controller-leader
  resources:
  - configmaps
  verbs:
  - get
  - update
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - create
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
  - patch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx-admission
  namespace: ingress-nginx
rules:
- apiGroups:
  - ""
  resources:
  - secrets
  verbs:
  - get
  - create
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx
rules:
- apiGroups:
  - ""
  resources:
  - configmaps
  - endpoints
  - nodes
  - pods
  - secrets
  - namespaces
  verbs:
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - networking.k8s.io
  resources:
  - ingresses
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
  - patch
- apiGroups:
  - networking.k8s.io
  resources:
  - ingresses/status
  verbs:
  - update
- apiGroups:
  - networking.k8s.io
  resources:
  - ingressclasses
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx-admission
rules:
- apiGroups:
  - admissionregistration.k8s.io
  resources:
  - validatingwebhookconfigurations
  verbs:
  - get
  - update
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx
  namespace: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: ingress-nginx
subjects:
- kind: ServiceAccount
  name: ingress-nginx
  namespace: ingress-nginx
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx-admission
  namespace: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: ingress-nginx-admission
subjects:
- kind: ServiceAccount
  name: ingress-nginx-admission
  namespace: ingress-nginx
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: ingress-nginx
subjects:
- kind: ServiceAccount
  name: ingress-nginx
  namespace: ingress-nginx
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx-admission
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: ingress-nginx-admission
subjects:
- kind: ServiceAccount
  name: ingress-nginx-admission
  namespace: ingress-nginx
---
apiVersion: v1
data:
  allow-snippet-annotations: "true"
kind: ConfigMap
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx-controller
  namespace: ingress-nginx
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
        # externalTrafficPolicy: Local
  ports:
  - appProtocol: http
    name: http
    port: 80
    protocol: TCP
    targetPort: http
  - appProtocol: https
    name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx-controller-admission
  namespace: ingress-nginx
spec:
  ports:
  - appProtocol: https
    name: https-webhook
    port: 443
    targetPort: webhook
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  minReadySeconds: 0
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app.kubernetes.io/component: controller
      app.kubernetes.io/instance: ingress-nginx
      app.kubernetes.io/name: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/component: controller
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/name: ingress-nginx
    spec:
      containers:
      - args:
        - /nginx-ingress-controller
        - --publish-service=$(POD_NAMESPACE)/ingress-nginx-controller
        - --election-id=ingress-controller-leader
        - --controller-class=k8s.io/ingress-nginx
        - --ingress-class=nginx
        - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
        - --validating-webhook=:8443
        - --validating-webhook-certificate=/usr/local/certificates/cert
        - --validating-webhook-key=/usr/local/certificates/key
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: LD_PRELOAD
          value: /usr/local/lib/libmimalloc.so
        image: k8s.gcr.io/ingress-nginx/controller:v1.2.1@sha256:5516d103a9c2ecc4f026efbd4b40662ce22dc1f824fb129ed121460aaa5c47f8
        imagePullPolicy: IfNotPresent
        lifecycle:
          preStop:
            exec:
              command:
              - /wait-shutdown
        livenessProbe:
          failureThreshold: 5
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        name: controller
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        - containerPort: 443
          name: https
          protocol: TCP
        - containerPort: 8443
          name: webhook
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /healthz
            port: 10254
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        resources:
          requests:
            cpu: 100m
            memory: 90Mi
        securityContext:
          allowPrivilegeEscalation: true
          capabilities:
            add:
            - NET_BIND_SERVICE
            drop:
            - ALL
          runAsUser: 101
        volumeMounts:
        - mountPath: /usr/local/certificates/
          name: webhook-cert
          readOnly: true
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/os: linux
      serviceAccountName: ingress-nginx
      terminationGracePeriodSeconds: 300
      volumes:
      - name: webhook-cert
        secret:
          secretName: ingress-nginx-admission
---
apiVersion: batch/v1
kind: Job
metadata:
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx-admission-create
  namespace: ingress-nginx
spec:
  template:
    metadata:
      labels:
        app.kubernetes.io/component: admission-webhook
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
        app.kubernetes.io/version: 1.2.1
      name: ingress-nginx-admission-create
    spec:
      containers:
      - args:
        - create
        - --host=ingress-nginx-controller-admission,ingress-nginx-controller-admission.$(POD_NAMESPACE).svc
        - --namespace=$(POD_NAMESPACE)
        - --secret-name=ingress-nginx-admission
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        image: k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1@sha256:64d8c73dca984af206adf9d6d7e46aa550362b1d7a01f3a0a91b20cc67868660
        imagePullPolicy: IfNotPresent
        name: create
        securityContext:
          allowPrivilegeEscalation: false
      nodeSelector:
        kubernetes.io/os: linux
      restartPolicy: OnFailure
      securityContext:
        fsGroup: 2000
        runAsNonRoot: true
        runAsUser: 2000
      serviceAccountName: ingress-nginx-admission
---
apiVersion: batch/v1
kind: Job
metadata:
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx-admission-patch
  namespace: ingress-nginx
spec:
  template:
    metadata:
      labels:
        app.kubernetes.io/component: admission-webhook
        app.kubernetes.io/instance: ingress-nginx
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
        app.kubernetes.io/version: 1.2.1
      name: ingress-nginx-admission-patch
    spec:
      containers:
      - args:
        - patch
        - --webhook-name=ingress-nginx-admission
        - --namespace=$(POD_NAMESPACE)
        - --patch-mutating=false
        - --secret-name=ingress-nginx-admission
        - --patch-failure-policy=Fail
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        image: k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1@sha256:64d8c73dca984af206adf9d6d7e46aa550362b1d7a01f3a0a91b20cc67868660
        imagePullPolicy: IfNotPresent
        name: patch
        securityContext:
          allowPrivilegeEscalation: false
      nodeSelector:
        kubernetes.io/os: linux
      restartPolicy: OnFailure
      securityContext:
        fsGroup: 2000
        runAsNonRoot: true
        runAsUser: 2000
      serviceAccountName: ingress-nginx-admission
---
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: nginx
spec:
  controller: k8s.io/ingress-nginx
---
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  labels:
    app.kubernetes.io/component: admission-webhook
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.2.1
  name: ingress-nginx-admission
webhooks:
- admissionReviewVersions:
  - v1
  clientConfig:
    service:
      name: ingress-nginx-controller-admission
      namespace: ingress-nginx
      path: /networking/v1/ingresses
  failurePolicy: Fail
  matchPolicy: Equivalent
  name: validate.nginx.ingress.kubernetes.io
  rules:
  - apiGroups:
    - networking.k8s.io
    apiVersions:
    - v1
    operations:
    - CREATE
    - UPDATE
    resources:
    - ingresses
  sideEffects: None
```





  

