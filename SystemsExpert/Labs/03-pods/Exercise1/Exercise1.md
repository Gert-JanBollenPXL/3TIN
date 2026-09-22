Exercise 1: One-Shot Pod

Create a Pod that prints a message once and exits.

    Pod name: hello-pod
    Image: busybox:1.37
    Command: Print the exact message Hello Kubernetes
    The Pod must not restart after completion (restartPolicy: Never).
    After the Pod finishes, its status phase should be Succeeded.
    Logs must contain the message exactly as specified.
    Delete the Pod once verified.

Commands used:

❯ kubectl apply -f hello-pod.yaml
pod/hello-pod created

❯ kubectl logs hello-pod
Hello Kubernetes