## Exercise 3: Pod-to-Pod Networking

- Demonstrate Pod-to-Pod communication using Pod IPs.
- Pods:
    - Backend Pod
        - Name: backend
        - Image: busybox:1.37 (ships nc, which can act as a minimal HTTP server)
        - Must start a lightweight HTTP server listening on port 8080.
        - The server should respond with the message: hello-from-backend
    - Frontend Pod
        - Name: frontend
        - Image: curlimages/curl:8.21.0 (client image with curl)
        - Keep the container alive (e.g. a long sleep) so you can test connectivity.

- Retrieve the Pod IP of the backend Pod.
- From inside the frontend Pod, send an HTTP request to:
    - http://<BACKEND_POD_IP>:8080
- Confirm that the response matches exactly:
    - "hello-from-backend"

Commands used:

❯ kubectl apply -f backend-pod.yaml
```text
pod/backend created
```
❯ kubectl apply -f frontend-pod.yaml
```text
pod/frontend created
```
❯ kubectl get pods
```text
NAME       READY   STATUS    RESTARTS   AGE
backend    1/1     Running   0          9s
frontend   1/1     Running   0          4s
```
❯ kubectl get pod backend -o wide
```text
NAME      READY   STATUS    RESTARTS   AGE   IP           NODE                       NOMINATED NODE   READINESS GATES
backend   1/1     Running   0          23s   10.42.0.16   k3d-k3s-default-server-0   <none>           <none>
```
❯ kubectl exec frontend -- curl http://10.42.0.16:8080
```text
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              100     19   0     19   0      0  31353      0                              100     19   0     19   0      0  28023      0                              100     19   0     19   0      0  25503      0                              0
hello-from-backend
```