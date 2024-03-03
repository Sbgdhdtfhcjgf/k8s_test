Use context: `kubectl config use-context k8s-c2-AC`

 

Create a *Pod* named `check-ip` in *Namespace* `default` using image `httpd:2.4.41-alpine`. Expose it on port 80 as a ClusterIP *Service* named `check-ip-service`. Remember/output the IP of that *Service*.

Change the Service CIDR to `11.96.0.0/12` for the cluster.

Then create a second *Service* named `check-ip-service2` pointing to the same *Pod* to check if your settings did take effect. Finally check if the IP of the first *Service* has changed.

 Let's create the *Pod* and expose it:

```
k run check-ip --image=httpd:2.4.41-alpine

k expose pod check-ip --name check-ip-service --port 80
```

And check the *Pod* and *Service* ips:

```
➜ k get svc,ep -l run=check-ip
NAME                       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/check-ip-service   ClusterIP   10.104.3.45   <none>        80/TCP    8s

NAME                         ENDPOINTS      AGE
endpoints/check-ip-service   10.44.0.3:80   7s
```

Now we change the *Service* CIDR on the kube-apiserver:

```
➜ ssh cluster2-controlplane1

➜ root@cluster2-controlplane1:~# vim /etc/kubernetes/manifests/kube-apiserver.yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.100.21
...
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-cluster-ip-range=11.96.0.0/12             # change
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
...
```

**Give it a bit for the kube-apiserver and controller-manager to restart**

Wait for the api to be up again:

```
➜ root@cluster2-controlplane1:~# kubectl -n kube-system get pod | grep api
kube-apiserver-cluster2-controlplane1            1/1     Running   0              49s
```

 

 

Now we do the same for the controller manager:

```
➜ root@cluster2-controlplane1:~# vim /etc/kubernetes/manifests/kube-controller-manager.yaml
# /etc/kubernetes/manifests/kube-controller-manager.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: kube-controller-manager
    tier: control-plane
  name: kube-controller-manager
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-controller-manager
    - --allocate-node-cidrs=true
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --bind-address=127.0.0.1
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --cluster-cidr=10.244.0.0/16
    - --cluster-name=kubernetes
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --leader-elect=true
    - --node-cidr-mask-size=24
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --root-ca-file=/etc/kubernetes/pki/ca.crt
    - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=11.96.0.0/12         # change
    - --use-service-account-credentials=true
```

**Give it a bit for the scheduler to restart**.

We can check if it was restarted using `crictl`:

```
➜ root@cluster2-controlplane1:~# crictl ps | grep scheduler
3d258934b9fd6    aca5ededae9c8    About a minute ago   Running    kube-scheduler ...
```

 

 

Checking our existing *Pod* and *Service* again:

```
➜ k get pod,svc -l run=check-ip
NAME           READY   STATUS    RESTARTS   AGE
pod/check-ip   1/1     Running   0          21m

NAME                       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/check-ip-service   ClusterIP   10.99.32.177   <none>        80/TCP    21m
```

Nothing changed so far. Now we create another *Service* like before:

```
k expose pod check-ip --name check-ip-service2 --port 80
```

And check again:

```
➜ k get svc,ep -l run=check-ip
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/check-ip-service    ClusterIP   10.109.222.111   <none>        80/TCP    8m
service/check-ip-service2   ClusterIP   11.111.108.194   <none>        80/TCP    6m32s

NAME                          ENDPOINTS      AGE
endpoints/check-ip-service    10.44.0.1:80   8m
endpoints/check-ip-service2   10.44.0.1:80   6m13s
```

There we go, the new *Service* got an ip of the new specified range assigned. We also see that both *Services* have our *Pod* as endpoint.