## Installation and configuration of Kubernetes on centos 7
### install ntp for Synchronization
## disabled Selinux
## start and enable ntpd
## update Hosts files in such away that every host can be ping by internal ip address
<!-- -->
172.31.45.153	gkdangal2 
<!-- -->
172.31.36.209	gkdangal3 
<!-- -->
172.31.120.7	gkdangal4 
<!-- -->
172.31.25.230	gkdangal5 
<!-- -->
## disable ip-tables if you are using centos 6
## Disable firewalld and stop on centos 7
## Disable selinux on both centos 7 and centos 6
### set up repo
<!-- -->
vim /etc/yum.repos.d/virt7-docker-common-release.repo
<!-- -->
[virt7-docker-common-release]
name=virt7-docker-common-release
<!-- -->
baseurl=http://cbs.centos.org/repos/virt7-docker-common-release/x86_64/os/
<!-- -->
gpgcheck=0
<!-- -->

### Install these packeges on all nodes master and minions
<!-- -->
yum -y install --enablerepo=virt7-docker-common-release kubernetes docker
<!-- -->
### Packages for Master
### logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

### journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

### Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"

### How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://gkdangal2:8080"

### etcd server on master
[root@gkdangal2 ~]# vim /etc/etcd/etcd.conf
KUBE_ETCD_SERVERS="--etcd-servers=http://gkdangal2:2379"
<!-- -->
###[Member]
###ETCD_CORS=""
ETCD_NAME="default"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
###ETCD_WAL_DIR=""
###ETCD_LISTEN_PEER_URLS="http://localhost:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
###ETCD_MAX_SNAPSHOTS="5"
###ETCD_MAX_WALS="5"
####ETCD_NAME="default"
####ETCD_SNAPSHOT_COUNT="100000"
####ETCD_HEARTBEAT_INTERVAL="100"
####ETCD_ELECTION_TIMEOUT="1000"
####ETCD_QUOTA_BACKEND_BYTES="0"
#
####[Clustering]
####ETCD_INITIAL_ADVERTISE_PEER_URLS="http://localhost:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://0.0.0.0:2379"

<!-- -->
#Api server on Master
[root@gkdangal2 ~]# vim /etc/kubernetes/apiserver
###
#### kubernetes system config
#
# The following values are used to configure the kube-apiserver
#

# The address on the local server to listen to.
KUBE_API_ADDRESS="--address=0.0.0.0"

# The port on the local server to listen on.
 KUBE_API_PORT="--port=8080"

# Port minions listen on
 KUBELET_PORT="--kubelet-port=10250"

# Comma separated list of nodes in the etcd cluster
KUBE_ETCD_SERVERS="--etcd-servers=http://127.0.0.1:2379"

# Address range to use for services
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"

# default admission control policies
#KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota"

# Add your own!
KUBE_API_ARGS=""
## Service enable and run on master
1. systemctl enable etcd
<!-- -->
2. systemctl enable kube-apiserver
<!-- -->
3. systemctl enable kube-controller-manager
<!-- -->
4. systemctl enable kube-scheduler
<!-- --> 
## By one command
[root@gkdangal2 ~]# systemctl enable etcd kube-apiserver kube-controller-manager kube-scheduler
<!-- -->
[root@gkdangal2 ~]# systemctl start etcd kube-apiserver kube-controller-manager kube-scheduler
<!-- -->
##checking running services
[root@gkdangal2 ~]# systemctl status etcd kube-apiserver kube-controller-manager kube-scheduler | grep "(running)" | wc -l
4
<!-- -->
## Now configuring Minions

[root@gkdangal2 ~]# vim /etc/kubernetes/config
# logging to stderr means we get it in the systemd journal
KUBE_LOGTOSTDERR="--logtostderr=true"

# journal message level, 0 is debug
KUBE_LOG_LEVEL="--v=0"

# Should this cluster be allowed to run privileged docker containers
KUBE_ALLOW_PRIV="--allow-privileged=false"

# How the controller-manager, scheduler, and proxy find the apiserver
KUBE_MASTER="--master=http://gkdangal2:8080"
#
# #etcd server
KUBE_ETCD_SERVERS="--etcd-servers=http://gkdangal2:2379"

## configuring kubelet on Minions
[root@gkdangal2 ~]# vim /etc/kubernetes/kubelet

###
# kubernetes kubelet (minion) config
<!-- -->
# The address for the info server to serve on (set to 0.0.0.0 or "" for all interfaces)
KUBELET_ADDRESS="--address=0.0.0.0"

# The port for the info server to serve on
KUBELET_PORT="--port=10250"

# You may leave this blank to use the actual hostname
KUBELET_HOSTNAME="--hostname-override=gkdangal5"

# location of the api-server
KUBELET_API_SERVER="--api-servers=http://gkdangal2:8080"

# pod infrastructure container
#KUBELET_POD_INFRA_CONTAINER="--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"

