Start with a Deployment running 3 replicas of the image nginx:1.29.
Pause the rollout before updating (tiny images finish rolling out faster than you can type), then perform a rolling update to nginx:1.31.

    While paused:
        Check the status of the Deployment with kubectl rollout status --timeout=5s (it reports it is waiting; without a timeout it blocks until you press Ctrl+C).
        List ReplicaSets and identify which one is new with kubectl get rs -n <ns>.
        List Pods and note which are on the old image vs the new one with kubectl get pods -o wide.

Resume the rollout and confirm that:

    All Pods are running the new image.
    Only one ReplicaSet is active (old ReplicaSets may remain at 0 replicas).


Commands used:

❯ kubectl apply -f ./Exercise1/rolling-update-drill.yaml
deployment.apps/rolling-update-drill created

❯ kubectl rollout pause deployment rolling-update-drill -n lab-deploy
deployment.apps/rolling-update-drill paused

❯ kubectl set image deployment/rolling-update-drill rolling-update=nginx:1.31 -n lab-deploy
deployment.apps/rolling-update-drill image updated

❯ kubectl rollout status deployment rolling-update-drill -n lab-deploy --timeout=5s
Waiting for deployment "rolling-update-drill" rollout to finish: 0 out of 3 new replicas have been updated...
error: timed out waiting for the condition

❯ kubectl get rs -n lab-deploy
NAME                              DESIRED   CURRENT   READY   AGE
rolling-update-drill-6744bb864f   3         3         3       2m16s

❯ kubectl get pods -o wide -n lab-deploy
NAME                                    READY   STATUS    RESTARTS   AGE     IP           NODE                       NOMINATED NODE   READINESS GATES
rolling-update-drill-6744bb864f-hd7k9   1/1     Running   0          2m36s   10.42.0.44   k3d-k3s-default-server-0   <none>           <none>
rolling-update-drill-6744bb864f-r85l6   1/1     Running   0          2m36s   10.42.0.46   k3d-k3s-default-server-0   <none>           <none>
rolling-update-drill-6744bb864f-szs77   1/1     Running   0          2m36s   10.42.0.45   k3d-k3s-default-server-0   <none>           <none>

❯ kubectl rollout resume deployment rolling-update-drill -n lab-deploy
deployment.apps/rolling-update-drill resumed

❯ kubectl rollout status deployment rolling-update-drill -n lab-deploy
deployment "rolling-update-drill" successfully rolled out

❯ kubectl get pods -o wide -n lab-deploy
NAME                                    READY   STATUS    RESTARTS   AGE   IP           NODE                       NOMINATED NODE   READINESS GATES
rolling-update-drill-54897f5c58-88pps   1/1     Running   0          31s   10.42.0.48   k3d-k3s-default-server-0   <none>           <none>
rolling-update-drill-54897f5c58-bbs7m   1/1     Running   0          31s   10.42.0.47   k3d-k3s-default-server-0   <none>           <none>
rolling-update-drill-54897f5c58-hvx87   1/1     Running   0          30s   10.42.0.49   k3d-k3s-default-server-0   <none>           <none>

❯ kubectl get rs -n lab-deploy
NAME                              DESIRED   CURRENT   READY   AGE
rolling-update-drill-54897f5c58   3         3         3       38s
rolling-update-drill-6744bb864f   0         0         0       3m45s

❯ kubectl get pods -n lab-deploy -o custom-columns="POD:.metadata.name,IMAGE:.spec.containers[*].image"
POD                                     IMAGE
rolling-update-drill-54897f5c58-88pps   nginx:1.31
rolling-update-drill-54897f5c58-bbs7m   nginx:1.31
rolling-update-drill-54897f5c58-hvx87   nginx:1.31