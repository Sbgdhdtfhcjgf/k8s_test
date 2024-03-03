## Question 6 | Storage, PV, PVC, Pod volume

*Task weight: 8%*

 

Use context: `kubectl config use-context k8s-c1-H`

 

Create a new *PersistentVolume* named `safari-pv`. It should have a capacity of *2Gi*, accessMode *ReadWriteOnce*, hostPath `/Volumes/Data` and no storageClassName defined.

Next create a new *PersistentVolumeClaim* in *Namespace* `project-tiger` named `safari-pvc` . It should request *2Gi* storage, accessMode *ReadWriteOnce* and should not define a storageClassName. The *PVC* should bound to the *PV* correctly.

Finally create a new *Deployment* `safari` in *Namespace* `project-tiger` which mounts that volume at `/tmp/safari-data`. The *Pods* of that *Deployment* should be of image `httpd:2.4.41-alpine`.

```yaml
apiVersion: v1
kind: PresistentVolume
metadata:
  name: safari-pv
spec:
  assessModes:
  - ReadWriteOnce
  capacity:
    storage: 2Gi
  hostPath:
    path: /Volumes/Data
---
apiVersion: v1
kind: PresistentVolumeClaim
metadata:
  name: safari-pvc
  namespace: project-tiger
spec:
  assessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

```yml
apiVerison: apps/v1
kind: Deployment
metadata: 
  name: safari
  namsspace: project-tiger
spec:
  replicas: 1
  selector:
    matchLabels:
      aa: bb
  template:
    metadata:
      labels:
        aa: bb 
     spec:
       volumes:
       - name: v1
         presistentVolumeClaim: 
           claimName: safari-pvc
       containers:
       - image: httpd:2.4.41-alpine
         name: c1
         volumeMounts:
         - name: v1
           mountPath: /tmp/safari-data
```

```bash
k describe po <> | grep -A10 -i mounts 
```

