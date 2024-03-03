## Question 7 | Node and Pod Resource Usage

*Task weight: 1%*

 

Use context: `kubectl config use-context k8s-c1-H`

 

The metrics-server has been installed in the cluster. Your college would like to know the kubectl commands to:

1. show *Nodes* resource usage
2. show *Pods* and their containers resource usage

Please write the commands into `/opt/course/7/node.sh` and `/opt/course/7/pod.sh`.

```bash
k top nodes 
k top pod -h 
k top po --containers=true 
```

