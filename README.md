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
    type: front-end_R

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


      
