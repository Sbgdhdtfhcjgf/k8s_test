Use context: `kubectl config use-context k8s-c1-H`

 

Check all available *Pods* in the *Namespace* `project-c13` and find the names of those that would probably be terminated first if the *nodes* run out of resources (cpu or memory) to schedule all *Pods*. Write the *Pod* names into `/opt/course/e1/pods-not-stable.txt`.

##### Answer:

When available cpu or memory resources on the nodes reach their limit, Kubernetes will look for *Pods* that are using more resources than they requested. These will be the first candidates for termination. If some *Pods* containers have no resource requests/limits set, then by default those are considered to use more than requested.

Kubernetes assigns Quality of Service classes to *Pods* based on the defined resources and limits, read more here: https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod

Hence we should look for *Pods* without resource requests defined, we can do this with a manual approach:

```
k -n project-c13 describe pod | less -p Requests # describe all pods and highlight Requests
```

Or we do:

```
k -n project-c13 describe pod | egrep "^(Name:|    Requests:)" -A1
```

We see that the *Pods* of *Deployment* `c13-3cc-runner-heavy` don't have any resources requests specified. Hence our answer would be:

```
# /opt/course/e1/pods-not-stable.txt
c13-3cc-runner-heavy-65588d7d6-djtv9map
c13-3cc-runner-heavy-65588d7d6-v8kf5map
c13-3cc-runner-heavy-65588d7d6-wwpb4map
o3db-0
o3db-1 # maybe not existing if already removed via previous scenario 
```

To automate this process you could use jsonpath like this:

```
➜ k -n project-c13 get pod \
  -o jsonpath="{range .items[*]} {.metadata.name}{.spec.containers[*].resources}{'\n'}"

 c13-2x3-api-86784557bd-cgs8gmap[requests:map[cpu:50m memory:20Mi]]
 c13-2x3-api-86784557bd-lnxvjmap[requests:map[cpu:50m memory:20Mi]]
 c13-2x3-api-86784557bd-mnp77map[requests:map[cpu:50m memory:20Mi]]
 c13-2x3-web-769c989898-6hbgtmap[requests:map[cpu:50m memory:10Mi]]
 c13-2x3-web-769c989898-g57nqmap[requests:map[cpu:50m memory:10Mi]]
 c13-2x3-web-769c989898-hfd5vmap[requests:map[cpu:50m memory:10Mi]]
 c13-2x3-web-769c989898-jfx64map[requests:map[cpu:50m memory:10Mi]]
 c13-2x3-web-769c989898-r89mgmap[requests:map[cpu:50m memory:10Mi]]
 c13-2x3-web-769c989898-wtgxlmap[requests:map[cpu:50m memory:10Mi]]
 c13-3cc-runner-98c8b5469-dzqhrmap[requests:map[cpu:30m memory:10Mi]]
 c13-3cc-runner-98c8b5469-hbtdvmap[requests:map[cpu:30m memory:10Mi]]
 c13-3cc-runner-98c8b5469-n9lswmap[requests:map[cpu:30m memory:10Mi]]
 c13-3cc-runner-heavy-65588d7d6-djtv9map[]
 c13-3cc-runner-heavy-65588d7d6-v8kf5map[]
 c13-3cc-runner-heavy-65588d7d6-wwpb4map[]
 c13-3cc-web-675456bcd-glpq6map[requests:map[cpu:50m memory:10Mi]]
 c13-3cc-web-675456bcd-knlpxmap[requests:map[cpu:50m memory:10Mi]]
 c13-3cc-web-675456bcd-nfhp9map[requests:map[cpu:50m memory:10Mi]]
 c13-3cc-web-675456bcd-twn7mmap[requests:map[cpu:50m memory:10Mi]]
 o3db-0{}
 o3db-1{}
```

This lists all *Pod* names and their requests/limits, hence we see the three *Pods* without those defined.

Or we look for the Quality of Service classes:

```
➜ k get pods -n project-c13 \
  -o jsonpath="{range .items[*]}{.metadata.name} {.status.qosClass}{'\n'}"

c13-2x3-api-86784557bd-cgs8g Burstable
c13-2x3-api-86784557bd-lnxvj Burstable
c13-2x3-api-86784557bd-mnp77 Burstable
c13-2x3-web-769c989898-6hbgt Burstable
c13-2x3-web-769c989898-g57nq Burstable
c13-2x3-web-769c989898-hfd5v Burstable
c13-2x3-web-769c989898-jfx64 Burstable
c13-2x3-web-769c989898-r89mg Burstable
c13-2x3-web-769c989898-wtgxl Burstable
c13-3cc-runner-98c8b5469-dzqhr Burstable
c13-3cc-runner-98c8b5469-hbtdv Burstable
c13-3cc-runner-98c8b5469-n9lsw Burstable
c13-3cc-runner-heavy-65588d7d6-djtv9 BestEffort
c13-3cc-runner-heavy-65588d7d6-v8kf5 BestEffort
c13-3cc-runner-heavy-65588d7d6-wwpb4 BestEffort
c13-3cc-web-675456bcd-glpq6 Burstable
c13-3cc-web-675456bcd-knlpx Burstable
c13-3cc-web-675456bcd-nfhp9 Burstable
c13-3cc-web-675456bcd-twn7m Burstable
o3db-0 BestEffort
o3db-1 BestEffort
```

Here we see three with BestEffort, which *Pods* get that don't have any memory or cpu limits or requests defined.

A good practice is to always set resource requests and limits. If you don't know the values your containers should have you can find this out using metric tools like Prometheus. You can also use `kubectl top pod` or even `kubectl exec` into the container and use `top` and similar tools.