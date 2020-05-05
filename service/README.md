# Kubernetes Examples | Service

## NodePort

```bash
# List services
$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   20h

# The service/kubernetes is provided by Kubernetes
$ kubectl describe service/kubernetes
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP:                10.96.0.1
Port:              https  443/TCP
TargetPort:        8443/TCP
Endpoints:         192.168.99.100:8443
Session Affinity:  None
Events:            <none>

# Create a deployment from YAML
$ kubectl create -f item-api-deployment.yml
deployment.apps/item-api-deployment created

# Each pod in the deployment has each different IP
$ kubectl get pods -o wide
NAME                                   READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
item-api-deployment-7d4ffbb8b6-59vxf   1/1     Running   0          85s   172.17.0.4   minikube   <none>           <none>
item-api-deployment-7d4ffbb8b6-hw7k5   1/1     Running   0          85s   172.17.0.6   minikube   <none>           <none>
item-api-deployment-7d4ffbb8b6-vj6lx   1/1     Running   0          85s   172.17.0.5   minikube   <none>           <none>

# Create a service with NodePort from YAML
$ kubectl create -f item-api-service.yml
service/item-api-service created

$ kubectl get services
NAME               TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
item-api-service   NodePort    10.109.119.130   <none>        80:30080/TCP   7s
kubernetes         ClusterIP   10.96.0.1        <none>        443/TCP        20h

# The service allows clients to access the pods having app=item-api via the node port:30080
$ kubectl describe services/item-api-service
Name:                     item-api-service
Namespace:                default
Labels:                   app=item-api
Annotations:              <none>
Selector:                 app=item-api
Type:                     NodePort
IP:                       10.109.119.130
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30080/TCP
Endpoints:                172.17.0.4:80,172.17.0.5:80,172.17.0.6:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>

# Minikube has the command to show the url
$ minikube service item-api-service --url
http://192.168.99.100:30080

$ curl http://192.168.99.100:30080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

# The logs command shows outputs from seleted containers
$ kubectl logs -f --selector='app=item-api' --all-containers
...

# Clean up resources...
$ kubectl delete -f item-api-service.yml
service "item-api-service" deleted

$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   20h

$ kubectl delete -f item-api-deployment.yml
deployment.apps "item-api-deployment" deleted

$ kubectl get deployments
No resources found in default namespace.
```

## ClusterIP

