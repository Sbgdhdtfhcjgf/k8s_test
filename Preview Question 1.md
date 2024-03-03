The cluster admin asked you to find out the following information about etcd running on cluster2-controlplane1:

- Server private key location
- Server certificate expiration date
- Is client certificate authentication enabled

Write these information into `/opt/course/p1/etcd-info.txt`



Finally you're asked to save an etcd snapshot at `/etc/etcd-snapshot.db` on cluster2-controlplane1 and display its status.

##### Answer:

###### Find out etcd information

Let's check the nodes:

```bash
k get node
ssh cluster2-controlplane1
```

First we check how etcd is setup in this cluster:

```bash
kubectl -n kube-system get pod
```

We see it's running as a *Pod*, more specific a static *Pod*. So we check for the default kubelet directory for static manifests:

```bash
find /etc/kubernetes/manifests/
vim /etc/kubernetes/manifests/etcd.yaml
```

````yml
# /etc/kubernetes/manifests/etcd.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://192.168.102.11:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt              # server certificate
    - --client-cert-auth=true                                      # enabled
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://192.168.102.11:2380
    - --initial-cluster=cluster2-controlplane1=https://192.168.102.11:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key               # server private key
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.102.11:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://192.168.102.11:2380
    - --name=cluster2-controlplane1
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
...
````

We see that client authentication is enabled and also the requested path to the server private key, now let's find out the expiration of the server certificate:

```bash
openssl x509  -noout -text -in /etc/kubernetes/pki/etcd/server.crt | grep Validity -A2
```

There we have it. Let's write the information into the requested file:

```bash
# /opt/course/p1/etcd-info.txt
Server private key location: /etc/kubernetes/pki/etcd/server.key
Server certificate expiration date: Sep 13 13:01:31 2022 GMT
Is client certificate authentication enabled: yes
```

###### Create etcd snapshot

First we try:

````bash
ETCDCTL_API=3 etcdctl snapshot save /etc/etcd-snapshot.db
````

We get the endpoint also from the yaml. But we need to specify more parameters, all of which we can find the yaml declaration above:

```bash
ETCDCTL_API=3 etcdctl snapshot save /etc/etcd-snapshot.db \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--cert /etc/kubernetes/pki/etcd/server.crt \
--key /etc/kubernetes/pki/etcd/server.key
```

This worked. Now we can output the status of the backup file:

```bash
ETCDCTL_API=3 etcdctl snapshot status /etc/etcd-snapshot.db
```

