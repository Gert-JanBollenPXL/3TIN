Create two namespaces: dev and prod.
Use the same Deployment spec in both, but ensure the correct namespace is targeted. If the manifest sets metadata.namespace, create a copy for each namespace.
In dev, set replicas to 2 and image to nginx:1.31 (mainline).
In prod, set replicas to 5 and image to nginx:1.30 (stable, pinned).
Compare how Deployments differ between namespaces using kubectl get deploy -n dev and -n prod.

Commands used:

❯ kubectl create namespace dev
namespace/dev created

❯ kubectl create namespace prod
namespace/prod created

❯ kubectl apply -f ./Exercise2/dev-deploy.yaml
deployment.apps/dev-deploy created

❯ kubectl apply -f ./Exercise2/prod-deploy.yaml
deployment.apps/prod-deploy created

❯ kubectl get deploy -n prod -o wide
NAME          READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS    IMAGES       SELECTOR
prod-deploy   5/5     5            5           64s   prod-deploy   nginx:1.30   app=prod

❯ kubectl get deploy -n dev -o wide
NAME         READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES       SELECTOR
dev-deploy   2/2     2            2           98s   dev-deploy   nginx:1.31   app=dev