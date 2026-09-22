Create a Deployment with a non-existing image doesnotexist:999.
Observe:

    Deployment status with kubectl rollout status --timeout=30s (it can never finish; the timeout stops the wait).
    Events from kubectl describe deployment.
    Pod details from kubectl describe pod <pod-name>. Logs may be empty if the container never starts.

Fix the image to a valid tag and observe automatic recovery and a successful rollout.

Commands used:

❯ kubectl apply -f ./Exercise5/failure-deploy.yaml
deployment.apps/failure-deploy created

❯ kubectl rollout status deployment/failure-deploy --timeout=5s
Waiting for deployment "failure-deploy" rollout to finish: 0 of 2 updated replicas are available...
error: timed out waiting for the condition

❯ kubectl get pods
NAME                              READY   STATUS             RESTARTS   AGE
failure-deploy-7cf8f78fb6-hg7c4   0/1     ImagePullBackOff   0          30s
failure-deploy-7cf8f78fb6-rrprj   0/1     ErrImagePull       0          30s

❯ kubectl describe deployment failure-deploy
Name:                   failure-deploy
Namespace:              default
CreationTimestamp:      Tue, 22 Sep 2026 15:58:20 +0200
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=failure
Replicas:               2 desired | 2 updated | 2 total | 0 available | 2 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=failure
  Containers:
   nginx:
    Image:         doesnotexist:999
    Port:          <none>
    Host Port:     <none>
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   failure-deploy-7cf8f78fb6 (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  46s   deployment-controller  Scaled up replica set failure-deploy-7cf8f78fb6 from 0 to 2

❯ kubectl get pods
NAME                              READY   STATUS             RESTARTS   AGE
failure-deploy-7cf8f78fb6-hg7c4   0/1     ImagePullBackOff   0          89s
failure-deploy-7cf8f78fb6-rrprj   0/1     ImagePullBackOff   0          89s

❯ kubectl describe pod failure-deploy-7cf8f78fb6-hg7c4
Name:             failure-deploy-7cf8f78fb6-hg7c4
Namespace:        default
Priority:         0
Service Account:  default
Node:             k3d-k3s-default-server-0/172.19.0.3
Start Time:       Tue, 22 Sep 2026 15:58:21 +0200
Labels:           app=failure
                  pod-template-hash=7cf8f78fb6
Annotations:      <none>
Status:           Pending
IP:               10.42.0.60
IPs:
  IP:           10.42.0.60
Controlled By:  ReplicaSet/failure-deploy-7cf8f78fb6
Containers:
  nginx:
    Container ID:
    Image:          doesnotexist:999
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-59s6f (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-59s6f:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  101s               default-scheduler  Successfully assigned default/failure-deploy-7cf8f78fb6-hg7c4 to k3d-k3s-default-server-0
  Normal   BackOff    18s (x5 over 99s)  kubelet            spec.containers{nginx}: Back-off pulling image "doesnotexist:999"
  Warning  Failed     18s (x5 over 99s)  kubelet            spec.containers{nginx}: Error: ImagePullBackOff
  Normal   Pulling    6s (x4 over 100s)  kubelet            spec.containers{nginx}: Pulling image "doesnotexist:999"
  Warning  Failed     5s (x4 over 99s)   kubelet            spec.containers{nginx}: Failed to pull image "doesnotexist:999": failed to pull and unpack image "docker.io/library/doesnotexist:999": failed to resolve reference "docker.io/library/doesnotexist:999": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     5s (x4 over 99s)   kubelet            spec.containers{nginx}: Error: ErrImagePull

❯ kubectl set image deployment/failure-deploy nginx=nginx:1.29
deployment.apps/failure-deploy image updated

❯ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
failure-deploy-78f9bb666d-kmp4h   1/1     Running   0          8s
failure-deploy-78f9bb666d-ndw7x   1/1     Running   0          8s
