## Challenge 1: multi-environment setup

Create a system that demonstrates environment-specific configuration management across development, staging, and production.

Requirements:

Create three namespaces:
- dev
- staging
- prod

In each namespace, create:
- A ConfigMap named app-config with the keys in the table below.
- A Secret named app-secrets with the keys DB_PASSWORD and API_KEY, with different values in each environment.
    | Key | dev | staging | prod |
|---|---|---|---|
| APP_ENV | development | staging | production |
| LOG_LEVEL | debug | info | error |
| DB_HOST | localhost | staging-db.internal | prod-db-cluster.internal |
| DB_PORT | "5432" | "5432" | "5432" |
| FEATURE_FLAGS | experimental=true,analytics=false | experimental=false,analytics=true | experimental=false,analytics=true |

Create a single Deployment manifest named universal-deployment.yaml that:
- Has the name multi-env-app
- Runs 2 replicas
- Uses the busybox:1.37 image
- Does not specify a namespace (will use current context when applied)
- Loads all ConfigMap keys as environment variables using envFrom
- Loads specific Secret keys as environment variables using env
- Has the container print out the environment name, log level, database host/port, and feature flags
- Verifies that DB_PASSWORD and API_KEY are set (without printing their values)
- Runs a loop that prints [timestamp] <environment> server running... every 30 seconds

Deploy this single manifest to all three namespaces using:
```bash
    kubectl apply -f universal-deployment.yaml -n dev
    kubectl apply -f universal-deployment.yaml -n staging
    kubectl apply -f universal-deployment.yaml -n prod
```

Verify that each deployment shows different configuration values appropriate to its environment

Success Criteria:
- Same deployment YAML works in all environments
- Each environment shows its specific configuration
- Secrets are referenced but never exposed in logs
- All three deployments run successfully with 2 replicas each

Commands used:

❯ kubectl create namespace dev
❯ kubectl create namespace staging
❯ kubectl create namespace prod
```text
namespace/dev created
namespace/staging created
namespace/prod created
```

❯ kubectl apply -f dev-config.yaml
❯ kubectl apply -f dev-secrets.yaml
❯ kubectl apply -f staging-config.yaml
❯ kubectl apply -f staging-secrets.yaml
❯ kubectl apply -f prod-config.yaml
❯ kubectl apply -f prod-secrets.yaml

```text
configmap/app-config created
secret/app-secrets created
configmap/app-config created
secret/app-secrets created
configmap/app-config created
secret/app-secrets created
```

❯ kubectl apply -f universal-deployment.yaml -n dev
❯ kubectl apply -f universal-deployment.yaml -n staging
❯ kubectl apply -f universal-deployment.yaml -n prod
```text
deployment.apps/multi-env-app configured
deployment.apps/multi-env-app configured
deployment.apps/multi-env-app configured
```

❯ kubectl logs -n dev deployment/multi-env-app
❯ kubectl logs -n staging deployment/multi-env-app
❯ kubectl logs -n prod deployment/multi-env-app

```text
Found 2 pods, using pod/multi-env-app-68dffbbc6-7mdnr
=== Application Starting ===
Environment: development
Log Level: debug
DB Host: localhost
Port: 5432
Feature Flag: experimental=true,analytics=false

=== Security Check ===
DB Password is set: YES
API Key is set: YES

=== Server Running ===
[10:53:54] Server running in development...

Found 2 pods, using pod/multi-env-app-68dffbbc6-dvb4c
=== Application Starting ===
Environment: staging
Log Level: info
DB Host: staging-db.internal
Port: 5432
Feature Flag: experimental=false,analytics=true

=== Security Check ===
DB Password is set: YES
API Key is set: YES

=== Server Running ===
[10:53:54] Server running in staging...

Found 2 pods, using pod/multi-env-app-68dffbbc6-bvsk4
=== Application Starting ===
Environment: production
Log Level: error
DB Host: prod-db-cluster.internal
Port: 5432
Feature Flag: experimental=false,analytics=true

=== Security Check ===
DB Password is set: YES
API Key is set: YES

=== Server Running ===
[10:53:54] Server running in production...
```
