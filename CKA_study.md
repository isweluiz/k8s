# Kubernetes key points 
- [Certificate Management with kubeadm](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/)
- [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
- [Share a Cluster with Namespaces](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/)
- [DNS label](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names)
- [Restoring etcd cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)
- [Upgrade kubeadm cluster](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [Kubectl](https://kubernetes.io/docs/reference/kubectl/)
- [Kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Object management](https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/)
- [Kubectl book](https://kubectl.docs.kubernetes.io/references/)
- [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
- [Tools for Monitoring Resources](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/)
- [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Resource requests and limits of Pod and container](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-requests-and-limits-of-pod-and-container)
- [Container probes](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
]

# Kubernetes extra links
- [CKA exam practice](https://www.youtube.com/watch?v=KfrZd9YCftU)
- [Monitoring chalenges](https://www.youtube.com/watch?v=eHdotsFBb0c)
- [Roles - rbac](https://octopus.com/blog/k8s-rbac-roles-and-bindings)
- [Kubernetes Debugging](https://www.containiq.com/post/debugging-kubernetes-nodes-in-not-ready-state#:~:text=The%20kubectl%20get%20nodes%20command,the%20state%20of%20your%20nodes.&text=A%20node%20with%20a%20NotReady,it%20doesn't%20lie%20unused.)
- [Commands tips](https://jamesdefabia.github.io/docs/user-guide/kubectl/kubectl_get/)
- [21 Resources and Tutorials to Learn Kubernetes](https://medium.com/@asad_5112/21-resources-and-tutorials-to-learn-kubernetes-ec7949b7a401)
- [Killer coda](https://killercoda.com/)
- [CKAD Exercises](https://github.com/dgkanatsios/CKAD-exercises)
- [CKA exam series](https://codeburst.io/kubernetes-ckad-weekly-challenges-overview-and-tips-7282b36a2681)
- [K8s hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
- [Dumps](https://free-braindumps.com/linux-foundation/free-cka-braindumps.html?p=2)
- [Dumps2](https://xcerts.com/linux-foundation/cka--exam-questions-and-answers.html)


## Comands

kubectl config view -o json | jq '.clusters[].name'
"kubernetes-dev"


## What is Kubernetes?
https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/
Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.

The name Kubernetes originates from Greek, meaning helmsman or pilot. K8s as an abbreviation results from counting the eight letters between the "K" and the "s". Google open-sourced the Kubernetes project in 2014. Kubernetes combines over 15 years of Google's experience running production workloads at scale with best-of-breed ideas and practices from the community.


## Why you need Kubernetes and what it can do


# Install Kubernetes Cluster using kubeadm
Follow this documentation to set up a Kubernetes cluster on __Ubuntu 20.04 LTS__.

This documentation guides you in setting up a cluster with one master node and one worker node.

## Assumptions
|Role|FQDN|IP|OS|RAM|CPU|
|----|----|----|----|----|----|
|Master|kmaster.example.com|172.16.16.100|Ubuntu 20.04|2G|2|
|Worker|kworker.example.com|172.16.16.101|Ubuntu 20.04|1G|1|

## On both Kmaster and Kworker
##### Login as `root` user
```
sudo su -
```
Perform all the commands as root user unless otherwise specified
##### Disable Firewall
```
ufw disable
```
##### Disable swap
```
swapoff -a; sed -i '/swap/d' /etc/fstab
```
##### Update sysctl settings for Kubernetes networking
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
##### Install docker engine
```
{
  apt install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  apt update
  apt install -y docker-ce=5:19.03.10~3-0~ubuntu-focal containerd.io
}
```
### Kubernetes Setup
##### Add Apt repository
```
{
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
}
```
##### Install Kubernetes components
```
apt update && apt install -y kubeadm=1.23.0-00 kubelet=1.23.0-00 kubectl=1.23.0-00
```
##### In case you are using LXC containers for Kubernetes nodes
Hack required to provision K8s v1.15+ in LXC containers
```
{
  mknod /dev/kmsg c 1 11
  echo '#!/bin/sh -e' >> /etc/rc.local
  echo 'mknod /dev/kmsg c 1 11' >> /etc/rc.local
  chmod +x /etc/rc.local
}
```

## On kmaster
##### Initialize Kubernetes Cluster
Update the below command with the ip address of kmaster
```
kubeadm init --apiserver-advertise-address=172.16.16.100 --pod-network-cidr=192.168.0.0/16  --ignore-preflight-errors=all
```
##### Deploy Calico network
```
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml

```

##### Cluster join command
```
kubeadm token create --print-join-command
```

##### To be able to run kubectl commands as non-root user
If you want to be able to run kubectl commands as non-root user, then as a non-root user perform these
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## On Kworker
##### Join the cluster
Use the output from __kubeadm token create__ command in previous step from the master server and run here.

## Verifying the cluster (On kmaster)
##### Get Nodes status
```
kubectl get nodes
```
##### Get component status
```
kubectl get cs
```

Have Fun!!



 https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/


cat << EOF | /etc/hosts
192.168.99.10 master
192.168.99.11 worker1
192.168.99.12 worker2
192.168.99.13 worker3
192.168.99.110 k8s

modprobe br_netfilter

 cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system

swapoff -a 

apt-get update && apt-get install containerd -y 

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list


mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml 
systemctl restart containerd 

1.23.0-00


sudo apt-get update -y
sudo apt-get install -y kubelet=1.23.0-00 kubeadm=1.23.0-00 kubectl=1.23.0-00
sudo apt-mark hold kubelet kubeadm kubectl

kubeadm init --apiserver-advertise-address="192.168.99.110" --pod-network-cidr="192.168.0.0/16" --ignore-preflight-errors=all --kubernetes-version 1.23.0


kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f


  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

sudo kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml


kubeadm join 192.168.99.10:6443 --token j9xxda.9k96vrggp1sdohc7 \
        --discovery-token-ca-cert-hash sha256:6ead0cd0ab221631544a53d1f38e900ab49c8160e5fee747e971316e0f53aaf0 


### Safely Draining a K8s Node
- [Official documentation](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)

You can use kubectl drain to safely evict all of your pods from a node before you perform maintenance on the node (e.g. kernel upgrade, hardware maintenance, etc.). Safe evictions allow the pod's containers to gracefully terminate and will respect the PodDisruptionBudgets you have specified.

Note: By default kubectl drain ignores certain system pods on the node that cannot be killed; see the kubectl drain documentation for more details.

When kubectl drain returns successfully, that indicates that all of the pods (except the ones excluded as described in the previous paragraph) have been safely evicted (respecting the desired graceful termination period, and respecting the PodDisruptionBudget you have defined). It is then safe to bring down the node by powering down its physical machine or, if running on a cloud platform, deleting its virtual machine.


```bash
kubectl drain <node> --ignore-daemonsets 

root@k8s:/opt/k8s# k drain worker2 --ignore-daemonsets --force
node/worker2 cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-54q2q, kube-system/kube-proxy-4cmzj
evicting pod default/my-deployment-76b6f467d9-vzt67
pod/my-deployment-76b6f467d9-vzt67 evicted
node/worker2 drained
```

To return the node 

```bash
kubectl uncordon <node name>
```

```bash
k get pods -o wide --all-namespaces
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: my-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-deployment
  template:
    metadata:
      labels:
        app: my-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

```bash
root@k8s:/opt/k8s# kubectl get pods  -o wide 
NAME                             READY   STATUS    RESTARTS   AGE     IP                NODE      NOMINATED NODE   READINESS GATES
my-deployment-76b6f467d9-h98c6   1/1     Running   0          6m10s   192.168.235.130   worker1   <none>           <none>
my-deployment-76b6f467d9-k4s92   1/1     Running   0          15m     192.168.235.129   worker1   <none>           <none>
```

```bash
root@k8s:/opt/k8s# k uncordon worker2
node/worker2 uncordoned
root@k8s:/opt/k8s# kubectl get nodes
NAME      STATUS   ROLES                  AGE   VERSION
k8s       Ready    control-plane,master   13d   v1.23.4
worker1   Ready    <none>                 13d   v1.23.0
worker2   Ready    <none>                 13d   v1.23.0
```

### Celan up 
```bash
k get deployment
kubectl delete deployment my-deployment
```
## Draining multiple nodes in parallel
The kubectl drain command should only be issued to a single node at a time. However, you can run multiple kubectl drain commands for different nodes in parallel, in different terminals or in the background. Multiple drain commands running concurrently will still respect the PodDisruptionBudget you specify.

For example, if you have a StatefulSet with three replicas and have set a PodDisruptionBudget for that set specifying minAvailable: 2, kubectl drain only evicts a pod from the StatefulSet if all three replicas pods are ready; if then you issue multiple drain commands in parallel, Kubernetes respects the PodDisruptionBudget and ensure that only 1 (calculated as replicas - minAvailable) Pod is unavailable at any given time. Any drains that would cause the number of ready replicas to fall below the specified budget are blocked.

### Upgrading kubeadm clusters
```
apt-cache madison kubeadm

apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.23.4-00 && \
apt-mark hold kubeadm
```
Verify that the download works and has the expected version:
```
kubeadm version
```

### Worker node update 
The upgrade procedure on worker nodes should be executed one node at a time or few nodes at a time, without compromising the minimum required capacity for running your workloads.

#### Upgrade kubeadm
# replace x in 1.23.x-00 with the latest patch version
```
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.23.4-00 && \
apt-mark hold kubeadm

apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.23.4-00 kubectl=1.23.4-00 && \
apt-mark hold kubelet kubectl
```

### Drain the node 
Prepare the node for maintenance by marking it unschedulable and evicting the workloads:

#### replace <node-to-drain> with the name of your node you are draining
```
kubectl drain <node-to-drain> --ignore-daemonsets
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

#### Uncordon the node 
Bring the node back online by marking it schedulable:

####  replace <node-to-drain> with the name of your node
kubectl uncordon <node-to-drain>

```bash
root@k8s:/opt/k8s# k get nodes
NAME      STATUS   ROLES                  AGE   VERSION
k8s       Ready    control-plane,master   13d   v1.23.4
worker1   Ready    <none>                 13d   v1.23.4
worker2   Ready    <none>                 13d   v1.23.4
```

### Verify the status of the cluster 
After the kubelet is upgraded on all nodes verify that all nodes are available again by running the following command from anywhere kubectl can access the cluster:

```
kubectl get nodes
```
The STATUS column should show Ready for all your nodes, and the version number should be updated.


# ETCD 
etcd is a strongly consistent, distributed key-value store that provides a reliable way to store data that needs to be accessed by a distributed system or cluster of machines. It gracefully handles leader elections during network partitions and can tolerate machine failure, even in the leader node

[Documentation](https://etcd.io/docs/v3.5/)

## Backing up an etcd cluster and Restoring
You are working for BeeBox, a subscription service company that provides weekly shipments of bees to customers. The company is using Kubernetes to run some of their applications, and they want to make sure their Kubernetes infrastructure is robust and able to recover from failures.

Your task is to establish a backup and restore process for the Kubernetes cluster data. Back up the cluster's etcd data, then restore it to verify the process works.

You can find certificates to authenticate with etcd in /home/cloud_user/etcd-certs.


Look up the value for the key cluster.name in the etcd cluster. The value should be beebox
```
cloud_user@etcd-1:~$ ETCDCTL_API=3 etcdctl get cluster.name \
--endpoints=https://10.0.1.101:2379 \
--cacert=/home/cloud_user/etcd-certs/etcd-server.crt \
--cert=/home/cloud_user/etcd-certs/etcd-server.crt \
--key=/home/cloud_user/etcd-certs/etcd-server.key

cluster.name
beebox
```

Below is an example for taking a snapshot of the keyspace served by $ENDPOINT to the file snapshotdb:

ETCDCTL_API=3 etcdctl --endpoints $ENDPOINT snapshot save snapshotdb
```
ETCDCTL_API=3 etcdctl snapshot save /home/cloud_user/etcd_backup.db \
--endpoints=https://10.0.1.101:2379 \
--cacert=/home/cloud_user/etcd-certs/etcd-server.crt \
--cert=/home/cloud_user/etcd-certs/etcd-server.crt \
--key=/home/cloud_user/etcd-certs/etcd-server.key

```

Verify the snapshot:

ETCDCTL_API=3 etcdctl --write-out=table snapshot status /home/cloud_user/etcd_backup.db

```
cloud_user@etcd-1:~$ ETCDCTL_API=3 etcdctl --write-out=table snapshot status /home/cloud_user/etcd_backup.db
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 68ae77f5 |        2 |          5 |      20 kB |
+----------+----------+------------+------------+
```

Reset etcd by removing all existing etcd data.

```
sudo systemctl stop etcd
sudo rm -rf /var/lib/etcd
```

Restore the etcd data from the backup. This command spins up a temporary etcd cluster, saving the data from the backup file
to a new data directory (in the same location where the previous data directory was).

```
sudo ETCDCTL_API=3 etcdctl snapshot restore /home/cloud_user/etcd_backup.db \
--initial-cluster etcd-restore=https://10.0.1.101:2380 \
--initial-advertise-peer-urls https://10.0.1.101:2380 \
--name etcd-restore \
--data-dir /var/lib/etcd
```

Set ownership on the new data directory.

```
sudo chown -R etcd:etcd /var/lib/etcd
```

Start etcd.
```
sudo systemctl start etcd
```

Verify that the restored data is present by looking up the value for the key cluster.name again. The value should be beebox .


## Troubleshooting




## NAMESPACES 

In Kubernetes, namespaces provides a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc).

Namespaces are intended for use in environments with many users spread across multiple teams, or projects. For clusters with a few to tens of users, you should not need to create or think about namespaces at all. Start using namespaces when you need the features they provide.

Namespaces provide a scope for names. Names of resources need to be unique within a namespace, but not across namespaces. Namespaces cannot be nested inside one another and each Kubernetes resource can only be in one namespace.

*** Namespaces are a way to divide cluster resources between multiple users (via resource quota).

It is not necessary to use multiple namespaces to separate slightly different resources, such as different versions of the same software: use labels to distinguish resources within the same namespace.

- Review
- - namespaces resources and quotas

```
kubectl get namespaces
```
Kubernetes starts with four initial namespaces:

default The default namespace for objects with no other namespace
- kube-system The namespace for objects created by the Kubernetes system
- kube-public This namespace is created automatically and is readable by all users (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement.
- kube-node-lease This namespace holds Lease objects associated with each node. Node leases allow the kubelet to send heartbeats so that the control plane can detect node failure.



To see which Kubernetes resources are and aren't in a namespace:

# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false


----------
## Questions about kubernetes management 
What software does Kubernetes use to store data about the state of the cluster?
- etcd 

Which tool(s) allow you to create Kubernetes clusters?
- kubeadm
- minikube

How can you make Kubernetes highly available?
- having multiple control planes nodes

Which tool provides a command-line interface for Kubernetes?
- kubectl 

Which command is used to safely evict your pods from a node before maintenance on the node?
- kubectl drain

Which command allows you to upgrade control plane components?
- kubectl upgrade apply (This command will upgrade the control plane.)

Which of the following are options for a highly-available Etcd architecture?
- Stacked Etcd
- External Etcd

Which command-line tool allows you to interact with Etcd and perform backups?
- etcdctl

Which tool can help you perform a Kubernetes upgrade?
- kubeadm

What command can you use to allow Pods to be scheduled on a previously-drained node after Node maintenance is complete?
- kubectl uncordon



## Kubernetes Object Management
CHAPTER 4
Kubernetes provides a command line tool for communicating with a Kubernetes cluster's control plane, using the Kubernetes API.
This tool is named kubectl.
For configuration, kubectl looks for a file named config in the $HOME/.kube directory. You can specify other kubeconfig files by setting the KUBECONFIG environment variable or by setting the --kubeconfig flag.

#### kubectl Tips
- [Kubectl](https://kubernetes.io/docs/reference/kubectl/)
- [Kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Object management](https://kubernetes.io/docs/concepts/overview/working-with-objects/object-management/)

### Kubectl context and configuration 
Set which Kubernetes cluster kubectl communicates with and modifies configuration information. See Authenticating Across Clusters with kubeconfig documentation for detailed config file information.

kubectl config view # Show Merged kubeconfig settings.

# use multiple kubeconfig files at the same time and view merged config
KUBECONFIG=~/.kube/config:~/.kube/kubconfig2 

kubectl config view

kubectl config view -o jsonpath='{.users[].name}'    # display the first user
kubectl config view -o jsonpath='{.users[*].name}'   # get a list of users
kubectl config get-contexts                          # display list of contexts 
kubectl config current-context                       # display the current-context
kubectl config use-context my-cluster-name           # set the default context to my-cluster-name

# add a new user to your kubeconf that supports basic auth
kubectl config set-credentials kubeuser/foo.kubernetes.com --username=kubeuser --password=kubepassword

# permanently save the namespace for all subsequent kubectl commands in that context.
kubectl config set-context --current --namespace=ggckad-s2

# set a context utilizing a specific username and namespace.
kubectl config set-context gce --user=cluster-admin --namespace=foo \
  && kubectl config use-context gce

kubectl config unset users.foo                       # delete user foo

# short alias to set/show context/namespace (only works for bash and bash-compatible shells, current context to be set before using kn to set namespace) 
alias kx='f() { [ "$1" ] && kubectl config use-context $1 || kubectl config current-context ; } ; f'
alias kn='f() { [ "$1" ] && kubectl config set-context --current --namespace $1 || kubectl config view --minify | grep namespace | cut -d" " -f6 ; } ; f'


### Pod basic

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ['sh', '-c', 'while true; do sleep 3600; done']
```

## Using RBAC Authorization
https://kubernetes.io/docs/reference/access-authn-authz/rbac/

Role-based access control (RBAC) is a method of regulating access to computer or network resources based on the roles of individual users within your organization.

RBAC authorization uses the rbac.authorization.k8s.io API group to drive authorization decisions, allowing you to dynamically configure policies through the Kubernetes API.

### RoleBinding and ClusterRoleBinding

A role binding grants the permissions defined in a role to a user or set of users. It holds a list of subjects (users, groups, or service accounts), and a reference to the role being granted. A RoleBinding grants permissions within a specific namespace whereas a ClusterRoleBinding grants that access cluster-wide.


Question
```
You are working for BeeBox, a company that provides regular shipments of bees to customers. The company is in the process of building a Kubernetes-based infrastructure for some of their software.

Your developers frequently request that you provide information from the Kubernetes cluster, so you would like to give them the ability to read data from the cluster but not make any changes to it. Using Kubernetes role-based access control, ensure the dev user can read pod metadata and container logs from any pod in the beebox-mobile namespace.

A kubeconfig file for the dev user has already been created on the server. You can use this file to test your RBAC setup as the dev user like so:

kubectl get pods -n beebox-mobile --kubeconfig /home/cloud_user/dev-k8s-config

---
- Create a Role for the `dev` User
- Create a role called pod-reader. Provide it with read access to pods and container logs in the beebox-mobile namespace.

Bind the Role to the `dev` User and Verify Your Setup Works
Create a RoleBinding to bind the pod-reader role to the dev user. Interact with pods in the beebox-mobile namespace to make sure you can read pod metadata and container logs as the dev user but not make any changes.

```

## Configure Service Accounts for Pods
A service account provides an identity for processes that run in a Pod.
When you (a human) access the cluster (for example, using kubectl), you are authenticated by the apiserver as a particular User Account (currently this is usually admin, unless your cluster administrator has customized your cluster). Processes in containers inside pods can also contact the apiserver. When they do, they are authenticated as a particular Service Account (for example, default).



https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

## Tools for Monitoring Resources
https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/

To scale an application and provide a reliable service, you need to understand how the application behaves when it is deployed. You can examine application performance in a Kubernetes cluster by examining the containers, pods, services, and the characteristics of the overall cluster. Kubernetes provides detailed information about an application's resource usage at each of these levels. This information allows you to evaluate your application's performance and where bottlenecks can be removed to improve overall performance.

In Kubernetes, application monitoring does not depend on a single monitoring solution. On new clusters, you can use resource metrics or full metrics pipelines to collect monitoring statistics.

kubectl top po -n beebox-mobile  --sort-by=cpu --selector=app=auth > /home/cloud_user/cpu-pod-name.txt



## Pods and Containers
CHAPTER 5

### ConfigMaps
https://kubernetes.io/docs/concepts/configuration/configmap/
A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

A ConfigMap allows you to decouple environment-specific configuration from your container images, so that your applications are easily portable.

Caution: ConfigMap does not provide secrecy or encryption. If the data you want to store are confidential, use a Secret rather than a ConfigMap, or use additional (third party) tools to keep your data private.

A ConfigMap is not designed to hold large chunks of data. The data stored in a ConfigMap cannot exceed 1 MiB. If you need to store settings that are larger than this limit, you may want to consider mounting a volume or use a separate database or file service.

### Secrets
https://kubernetes.io/docs/concepts/configuration/secret/
A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a Pod specification or in a container image. Using a Secret means that you don't need to include confidential data in your application code.

Because Secrets can be created independently of the Pods that use them, there is less risk of the Secret (and its data) being exposed during the workflow of creating, viewing, and editing Pods. Kubernetes, and applications that run in your cluster, can also take additional precautions with Secrets, such as avoiding writing secret data to nonvolatile storage.

Secrets are similar to ConfigMaps but are specifically intended to hold confidential data

https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

```
httpasswd -c user 
k create secret generic nginx-htpasswd --from-file .htpasswd
rm -rf .htpasswd 

k get secret/mysecret2 -o json | jq '.data.secret'
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.19.1
    ports: 
    - containerPort: 80
    volumeMounts:
    - name: config-volume
      mountPath: "/etc/nginx"
    - name: htpasswd-volume
      mountPath: "/etc/nginx/config"
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config
  - name: htpasswd-volume
    secret: 
      secretName: nginx-htpasswd
```


### Resource requests and limits of Pod and container 
https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-requests-and-limits-of-pod-and-container
When you specify a Pod, you can optionally specify how much of each resource a container needs. The most common resources to specify are CPU and memory (RAM); there are others.

When you specify the resource request for containers in a Pod, the kube-scheduler uses this information to decide which node to place the Pod on. When you specify a resource limit for a container, the kubelet enforces those limits so that the running container is not allowed to use more of that resource than the limit you set. The kubelet also reserves at least the request amount of that system resource specifically for that container to use.

### Container probes 
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes
A probe is a diagnostic performed periodically by the kubelet on a container. To perform a diagnostic, the kubelet either executes code within the container, or makes a network request.

Status for Pod readiness
The kubectl patch command does not support patching object status. To set these status.conditions for the pod, applications and operators should use the PATCH action. You can use a Kubernetes client library to write code that sets custom Pod conditions for Pod readiness.

For a Pod that uses custom conditions, that Pod is evaluated to be ready only when both the following statements apply:

All containers in the Pod are ready.
All conditions specified in readinessGates are True.
When a Pod's containers are Ready but at least one custom condition is missing or False, the kubelet sets the Pod's condition to ContainersReady.

### Pod Lifecycle
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
This page describes the lifecycle of a Pod. Pods follow a defined lifecycle, starting in the Pending phase, moving through Running if at least one of its primary containers starts OK, and then through either the Succeeded or Failed phases depending on whether any container in the Pod terminated in failure.

Whilst a Pod is running, the kubelet is able to restart containers to handle some kind of faults. Within a Pod, Kubernetes tracks different container states and determines what action to take to make the Pod healthy again.

In the Kubernetes API, Pods have both a specification and an actual status. The status for a Pod object consists of a set of Pod conditions. You can also inject custom readiness information into the condition data for a Pod, if that is useful to your application.

Pods are only scheduled once in their lifetime. Once a Pod is scheduled (assigned) to a Node, the Pod runs on that Node until it stops or is terminated.

### Container restart policy 
https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

The spec of a Pod has a restartPolicy field with possible values Always, OnFailure, and Never. The default value is Always.

The restartPolicy applies to all containers in the Pod. restartPolicy only refers to restarts of the containers by the kubelet on the same node. After containers in a Pod exit, the kubelet restarts them with an exponential back-off delay (10s, 20s, 40s, …), that is capped at five minutes. Once a container has executed for 10 minutes without any problems, the kubelet resets the restart backoff timer for that container.

Pod conditions 
A Pod has a PodStatus, which has an array of PodConditions through which the Pod has or has not passed:

PodScheduled: the Pod has been scheduled to a node.
ContainersReady: all containers in the Pod are ready.
Initialized: all init containers have completed successfully.
Ready: the Pod is able to serve requests and should be added to the load balancing pools of all matching Services.


```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3

```


## Creating Multi-Container Pods
https://kubernetes.io/docs/concepts/workloads/pods/#using-pods
https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/
https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: two-containers
spec:

  restartPolicy: Never

  volumes:
  - name: shared-data
    emptyDir: {}

  containers:

  - name: nginx-container
    image: nginx
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html

  - name: debian-container
    image: debian
    volumeMounts:
    - name: shared-data
      mountPath: /pod-data
    command: ["/bin/sh"]
    args: ["-c", "echo Hello from the debian container > /pod-data/index.html"]

```


## Init Containers
https://kubernetes.io/docs/concepts/workloads/pods/init-containers/
You can specify init containers in the Pod specification alongside the containers array (which describes app containers).

A Pod can have multiple containers running apps within it, but it can also have one or more init containers, which are run before the app containers are started.

Init containers are exactly like regular containers, except:

Init containers always run to completion.
Each init container must complete successfully before the next one starts.
If a Pod's init container fails, the kubelet repeatedly restarts that init container until it succeeds. However, if the Pod has a restartPolicy of Never, and an init container fails during startup of that Pod, Kubernetes treats the overall Pod as failed.


This example defines a simple Pod that has two init containers. The first waits for myservice, and the second waits for mydb. Once both init containers complete, the Pod runs the app container from its spec section.
```yaml
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
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```


## DaemonSet
https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
DaemonSets are a great way to ensure a pod replica runs dynamically on each node. They even automatically handle the creation and removal of such pods when nodes join or leave the cluster. In this lab, you will have the opportunity to practice your skills with DaemonSets by using them to run pods on all nodes in an existing cluster.

A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.

Some typical uses of a DaemonSet are:

running a cluster storage daemon on every node
running a logs collection daemon on every node
running a node monitoring daemon on every node
In a simple case, one DaemonSet, covering all nodes, would be used for each type of daemon. A more complex setup might use multiple DaemonSets for a single type of daemon, but with different flags and/or different memory and cpu requests for different hardware types.

```yaml
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
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
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
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## Create static Pods
https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/
Static Pods are managed directly by the kubelet daemon on a specific node, without the API server observing them. Unlike Pods that are managed by the control plane (for example, a Deployment); instead, the kubelet watches each static Pod (and restarts it if it fails).

Static Pods are always bound to one Kubelet on a specific node.

The kubelet automatically tries to create a mirror Pod on the Kubernetes API server for each static Pod. This means that the Pods running on a node are visible on the API server, but cannot be controlled from there. The Pod names will be suffixed with the node hostname with a leading hyphen.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-pod-diagnostic
spec:
  containers:
    - name: static-pod-disgnostic01
      image: acgorg/beebox-diagnostic:1
```


## Deployments
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
A Deployment provides declarative updates for Pods and ReplicaSets.

You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

k edit deployment <deployment name>
k  rollout status deployment.v1.apps/beebox-web
k scale deployment --replicas=30 beebox-web

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
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

## Network Plugins
https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/
There is a pod called maintenance in the foo namespace. Create a NetworkPolicy that blocks all traffic to and from this pod.
```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all-foo-maintence
  namespace: foo
spec:
  podSelector:
    matchLabels:
      app: maintenance 
  policyTypes:
  - Ingress
  - Egress
```

There are some pods in the users-backend namespace. Create a NetworkPolicy that blocks all traffic to pods in this namespace, except for traffic from pods in the same namespace on port 80.
```yaml
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: network-policy-users-backend
  namespace: users-backend
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          network-policy: "true"
    ports:
    - protocol: TCP
      port: 80
```

## Persistent Volumes
https://kubernetes.io/docs/concepts/storage/persistent-volumes/
A PersistentVolume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. It is a resource in the cluster just like a node is a cluster resource. PVs are volume plugins like Volumes, but have a lifecycle independent of any individual Pod that uses the PV. This API object captures the details of the implementation of the storage, be that NFS, iSCSI, or a cloud-provider-specific storage system.

A PersistentVolumeClaim (PVC) is a request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources. Pods can request specific levels of resources (CPU and Memory). Claims can request specific size and access modes (e.g., they can be mounted ReadWriteOnce, ReadOnlyMany or ReadWriteMany, see AccessModes).

While PersistentVolumeClaims allow a user to consume abstract storage resources, it is common that users need PersistentVolumes with varying properties, such as performance, for different problems. Cluster administrators need to be able to offer a variety of PersistentVolumes that differ in more ways than size and access modes, without exposing users to the details of how those volumes are implemented. For these needs, there is the StorageClass resource.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: host-pv
spec:
  storageClassName: localdisk
  persistentVolumeReclaimPolicy: Recycle
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /var/output

---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: localdisk
provisioner: kubernetes.io/no-provisioner
allowVolumeExpansion: true    

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  storageClassName: localdisk
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi

---
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "while true; do echo CKA APPROVED! >> /output/success; sleep5; done"]
      volumeMounts:
        - name: pv-storage
          mountPath: /output
   volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: myclaim
```
## Services
https://kubernetes.io/docs/concepts/services-networking/service/

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:11.14.2
    ports:
      - containerPort: 80
        name: http-web-svc
        
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    targetPort: http-web-svc
```


## Volumes
https://kubernetes.io/docs/concepts/storage/volumes/
On-disk files in a container are ephemeral, which presents some problems for non-trivial applications when running in containers. One problem is the loss of files when a container crashes. The kubelet restarts the container but with a clean state. A second problem occurs when sharing files between containers running together in a Pod. The Kubernetes volume abstraction solves both of these problems. Familiarity with Pods is suggested.

- Emptydir 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

- hostPath 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
      # this field is optional
      type: Directory
```      

- persistent volume node affinit

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: local-storage
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - example-node
```          

## Scheduling - NodeSelector NodeName 
- [Kubernetes Scheduler](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/)
- [Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)
- [Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
create label: 
```
k label nodes k8s-worker2 external-auth-services=true 

nodeSelector:
  external-auth-services: "true"
```

OR 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
```

### Exercices 



Get pod with more CPU usage | second line with sed
k top po -n web --sort-by=cpu -l app=auth | awk '{print $1}' | sed -sn 2p > /k8s/0003/cpu-pod.txt

List pods with specific service account after create role and role-binding
k get pods -n web --as=system:serviceaccount:web:webautomation


ETCDCTL_API=3 etcdctl --endpoints 10.0.1.102:2379 \
  --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
  --key=/home/cloud_user/etcd-certs/etcd-server.key \
  --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
  member list

ETCDCTL_API=3 etcdctl --endpoints 10.0.1.102:2379 \
  --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
  --key=/home/cloud_user/etcd-certs/etcd-server.key \
  --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
   snapshot save /home/cloud_user/etcd_backup.db


ETCDCTL_API=3 etcdctl --write-out=table snapshot status /home/cloud_user/etcd_backup.db

ETCDCTL_API=3 etcdctl --endpoints 10.0.1.102:2379 \
  --cert=/home/cloud_user/etcd-certs/etcd-server.crt \
  --key=/home/cloud_user/etcd-certs/etcd-server.key \
  --cacert=/home/cloud_user/etcd-certs/etcd-ca.pem \
   snapshot restore /home/cloud_user/etcd_backup.db
+

---
# replace x in 1.23.x-00 with the latest patch version
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.22.2-00 && \
apt-mark hold kubeadm

apt-mark unhold kubelet kubectl && \
apt-get update && apt-get install -y kubelet=1.22.2-00 kubectl=1.22.2-00 && \
apt-mark hold kubelet kubectl

kubectl uncordon <node-to-drain>


# replace x in 1.23.x-00 with the latest patch version
apt-mark unhold kubeadm && \
apt-get update && apt-get install -y kubeadm=1.23.x-00 && \
apt-mark hold kubeadm


--------------
### Some commands 
kill.sh 

- Uncordon Drain
k uncordon $(k get nodes | awk '{print $1}' | sed -sn 4p )
k get all --selector env=prod --no-headers | wc -l

- Get contexts without header 
kubectl config get-contexts --no-headers --output=name
k config get-contexts | awk '{print $2}' | grep -vi name

- Get pods without labels 
k get pods --show-labels 

- Create service expose po
k expose po <name-pod> --name <name-svc> --port <port-expose> --target-port <pod-port>
k expose deployment <deployment-name> --name <name-svc> --port <port-expose> --target-port <pod-port> --typoe=NodePort|ClusterIp -------Update manually the node port nodePort: 


k uncordon $(k get nodes | awk '{print $1}' | sed -sn 4p )
k get all --selector env=prod --no-headers | wc -l
k get po --selector env=prod,bu=finance,tier=frontend


kubectl create service clusterip clusterip-nginx
--namespace=modulo3
--tcp=8080:80
--dry-run=client
-o yaml
| kubectl set selector "tier=backend" --local -f -
-o yaml

get json file item
k get nodes -o json | jq -r '.items[].status.nodeInfo.osImage' > /opt/outputs/nodes_os_x43kj56.txt


Untaint node 
kubectl taint nodes controlplane node-role.kubernetes.io/master:NoSchedule-

## Taints and Tolerations
https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/
Node affinity is a property of Pods that attracts them to a set of nodes (either as a preference or a hard requirement). Taints are the opposite -- they allow a node to repel a set of pods.

Tolerations are applied to pods, and allow (but do not require) the pods to schedule onto nodes with matching taints.

Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints.

Concepts
You add a taint to a node using kubectl taint. For example,

kubectl taint nodes node1 key1=value1:NoSchedule

places a taint on node node1. The taint has key key1, value value1, and taint effect NoSchedule. This means that no pod will be able to schedule onto node1 unless it has a matching toleration.

To remove the taint added by the command above, you can run:

kubectl taint nodes node1 key1=value1:NoSchedule-
You specify a toleration for a pod in the PodSpec. Both of the following tolerations "match" the taint created by the kubectl taint line above, and thus a pod with either toleration would be able to schedule onto node1:

## Json path
https://kubernetes.io/docs/reference/kubectl/jsonpath/


kubectl get pods -o json
kubectl get pods -o=jsonpath='{@}'
kubectl get pods -o=jsonpath='{.items[0]}'
kubectl get pods -o=jsonpath='{.items[0].metadata.name}'
kubectl get pods -o=jsonpath="{.items[*]['metadata.name', 'status.capacity']}"
kubectl get pods -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.startTime}{"\n"}{end}'


## Certification get expirate date 
openssl x509  -noout -text -in /etc/kubernetes/pki/apiserver.crt | grep Validity -A2
 kubeadm certs check-expiration | grep apiserver
