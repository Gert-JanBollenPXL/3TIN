Understand how Pods reflect different container exit codes.

Specifications:

    Pods:
        Success Pod
            Name: success-pod
            Image: busybox:1.37
            Command: Print all good and exit with status code 0.
        Failure Pod
            Name: fail-pod
            Image: busybox:1.37
            Command: Print oops to standard error and exit with status code 1.
    Observe their STATUS in kubectl get pods.
        The success Pod should reach a phase of Succeeded.
        The failure Pod should reach a phase of Failed.
    Check their logs before deleting them.

Commands used:

❯ kubectl apply -f ./Exercise4/success-pod.yaml
pod/success-pod created

❯ kubectl apply -f ./Exercise4/fail-pod.yaml
pod/fail-pod created

❯ kubectl get pods
NAME         READY   STATUS      RESTARTS   AGE
fail-pod     0/1     Error       0          22s
success-pod   0/1     Completed   0          27s

❯ kubectl get pod success-pod -o jsonpath='{.status.phase}'
❯ kubectl get pod fail-pod -o jsonpath='{.status.phase}'
Succeeded
Failed

❯ kubectl logs success-pod
❯ kubectl logs fail-pod
All good
Oops