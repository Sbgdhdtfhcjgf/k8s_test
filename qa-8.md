## Question 8 | Get Controlplane Information

*Task weight: 2%*

 

Use context: `kubectl config use-context k8s-c1-H`

 

Ssh into the controlplane node with `ssh cluster1-controlplane1`. Check how the controlplane components kubelet, kube-apiserver, kube-scheduler, kube-controller-manager and etcd are started/installed on the controlplane node. Also find out the name of the DNS application and how it's started/installed on the controlplane node.

Write your findings into file `/opt/course/8/controlplane-components.txt`. The file should be structured like:

````bash
# 首先是kubelet
ssh cluster1-controlplane1
ps -aux | grep kubelet 
find /etc/systemd/system/ | grep kube
find /etc/kubernetes/manifests/
````

````bash
# /opt/course/8/controlplane-components.txt
kubelet: process
kube-apiserver: static-pod
kube-scheduler: static-pod
kube-controller-manager: static-pod
etcd: static-pod
````

