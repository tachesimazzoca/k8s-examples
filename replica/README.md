# Kubernetes Examples | ReplicaSet

```bash
# Create a replica set from YAML
$ kubectl create -f replicaset.yml
replicaset.apps/item-api-replicaset created

$ kubectl get replicaset
NAME                  DESIRED   CURRENT   READY   AGE
item-api-replicaset   3         3         3       14s

$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
item-api-replicaset-g4rfd   1/1     Running   0          18s
item-api-replicaset-lphbm   1/1     Running   0          18s
item-api-replicaset-qd476   1/1     Running   0          18s

# Even if one of the pods in ReplicaSet is deleted, ...
$ kubectl delete pod item-api-replicaset-g4rfd
pod "item-api-replicaset-g4rfd" deleted

# ReplicaSet will keep the number of replicas: 3
$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
item-api-replicaset-lphbm   1/1     Running   0          50s
item-api-replicaset-qd476   1/1     Running   0          50s
item-api-replicaset-r4lx9   1/1     Running   0          16s

# Even if a new pod is created, ...
$ kubectl create -f single-pod.yml
pod/item-api-pod created

# ReplicaSet will also keep the number of replicas: 3
$ kubectl get pods
NAME                        READY   STATUS        RESTARTS   AGE
item-api-pod                0/1     Terminating   0          2s
item-api-replicaset-lphbm   1/1     Running       0          74s
item-api-replicaset-qd476   1/1     Running       0          74s
item-api-replicaset-r4lx9   1/1     Running       0          40s

$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
item-api-replicaset-lphbm   1/1     Running   0          80s
item-api-replicaset-qd476   1/1     Running   0          80s
item-api-replicaset-r4lx9   1/1     Running   0          46s

# The scale command increases the number of replicas
$ kubectl scale --replicas=5 -f replicaset.yml
replicaset.apps/item-api-replicaset scaled

$ kubectl get pods
NAME                        READY   STATUS              RESTARTS   AGE
item-api-replicaset-c7r9s   0/1     ContainerCreating   0          3s
item-api-replicaset-lphbm   1/1     Running             0          106s
item-api-replicaset-qd476   1/1     Running             0          106s
item-api-replicaset-r4lx9   1/1     Running             0          72s
item-api-replicaset-xs799   0/1     ContainerCreating   0          3s

$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
item-api-replicaset-c7r9s   1/1     Running   0          10s
item-api-replicaset-lphbm   1/1     Running   0          113s
item-api-replicaset-qd476   1/1     Running   0          113s
item-api-replicaset-r4lx9   1/1     Running   0          79s
item-api-replicaset-xs799   1/1     Running   0          10s

# Replace the replica set with the original from YAML
$ kubectl replace -f replicaset.yml
replicaset.apps/item-api-replicaset replaced

$ kubectl get pods
NAME                        READY   STATUS        RESTARTS   AGE
item-api-replicaset-c7r9s   0/1     Terminating   0          24s
item-api-replicaset-lphbm   1/1     Running       0          2m7s
item-api-replicaset-qd476   1/1     Running       0          2m7s
item-api-replicaset-r4lx9   1/1     Running       0          93s
item-api-replicaset-xs799   0/1     Terminating   0          24s

$ kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
item-api-replicaset-lphbm   1/1     Running   0          2m20s
item-api-replicaset-qd476   1/1     Running   0          2m20s
item-api-replicaset-r4lx9   1/1     Running   0          106s

# Delete the replica set with YAML
$ kubectl delete -f replicaset.yml
replicaset.apps "item-api-replicaset" deleted

$ kubectl get pods
NAME                        READY   STATUS        RESTARTS   AGE
item-api-replicaset-lphbm   0/1     Terminating   0          2m37s
item-api-replicaset-qd476   0/1     Terminating   0          2m37s

$ kubectl get pods
No resources found in default namespace.
```
