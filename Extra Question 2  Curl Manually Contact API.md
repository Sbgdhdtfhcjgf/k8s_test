Use context: `kubectl config use-context k8s-c1-H`

 

There is an existing *ServiceAccount* `secret-reader` in *Namespace* `project-hamster`. Create a *Pod* of image `curlimages/curl:7.65.3` named `tmp-api-contact` which uses this *ServiceAccount*. Make sure the container keeps running.

Exec into the *Pod* and use `curl` to access the Kubernetes Api of that cluster manually, listing all available secrets. You can ignore insecure https connection. Write the command(s) for this into file `/opt/course/e4/list-secrets.sh`.

 

##### Answer:

https://kubernetes.io/docs/tasks/run-application/access-api-from-pod

It's important to understand how the Kubernetes API works. For this it helps connecting to the api manually, for example using curl. You can find information fast by search in the Kubernetes docs for "curl api" for example.

First we create our *Pod*:

```
k run tmp-api-contact \
  --image=curlimages/curl:7.65.3 $do \
  --command > e2.yaml -- sh -c 'sleep 1d'

vim e2.yaml
```

Add the service account name and *Namespace*:

```
# e2.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: tmp-api-contact
  name: tmp-api-contact
  namespace: project-hamster          # add
spec:
  serviceAccountName: secret-reader   # add
  containers:
  - command:
    - sh
    - -c
    - sleep 1d
    image: curlimages/curl:7.65.3
    name: tmp-api-contact
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Then run and exec into:

```
k -f 6.yaml create

k -n project-hamster exec tmp-api-contact -it -- sh
```

Once on the container we can try to connect to the api using `curl`, the api is usually available via the *Service* named `kubernetes` in *Namespace* `default` (You should know how dns resolution works across *Namespaces*.). Else we can find the endpoint IP via environment variables running `env`.

So now we can do:

```
curl https://kubernetes.default
curl -k https://kubernetes.default # ignore insecure as allowed in ticket description
curl -k https://kubernetes.default/api/v1/secrets # should show Forbidden 403
```

The last command shows 403 forbidden, this is because we are not passing any authorisation information with us. The Kubernetes Api Server thinks we are connecting as `system:anonymous`. We want to change this and connect using the *Pods* *ServiceAccount* named `secret-reader`.

We find the the token in the mounted folder at `/var/run/secrets/kubernetes.io/serviceaccount`, so we do:

```
➜ TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
➜ curl -k https://kubernetes.default/api/v1/secrets -H "Authorization: Bearer ${TOKEN}"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0{
  "kind": "SecretList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/secrets",
    "resourceVersion": "10697"
  },
  "items": [
    {
      "metadata": {
        "name": "default-token-5zjbd",
        "namespace": "default",
        "selfLink": "/api/v1/namespaces/default/secrets/default-token-5zjbd",
        "uid": "315dbfd9-d235-482b-8bfc-c6167e7c1461",
        "resourceVersion": "342",
...
```

Now we're able to list all *Secrets*, registering as the *ServiceAccount* `secret-reader` under which our *Pod* is running.

To use encrypted https connection we can run:

```
CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
curl --cacert ${CACERT} https://kubernetes.default/api/v1/secrets -H "Authorization: Bearer ${TOKEN}"
```

For troubleshooting we could also check if the *ServiceAccount* is actually able to list *Secrets* using:

```
➜ k auth can-i get secret --as system:serviceaccount:project-hamster:secret-reader
yes
```

Finally write the commands into the requested location:

```
# /opt/course/e4/list-secrets.sh
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -k https://kubernetes.default/api/v1/secrets -H "Authorization: Bearer ${TOKEN}"
```

 