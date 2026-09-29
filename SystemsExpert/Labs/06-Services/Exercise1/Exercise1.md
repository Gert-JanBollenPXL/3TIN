## Exercise 1: Service and DNS

Understand how Services provide stable DNS names for Pod access.

- Create a namespace called svc-lab
- Deploy 2 replicas of nginx:1.30 with labels app: web and tier: frontend
- Create a ClusterIP Service named web that exposes port 80
- Create a test Pod using the curlimages/curl:8.21.0 image with a long sleep command (it has both curl and nslookup)
- From the test Pod, verify:
    - DNS resolution of web returns the Service ClusterIP (not Pod IPs)
    - Both web and web.svc-lab.svc.cluster.local resolve to the same IP
    - HTTP requests to the Service work using the DNS name

result:
- Service DNS name resolves to ClusterIP
- HTTP requests succeed using Service name
- You can explain why using Service DNS is better than Pod IPs

Commands used:

❯ kubectl create namespace svc-lab
```text
namespace/svc-lab created
```
❯ kubectl apply -f deployment.yaml
❯ kubectl apply -f service.yaml
❯ kubectl apply -f test-pod.yaml
```text
deployment.apps/web created
service/web created
pod/curl created
```

❯ kubectl get all -n svc-lab
```text
NAME                       READY   STATUS              RESTARTS   AGE
pod/curl                   0/1     ContainerCreating   0          12s
pod/web-6d598b84d6-rt7w7   0/1     ContainerCreating   0          12s
pod/web-6d598b84d6-spwtx   0/1     ContainerCreating   0          12s

NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/web   ClusterIP   10.43.91.215   <none>        80/TCP    12s

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web   0/2     2            0           13s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/web-6d598b84d6   2         2         0       13s
```
❯ kubectl get svc web -n svc-lab
```text
NAME   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
web    ClusterIP   10.43.91.215   <none>        80/TCP    28s
```
❯ kubectl exec -it curl -n svc-lab -- sh
~ $ nslookup web
```text
Server:         10.43.0.10
Address:        10.43.0.10:53

** server can't find web.cluster.local: NXDOMAIN
** server can't find web.svc.cluster.local: NXDOMAIN
** server can't find web.cluster.local: NXDOMAIN
** server can't find web.svc.cluster.local: NXDOMAIN

Name:   web.svc-lab.svc.cluster.local
Address: 10.43.91.215
```
~ $ nslookup web.svc-lab.svc.cluster.local
```text
Server:         10.43.0.10
Address:        10.43.0.10:53


Name:   web.svc-lab.svc.cluster.local
Address: 10.43.91.215
```
~ $ curl http://web
```text
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy,
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
~ $ curl http://web.svc-lab.svc.cluster.local
```text
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy,
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
~ $ exit

❯ kubectl get pods -n svc-lab -o wide
```text
NAME                   READY   STATUS    RESTARTS   AGE    IP           NODE                       NOMINATED NODE   READINESS GATES
curl                   1/1     Running   0          107s   10.42.0.11   k3d-k3s-default-server-0   <none>           <none>
web-6d598b84d6-rt7w7   1/1     Running   0          107s   10.42.0.10   k3d-k3s-default-server-0   <none>           <none>
web-6d598b84d6-spwtx   1/1     Running   0          107s   10.42.0.9    k3d-k3s-default-server-0   <none>           <none>
```