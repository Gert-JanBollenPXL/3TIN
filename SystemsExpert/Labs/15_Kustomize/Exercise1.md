## Exercise 1: add a staging environment

Add a staging environment between dev and prod:
- namespace shop-staging, 2 replicas, podinfo 6.15.0
- the message Hello from staging
- host staging.shop.example.com
- the ha component

Tasks:
- Create overlays/staging, using the base and the component.
    - Copy nothing from the base.
- Compare its build with prod's.
    - Check that you see only the differences you intended.
- Apply it, and call it through the Ingress.
- Check how many disruptions the PDB allows in staging, and explain the number.

```txt
diff <(kustomize build overlays/staging) <(kustomize build overlays/prod)
``

The differences: namespace, environment label, message and ConfigMap hash, replicas (2 against 3), prod's resources patch, and the host. The image is the same: both run 6.15.0 after Step 6.

```bash
kubectl apply -k overlays/staging
kubectl rollout status deployment/web -n shop-staging
kubectl get pdb -n shop-staging
curl -s -H "Host: staging.shop.example.com" http://localhost:8080 | grep -E '"version"|"message"'
```
``txt
NAME   MIN AVAILABLE   MAX UNAVAILABLE   ALLOWED DISRUPTIONS   AGE
web    1               N/A               1                     3s
  "version": "6.15.0",
  "message": "Hello from staging",
``

The PDB allows 1 disruption: 2 healthy Pods, and minAvailable: 1. The component is the same file for staging and prod; the result depends on the replicas of each environment.