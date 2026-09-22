Exercise 2: Multi-Container Pod with Shared Volume

Create a Pod with two containers sharing data via a volume.

    Pod name: nginx-with-sidecar
    Containers:
        Main container
            Name: nginx
            Image: nginx:1.30-alpine
            Mount path: /shared
        Sidecar container
            Name: writer
            Image: busybox:1.37
            Behavior: Append the current date and time to a file every 10 seconds. (just use a bash while command)
            Log file path: /shared/time.log
    Shared volume:
        Name: shared
        Type: emptyDir
        Mount path: /shared in both containers
    Confirm the Pod reaches Running status.
    Check /shared/time.log from both containers - the contents must match.
    Ensure new timestamps are being appended continuously.

Commands used:

❯ kubectl apply -f nginx-with-sidecar.yaml
pod/nginx-with-sidecar created

❯ kubectl exec -it nginx-with-sidecar -c nginx -- sh
/ # cat /shared/time.log

❯ kubectl exec -it nginx-with-sidecar -c writer -- sh
/ # cat /shared/time.log