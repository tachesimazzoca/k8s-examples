# Kubernetes Examples | Rollout

```bash
$ kubectl get deployments
No resources found in default namespace.

# Create a deployment from YAML with --record
$ kubectl create -f deployment.yml --record
deployment.apps/item-api-deployment created

# Watch the rollout status
$ kubectl rollout status deployment/item-api-deployment
Waiting for deployment "item-api-deployment" rollout to finish: 0 of 3 updated replicas are available...
Waiting for deployment "item-api-deployment" rollout to finish: 1 of 3 updated replicas are available...
Waiting for deployment "item-api-deployment" rollout to finish: 2 of 3 updated replicas are available...
deployment "item-api-deployment" successfully rolled out

# View the rollout revisions
$ kubectl rollout history deployment/item-api-deployment
deployment.apps/item-api-deployment
REVISION  CHANGE-CAUSE
1         kubectl.exe create --filename=deployment.yml --record=true

# The replica set has been created
$ kubectl get replicasets
NAME                             DESIRED   CURRENT   READY   AGE
item-api-deployment-7d6f8987b5   3         3         3       28s

# Update nginx:1.17 to 1.18
$ kubectl set image deployment/item-api-deployment item-api-proxy=nginx:1.18 --record
deployment.apps/item-api-deployment image updated

$ kubectl rollout history deployment/item-api-deployment
deployment.apps/item-api-deployment
REVISION  CHANGE-CAUSE
1         kubectl.exe create --filename=deployment.yml --record=true
2         kubectl.exe set image deployment/item-api-deployment item-api-proxy=nginx:1.18 --record=true

# The replica set has been rolling updated, keeping the old replica set as replicas:0
$ kubectl get replicasets
NAME                             DESIRED   CURRENT   READY   AGE
item-api-deployment-77cf8ddbb8   3         3         3       22s
item-api-deployment-7d6f8987b5   0         0         0       80s

$ kubectl describe deployment/item-api-deployment
Name:                   item-api-deployment
Namespace:              default
...
Labels:                 app=item-api
...
Selector:               app=item-api
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=item-api
  Containers:
   item-api-proxy:
    Image:        nginx:1.18
    ...
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   item-api-deployment-77cf8ddbb8 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  101s  deployment-controller  Scaled up replica set item-api-deployment-7d6f8987b5 to 3
  Normal  ScalingReplicaSet  43s   deployment-controller  Scaled up replica set item-api-deployment-77cf8ddbb8 to 1
  Normal  ScalingReplicaSet  40s   deployment-controller  Scaled down replica set item-api-deployment-7d6f8987b5 to 2
  Normal  ScalingReplicaSet  40s   deployment-controller  Scaled up replica set item-api-deployment-77cf8ddbb8 to 2
  Normal  ScalingReplicaSet  37s   deployment-controller  Scaled down replica set item-api-deployment-7d6f8987b5 to 1
  Normal  ScalingReplicaSet  37s   deployment-controller  Scaled up replica set item-api-deployment-77cf8ddbb8 to 3
  Normal  ScalingReplicaSet  34s   deployment-controller  Scaled down replica set item-api-deployment-7d6f8987b5 to 0

# Rollback to the previous deployment
$ kubectl rollout undo deployment/item-api-deployment
deployment.apps/item-api-deployment rolled back

# The replica set is going to updated...
$ kubectl get replicasets
NAME                             DESIRED   CURRENT   READY   AGE
item-api-deployment-77cf8ddbb8   2         2         2       68s
item-api-deployment-7d6f8987b5   2         2         1       2m6s

$ kubectl get replicasets
NAME                             DESIRED   CURRENT   READY   AGE
item-api-deployment-77cf8ddbb8   1         1         1       70s
item-api-deployment-7d6f8987b5   3         3         2       2m8s

$ kubectl get replicasets
NAME                             DESIRED   CURRENT   READY   AGE
item-api-deployment-77cf8ddbb8   0         0         0       72s
item-api-deployment-7d6f8987b5   3         3         3       2m10s

# The rollout revision shows undo history
$ kubectl rollout history deployment/item-api-deployment
deployment.apps/item-api-deployment
REVISION  CHANGE-CAUSE
2         kubectl.exe set image deployment/item-api-deployment item-api-proxy=nginx:1.18 --record=true
3         kubectl.exe create --filename=deployment.yml --record=true

$ kubectl describe deployment/item-api-deployment
Name:                   item-api-deployment
Namespace:              default
...
Labels:                 app=item-api
...
Selector:               app=item-api
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=item-api
  Containers:
   item-api-proxy:
    Image:        nginx:1.17
    ...
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   item-api-deployment-7d6f8987b5 (3/3 replicas created)
Events:
  Type    Reason             Age                  From                   Message
  ----    ------             ----                 ----                   -------
  Normal  ScalingReplicaSet  88s                  deployment-controller  Scaled up replica set item-api-deployment-77cf8ddbb8 to 1
  Normal  ScalingReplicaSet  85s                  deployment-controller  Scaled down replica set item-api-deployment-7d6f8987b5 to 2
  Normal  ScalingReplicaSet  85s                  deployment-controller  Scaled up replica set item-api-deployment-77cf8ddbb8 to 2
  Normal  ScalingReplicaSet  82s                  deployment-controller  Scaled down replica set item-api-deployment-7d6f8987b5 to 1
  Normal  ScalingReplicaSet  82s                  deployment-controller  Scaled up replica set item-api-deployment-77cf8ddbb8 to 3
  Normal  ScalingReplicaSet  79s                  deployment-controller  Scaled down replica set item-api-deployment-7d6f8987b5 to 0
  Normal  ScalingReplicaSet  25s                  deployment-controller  Scaled up replica set item-api-deployment-7d6f8987b5 to 1
  Normal  ScalingReplicaSet  22s                  deployment-controller  Scaled down replica set item-api-deployment-77cf8ddbb8 to 2
  Normal  ScalingReplicaSet  19s (x2 over 2m26s)  deployment-controller  Scaled up replica set item-api-deployment-7d6f8987b5 to 3
  Normal  ScalingReplicaSet  16s (x3 over 22s)    deployment-controller  (combined from similar events): Scaled down replica set item-api-deployment-77cf8ddbb8 to 0

# Delete the deployment
$ kubectl delete deployment/item-api-deployment
deployment.apps "item-api-deployment" deleted

$ kubectl get deployments
No resources found in default namespace.
```