# Add your own!
KUBELET_ARGS=""
<!-- -->
## Restarting services on Minions
[root@gkdangal5 kubernetes]# systemctl enable kube-proxy kubelet docker
<!-- -->
[root@gkdangal5 kubernetes]# systemctl start kube-proxy kubelet docker
<!-- -->
[root@gkdangal5 kubernetes]# systemctl status kube-proxy kubelet docker | grep "(running)" | wc -l
3
<!-- -->
[root@gkdangal5 kubernetes]#
###################################################################
## geting external ip information
[root@gkdangal2 ~]# kubectl get nodes -o jsonpath='{ .items[*].status.addresses[?(@.type=="ExternalIP")].address}'
##to get how many nodes are ready
[root@gkdangal2 ~]# kubectl get nodes -o jsonpath='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' | tr ';' "\n" | grep "Ready=True"
<!-- -->
Ready=True
Ready=True
Ready=True
<!-- -->
[root@gkdangal2 ~]# 
## with out grep 
[root@gkdangal2 ~]# kubectl get nodes -o jsonpath='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' | tr ';' "\n" 
gkdangal3:OutOfDisk=False
MemoryPressure=False
DiskPressure=False
Ready=True
<!-- -->
gkdangal4:OutOfDisk=False
MemoryPressure=False
DiskPressure=False
Ready=True
<!-- -->
gkdangal5:OutOfDisk=False
MemoryPressure=False
DiskPressure=False
Ready=True
<!-- -->
## Right now to get pods
[root@gkdangal2 ~]# kubectl get pods
No resources found. Because we havent cretae any pods on it.
[root@gkdangal2 ~]#
##Docker Fundamentals
# During This Installation i faced one problem i.e. i can run all docker command easilty but i can not run docker command vai normal user
<!-- -->
[user@gkdangal3 ~]$ docker images
Cannot connect to the Docker daemon. Is the docker daemon running on this host?
[root@gkdangal5 ~]# ls -la /var/run/docker.sock
srw-rw---- 1 root root 0 Jan 10 21:06 /var/run/docker.sock

<!-- -->
# to fixed this issue i search several documentation and i found the solution

[root@gkdangal3 ~]# groupadd docker
[root@gkdangal3 ~]# usermod -aG docker user
[root@gkdangal3 ~]# systemctl restart docker

[root@gkdangal5 ~]# ls -la /var/run/docker.sock
srw-rw---- 1 root docker 0 Jan 10 21:06 /var/run/docker.sock
exit from terminal and relogin then you can use docker command as a normal user.
Afater doing this i can run docker command as a normal user.
<!-- -->
[user@gkdangal5 ~]$ docker images
<!-- -->
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
<!-- -->
docker.io/hello-world   latest              f2a91732366c        7 weeks ago         1.848 kB
<!-- -->
[user@gkdangal5 ~]$ 
<!-- -->

## How to create non-privilege user on docker imaages
1. cerete a Docker files
[user@gkdangal5 ~]$ vim DockerFile
#### Dockerfile based on the latest centos 7 image - non-privileged user
<!-- --> 
FROM centos:latest
<!-- -->
MAINTAINER gkdangal@gmail.com
<!-- -->
RUN useradd -ms /bin/bash govinda
<!-- -->
USER govinda
<!-- -->
[user@gkdangal5 ~]$ docker build -t centos7/nonroot:v1 .
$$ here . means i am going to create docker image at the same location of Dockerfile.
<!-- -->
[user@gkdangal3 ~]$ docker images
<!-- -->
[user@gkdangal3 ~]$ docker run -it -u 0 centos7/nonroot:v1
[root@edc85ad65e15 /]# exit
<!-- -->
[user@gkdangal3 ~]$ docker run -it centos7/nonroot:v1
[govinda@64df2041cdf5 /]$ exit
<!-- -->
[user@gkdangal3 ~]$ 
<!-- -->
### Create and Deploy Pods Defination using kubectl command
<!-- -->
 kubectl get nodes
<!-- -->
 kubectl get pods
<!-- -->
craete a folder first
Create .yaml or .yml files
Here i am going to create one nginx pod => Indentation is very important for yml command
[root@gkdangal2 Build]# vim nginx.yaml 

apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.7.9
    ports:
    - containerPort: 80
-----------------------
save and run this command
[root@gkdangal2 Build]# kubectl delete pods nginx
pod "nginx" deleted
[root@gkdangal2 Build]#
[root@gkdangal2 Build]# kubectl create -f nginx.yaml 
pod "nginx" created
you can create another pod this  way but you have to delete first one
[root@gkdangal2 Build]# kubectl create -f ./nginx.yaml 
pod "nginx" created
[root@gkdangal2 Build]# 

==Port forwarding of pod to outer world 

[root@gkdangal2 Build]# kubectl port-forward nginx :80 &
[1] 2137
[root@gkdangal2 Build]# Forwarding from 127.0.0.1:36778 -> 80
Forwarding from [::1]:36778 -> 80

[root@gkdangal2 Build]#
<!-- --> 
-- here nginx pods is running on port 80 and external host port number to access it is 36778
#### if you want test with in container using bysy box here is the command
you can see history folder to check command
[root@gkdangal2 Build]# kubectl run busybox --image=busybox --restart=Never --tty -i --generator=run-pod/v1
###
<!-- --> 


