```bash
# Create backend-deployment
$ kubectl create -f backend-deployment.yml
deployment.apps/backend-deployment created

$ kubectl get deployments
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
backend-deployment   3/3     3            3           12s

# Create backend-service
$ kubectl create -f backend-service.yml
service/backend-service created

$ kubectl get services
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
backend-service   ClusterIP   10.108.139.162   <none>        80/TCP    8s
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP   24h

# backend-service has 3 endpoints
$ kubectl describe service/backend-service
Name:              backend-service
Namespace:         default
Labels:            app=backend
Annotations:       <none>
Selector:          app=backend
Type:              ClusterIP
IP:                10.108.139.162
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         172.17.0.4:80,172.17.0.5:80,172.17.0.6:80
Session Affinity:  None
Events:            <none>

# The port-forward command can forward a local port to a port on any pods.
$ kubectl port-forward service/backend-service 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
Handling connection for 8080

# Try `curl localhost:8080` on another termial;
# Press CTRL-C to stop...

# Create check-deployment, which periodically send HTTP requests to backend-service
$ kubectl create -f check-deployment.yml
deployment.apps/check-deployment created

$ kubectl get pods
NAME                                  READY   STATUS    RESTARTS   AGE
backend-deployment-6dd598965c-kpcfq   1/1     Running   0          3m13s
backend-deployment-6dd598965c-qn94s   1/1     Running   0          3m13s
backend-deployment-6dd598965c-xz7t9   1/1     Running   0          3m13s
check-deployment-597d756d46-fjq85     1/1     Running   0          11s
check-deployment-597d756d46-lt5zq     1/1     Running   0          10s

# Check the client IPs of the pods labeled as app=check
$ kubectl get pods --selector='app=check' -o wide
NAME                                READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
check-deployment-597d756d46-fjq85   1/1     Running   0          93s   172.17.0.7   minikube   <none>           <none>
check-deployment-597d756d46-lt5zq   1/1     Running   0          92s   172.17.0.8   minikube   <none>           <none>

# Watch logs selected by app=backend. The check clients actually reach out backend-service
$ kubectl logs -f --selector='app=backend'
2020/05/05 10:56:11 Starting server...
2020/05/05 10:56:11 Health service listening on 0.0.0.0:81
2020/05/05 10:56:11 HTTP service listening on 0.0.0.0:80
2020/05/05 10:56:11 Starting server...
2020/05/05 10:56:11 Health service listening on 0.0.0.0:81
2020/05/05 10:56:11 HTTP service listening on 0.0.0.0:80
2020/05/05 10:56:11 Starting server...
2020/05/05 10:56:11 Health service listening on 0.0.0.0:81
2020/05/05 10:56:11 HTTP service listening on 0.0.0.0:80
172.17.0.7:42930 - - [Tue, 05 May 2020 10:59:16 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.8:56548 - - [Tue, 05 May 2020 10:59:19 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.7:42996 - - [Tue, 05 May 2020 10:59:26 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.8:56608 - - [Tue, 05 May 2020 10:59:29 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.7:43056 - - [Tue, 05 May 2020 10:59:36 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.8:56668 - - [Tue, 05 May 2020 10:59:39 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.7:43116 - - [Tue, 05 May 2020 10:59:46 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.8:56728 - - [Tue, 05 May 2020 10:59:49 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.7:43176 - - [Tue, 05 May 2020 10:59:56 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.8:56788 - - [Tue, 05 May 2020 10:59:59 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.7:43242 - - [Tue, 05 May 2020 11:00:06 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.8:56858 - - [Tue, 05 May 2020 11:00:09 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.7:43302 - - [Tue, 05 May 2020 11:00:16 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.8:56918 - - [Tue, 05 May 2020 11:00:19 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV

# Watch logs selected by app=check
$ kubectl logs -f --selector='app=check'
...
---
check-deployment-597d756d46-fjq85
HTTP/1.1 200 OK
Date: Tue, 05 May 2020 11:00:56 GMT
Content-Length: 20
Content-Type: text/plain; charset=utf-8

Content-Length: 20
Content-Type: text/plain; charset=utf-8

---
check-deployment-597d756d46-lt5zq
HTTP/1.1 200 OK
Date: Tue, 05 May 2020 11:00:59 GMT
Content-Length: 20
Content-Type: text/plain; charset=utf-8
...

# Scale down the replicas of backend-deployment to 1
$ kubectl scale --replicas=1 deployment/backend-deployment
deployment.apps/backend-deployment scaled

$ kubectl get pods
NAME                                  READY   STATUS    RESTARTS   AGE
backend-deployment-6dd598965c-qn94s   1/1     Running   0          6m10s
check-deployment-597d756d46-fjq85     1/1     Running   0          3m8s
check-deployment-597d756d46-lt5zq     1/1     Running   0          3m7s

# The check clients keep receiving valid responses
$ kubectl logs -f --selector='app=check'
...
---
check-deployment-597d756d46-fjq85
HTTP/1.1 200 OK
Date: Tue, 05 May 2020 11:02:06 GMT
Content-Length: 20
Content-Type: text/plain; charset=utf-8

---
check-deployment-597d756d46-lt5zq
HTTP/1.1 200 OK
Date: Tue, 05 May 2020 11:02:10 GMT
Content-Length: 20
Content-Type: text/plain; charset=utf-8
...

# Scale up the replicas of check-deployment to 3
$ kubectl scale --replicas=3 deployment/check-deployment
deployment.apps/check-deployment scaled

$ kubectl get pods --selector='app=check' -o wide
NAME                                READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
check-deployment-597d756d46-7h8d5   1/1     Running   0          15s    172.17.0.4   minikube   <none>           <none>
check-deployment-597d756d46-fjq85   1/1     Running   0          4m9s   172.17.0.7   minikube   <none>           <none>
check-deployment-597d756d46-lt5zq   1/1     Running   0          4m8s   172.17.0.8   minikube   <none>           <none>

# The new check client also reach out backend-service
$ kubectl logs -f --selector='app=backend'
172.17.0.7:44216 - - [Tue, 05 May 2020 11:02:47 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.8:57826 - - [Tue, 05 May 2020 11:02:50 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.7:44276 - - [Tue, 05 May 2020 11:02:57 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.8:57884 - - [Tue, 05 May 2020 11:03:00 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.7:44340 - - [Tue, 05 May 2020 11:03:07 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.4:51386 - - [Tue, 05 May 2020 11:03:09 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.8:57954 - - [Tue, 05 May 2020 11:03:10 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.7:44404 - - [Tue, 05 May 2020 11:03:17 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.4:51448 - - [Tue, 05 May 2020 11:03:19 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.8:58016 - - [Tue, 05 May 2020 11:03:20 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.7:44468 - - [Tue, 05 May 2020 11:03:27 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.4:51512 - - [Tue, 05 May 2020 11:03:29 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.8:58080 - - [Tue, 05 May 2020 11:03:30 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.7:44530 - - [Tue, 05 May 2020 11:03:37 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.4:51574 - - [Tue, 05 May 2020 11:03:39 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV
172.17.0.8:58142 - - [Tue, 05 May 2020 11:03:40 UTC] "HEAD / HTTP/1.1" curl/7.70.0-DEV

# Clean up resources...
$ kubectl delete -f check-deployment.yml
deployment.apps "check-deployment" deleted

$ kubectl get pods
NAME                                  READY   STATUS    RESTARTS   AGE
backend-deployment-6dd598965c-qn94s   1/1     Running   0          8m36s

$ kubectl delete -f backend-service.yml
service "backend-service" deleted

$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   24h

$ kubectl delete -f backend-deployment.yml
deployment.apps "backend-deployment" deleted

$ kubectl get deployments
No resources found in default namespace.
```