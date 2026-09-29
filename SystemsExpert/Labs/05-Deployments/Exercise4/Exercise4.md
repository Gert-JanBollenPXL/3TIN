## Exercise 4: Label and Selector Challenge

- Add a new label tier=frontend to your Deployment Pod template labels.
- Query Pods by that label using kubectl get pods -l app=hello,tier=frontend.
- Explain why changing .spec.selector on an existing Deployment is not allowed and how it would disrupt ownership of Pods.

".spec.selector" is made immutable after the deployment is created. If you could change it, the deployment could suddenly stop matching the pods it originally created. Those pods could become orphaned from that deployment, while the deployment might start matching different pods it wasn't originally intended to manage.

Commands used:

❯ kubectl edit deploy dev-deploy -n dev
```text
deployment.apps/dev-deploy edited
```
❯ kubectl get pods -l app=hello,tier=frontend -n dev
```text
NAME                         READY   STATUS    RESTARTS   AGE
dev-deploy-8b7f777f8-c2p59   1/1     Running   0          23s
dev-deploy-8b7f777f8-t4pp5   1/1     Running   0          24s
```