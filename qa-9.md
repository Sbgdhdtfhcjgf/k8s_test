## Question 9 | Kill Scheduler, Manual Scheduling

*Task weight: 5%*

 

Use context: `kubectl config use-context k8s-c2-AC`

 

Ssh into the controlplane node with `ssh cluster2-controlplane1`. **Temporarily** stop the kube-scheduler, this means in a way that you can start it again afterwards.

Create a single *Pod* named `manual-schedule` of image `httpd:2.4-alpine`, confirm it's created but not scheduled on any node.

Now you're the scheduler and have all its power, manually schedule that *Pod* on node cluster2-controlplane1. Make sure it's running.

Start the kube-scheduler again and confirm it's running correctly by creating a second *Pod* named `manual-schedule2` of image `httpd:2.4-alpine` and check if it's running on cluster2-node1.

##### Answer:

###### Stop the Scheduler

First we find the controlplane node:

```bash
k get node
ssh cluster2-controlplane1
```

Kill the Scheduler (temporarily):

```
➜ root@cluster2-controlplane1:~# cd /etc/kubernetes/manifests/
➜ root@cluster2-controlplane1:~# mv kube-scheduler.yaml ..
```

没有想到仅仅是把这个清单移动一下就是暂停使用

And it should be stopped:

```
➜ root@cluster2-controlplane1:~# kubectl -n kube-system get pod | grep schedule
➜ root@cluster2-controlplane1:~# 
```

 创建一个pod

```bash
k run  manual-schedule --image=httpd:2.4-alpine
k get po manual-schedule  -owide 
k describe po manual-schedule 
k get events 
```

###### Manually schedule the *Pod*

Let's play the scheduler now:

```
k get pod manual-schedule -o yaml > 9.yaml
```

```yaml
# 9.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-09-04T15:51:02Z"
  labels:
    run: manual-schedule
  managedFields:
...
    manager: kubectl-run
    operation: Update
    time: "2020-09-04T15:51:02Z"
  name: manual-schedule
  namespace: default
  resourceVersion: "3515"
  selfLink: /api/v1/namespaces/default/pods/manual-schedule
  uid: 8e9d2532-4779-4e63-b5af-feb82c74a935
spec:
  nodeName: cluster2-controlplane1        # add the controlplane node name
  containers:
  - image: httpd:2.4-alpine
    imagePullPolicy: IfNotPresent
    name: manual-schedule
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-nxnc7
      readOnly: true
  dnsPolicy: ClusterFirst
...
```

或者直接edit 试一下

直接edit 不行看来还是得 k apply -f xxx.yml --force

```bash
# 直接添加一行nodeName
k apply -f xxx.yml --force
```

看起来我们的吊舱现在按照要求在控制平面上运行，尽管未指定公差。只有在找到正确的节点名称时，只有调度程序考虑到tains/bol/tolerations/affinity。这就是为什么仍然可以将POD直接分配到Control Plane节点并跳过调度程序的原因。

###### Start the scheduler again

```
➜ ssh cluster2-controlplane1
➜ root@cluster2-controlplane1:~# cd /etc/kubernetes/manifests/
➜ root@cluster2-controlplane1:~# mv ../kube-scheduler.yaml .
```

Checks it's running:

```
➜ root@cluster2-controlplane1:~# kubectl -n kube-system get pod | grep schedule
kube-scheduler-cluster2-controlplane1            1/1     Running   0          16s
```

Schedule a second test *Pod*:

```
k run manual-schedule2 --image=httpd:2.4-alpine
➜ k get pod -o wide | grep schedule
manual-schedule    1/1     Running   ...   cluster2-controlplane1
manual-schedule2   1/1     Running   ...   cluster2-node1
```

Back to normal.