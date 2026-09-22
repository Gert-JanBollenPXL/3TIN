Delete one Pod managed by the Deployment using kubectl delete pod <pod-name>.
Watch Kubernetes automatically recreate it with kubectl get pods -w.
Check ReplicaSets before and after deletion and confirm the desired replica count is maintained.

Commands used:

❯ kubectl get pods -n dev
NAME                         READY   STATUS    RESTARTS   AGE
dev-deploy-847dc7cbd-7xmbc   1/1     Running   0          7m38s
dev-deploy-847dc7cbd-nkggv   1/1     Running   0          7m38s

❯ kubectl get rs -n dev
NAME                   DESIRED   CURRENT   READY   AGE
dev-deploy-847dc7cbd   2         2         2       7m55s

❯ kubectl delete pod dev-deploy-847dc7cbd-7xmbc
Error from server (NotFound): pods "dev-deploy-847dc7cbd-7xmbc" not found

❯ kubectl delete pod dev-deploy-847dc7cbd-7xmbc -n dev
pod "dev-deploy-847dc7cbd-7xmbc" deleted from dev namespace

❯ kubectl get rs -n dev
NAME                   DESIRED   CURRENT   READY   AGE
dev-deploy-847dc7cbd   2         2         2       8m23s