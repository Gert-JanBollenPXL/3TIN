## Exercise 1: weighted traffic splitting (canary)

Tasks:
- Deploy two versions of an echo app in default:
    - echo-v1: hashicorp/http-echo:1.0 with -text=v1
    - echo-v2: hashicorp/http-echo:1.0 with -text=v2
    - one Service per Deployment (port 80, targetPort 5678)

- Create an HTTPRoute on hostname canary.example.com with one rule and two backendRefs, weighted 90 (v1) and 10 (v2).

- Send 50 requests and count the split:
```bash
for i in $(seq 1 50); do curl -s -H "Host: canary.example.com" http://localhost/; done | sort | uniq -c
```

Passing result:
- roughly 45 v1 and 5 v2 (the split is statistical).
- Then set the weights to 0/100 and confirm all traffic goes to v2.

Commands used:

❯ k3d cluster create gw-lab --image rancher/k3s:v1.36.3-k3s1 \
  --k3s-arg "--disable=traefik@server:0" \
  --port "80:80@loadbalancer" --port "443:443@loadbalancer"
❯ kubectl get nodes

```text
INFO[0000] portmapping '443:443' targets the loadbalancer: defaulting to [servers:*:proxy agents:*:proxy]
INFO[0000] portmapping '80:80' targets the loadbalancer: defaulting to [servers:*:proxy agents:*:proxy]
INFO[0000] Prep: Network
INFO[0000] Created network 'k3d-gw-lab'
INFO[0000] Created image volume k3d-gw-lab-images
INFO[0000] Starting new tools node...
INFO[0000] Starting node 'k3d-gw-lab-tools'
INFO[0001] Creating node 'k3d-gw-lab-server-0'
INFO[0001] Creating LoadBalancer 'k3d-gw-lab-serverlb'
INFO[0001] Using the k3d-tools node to gather environment information
INFO[0001] HostIP: using network gateway 172.19.0.1 address
INFO[0001] Starting cluster 'gw-lab'
INFO[0001] Starting servers...
INFO[0001] Starting node 'k3d-gw-lab-server-0'
INFO[0007] All agents already running.
INFO[0007] Starting helpers...
INFO[0007] Starting node 'k3d-gw-lab-serverlb'
INFO[0014] Injecting records for hostAliases (incl. host.k3d.internal) and for 2 network members into CoreDNS configmap...
INFO[0016] Cluster 'gw-lab' created successfully!
INFO[0016] You can now use it like this:
kubectl cluster-info
NAME                  STATUS   ROLES           AGE   VERSION
k3d-gw-lab-server-0   Ready    control-plane   9s    v1.36.3+k3s1
```

❯ helm install eg oci://registry-1.docker.io/envoyproxy/gateway-helm --version v1.9.0 -n envoy-gateway-system --create-namespace
❯ kubectl wait -n envoy-gateway-system deployment/envoy-gateway --for=condition=Available --timeout=5m
```text
Pulled: registry-1.docker.io/envoyproxy/gateway-helm:v1.9.0
Digest: sha256:06e7c26e50d40f0b98d6d1243a3c8dd094464c6099df727216876c19401ffe5f
NAME: eg
LAST DEPLOYED: Tue Oct  6 16:09:30 2026
NAMESPACE: envoy-gateway-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
**************************************************************************
*** PLEASE BE PATIENT: Envoy Gateway may take a few minutes to install ***
**************************************************************************

Envoy Gateway is an open source project for managing Envoy Proxy as a standalone or Kubernetes-based application gateway.

Thank you for installing Envoy Gateway! 🎉

Your release is named: eg. 🎉

Your release is in namespace: envoy-gateway-system. 🎉

To learn more about the release, try:

  $ helm status eg -n envoy-gateway-system
  $ helm get all eg -n envoy-gateway-system

To have a quickstart of Envoy Gateway, please refer to https://gateway.envoyproxy.io/latest/tasks/quickstart.

To get more details, please visit https://gateway.envoyproxy.io and https://github.com/envoyproxy/gateway.
deployment.apps/envoy-gateway condition met
```

❯ kubectl apply -n default -f https://github.com/envoyproxy/gateway/releases/download/v1.9.0/quickstart.yaml
❯ kubectl wait --for=condition=Programmed gateway/eg --timeout=3m
❯ kubectl get gateway,httproute -A
```text
gatewayclass.gateway.networking.k8s.io/eg created
gateway.gateway.networking.k8s.io/eg created
serviceaccount/backend created
service/backend created
deployment.apps/backend created
httproute.gateway.networking.k8s.io/backend created
gateway.gateway.networking.k8s.io/eg condition met
NAMESPACE   NAME                                   CLASS   ADDRESS      PROGRAMMED   AGE
default     gateway.gateway.networking.k8s.io/eg   eg      172.19.0.2   True         12s

NAMESPACE   NAME                                               HOSTNAMES                AGE
default     httproute.gateway.networking.k8s.io/backend        ["www.example.com"]      13s
default     httproute.gateway.networking.k8s.io/canary-route   ["canary.example.com"]   3m2s
```

❯ kubectl apply -f ../lab-nginx.yaml
```text
deployment.apps/lab-nginx created
service/lab-nginx-svc created
httproute.gateway.networking.k8s.io/lab-nginx-route created
```

❯ kubectl apply -f canary.yaml
❯ kubectl rollout status deploy/echo-v1
❯ kubectl rollout status deploy/echo-v2
```text
deployment.apps/echo-v1 created
deployment.apps/echo-v2 created
service/echo-v1 created
service/echo-v2 created
httproute.gateway.networking.k8s.io/canary-route created
Waiting for deployment "echo-v1" rollout to finish: 0 of 1 updated replicas are available...
deployment "echo-v1" successfully rolled out
deployment "echo-v2" successfully rolled out
```

❯ for i in $(seq 1 50); do curl -s -H "Host: canary.example.com" http://localhost/; done | sort | uniq -c
```text
     45 v1
      5 v2
```

Change the weights of v1 and v2 to 0/100, then reapply the deployment

❯ kubectl apply -f canary.yaml
❯ for i in $(seq 1 50); do curl -s -H "Host: canary.example.com" http://localhost/; done | sort | uniq -c
```text
deployment.apps/echo-v1 unchanged
deployment.apps/echo-v2 unchanged
service/echo-v1 unchanged
service/echo-v2 unchanged
httproute.gateway.networking.k8s.io/canary-route configured
     50 v2
```