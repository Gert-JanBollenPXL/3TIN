## Challenge 2: configuration validation

Create a Pod that validates its configuration on startup and fails gracefully if required configuration is missing.

Requirements:
- Create a ConfigMap named validated-config with:
    - DB_HOST: "database.local"
    - DB_PORT: "5432"
    - DB_NAME: "myapp"
    - CACHE_HOST: "redis.local"
    - APP_ENV: "production"
    - Do not include CACHE_PORT: the Pod supplies a default for it via env, so the validator should still see a value. The truly missing optional config in this challenge is API_KEY.

- Create a Secret named validated-secrets with:
    - DB_USER: "appuser"
    - DB_PASSWORD: "secretpass"
    - Do not include API_KEY (to test optional secret handling).

- Create a Pod named config-validator that:
    - Uses busybox:1.37 image
    - Sets restartPolicy: Never
    - Loads ConfigMap and Secret using envFrom with optional: true
    - Sets default values for missing optional configs, via env: CACHE_PORT "6379" and LOG_LEVEL "info".

- Implement validation logic in the container that checks:
    - Required configs (must exist or validation fails): DB_HOST, DB_PORT, DB_NAME and APP_ENV from the ConfigMap, DB_USER and DB_PASSWORD from the Secret.
    - Optional configs (warn if missing but do not fail): CACHE_HOST, CACHE_PORT and LOG_LEVEL from the ConfigMap, API_KEY from the Secret.
    - Values: DB_PORT is a valid number (digits only), and APP_ENV is one of development, staging, production.

It prints one line per check:
| Case | Output |
|---|---|
| Required config present | `OK: <config> = <value>` |
| Secret present | `OK: <config> is set (length: <length>)` |
| Optional config missing | `WARNING: Optional config not set: <config> (using defaults)` |
| Required config missing | `ERROR: Required config missing: <config>` |
| Invalid value | `ERROR: <config> is not a valid <type>: <value>` |

Exit behavior:
- If all validations pass: print "VALIDATION PASSED" and sleep 3600
- If any validation fails: print "VALIDATION FAILED" and exit 1

Test the validation by:
- First run with the configs as specified (should pass; expect a warning for API_KEY, while CACHE_PORT is filled by the env default)
- Then remove DB_NAME from the live ConfigMap and recreate only the Pod (should fail). If you re-apply a file that also contains the ConfigMap, you restore DB_NAME and the failure never happens.
- Verify the pod exits with error when required config is missing

Success Criteria:
- Pod successfully validates all required configurations
- Missing optional configurations produce warnings but don't fail
- Missing required configurations cause pod to exit with error
- Invalid configuration values are detected and reported
- Secrets are validated without exposing their values

Commands used:

❯ kubectl apply -f validated-config.yaml
❯ kubectl apply -f validated-secrets.yaml
❯ kubectl apply -f config-validator.yaml
```text
configmap/validated-config created
secret/validated-secrets created
pod/config-validator created
```

❯ kubectl logs config-validator
```text
=== Configuration Validation ===
OK: DB_HOST = database.local
OK: DB_PORT = 5432
OK: DB_NAME = myapp
OK: APP_ENV = production
OK: DB_USER is set (length: 7)
OK: DB_PASSWORD is set (length: 10)
OK: CACHE_HOST = redis.local
OK: CACHE_PORT = 6379
OK: LOG_LEVEL = info
WARNING: Optional config not set: API_KEY (using defaults)
VALIDATION PASSED
```

❯ kubectl patch configmap validated-config \
  --type=json \
  -p='[{"op":"remove","path":"/data/DB_NAME"}]'
```text
configmap/validated-config patched
```

❯ kubectl delete pod config-validator # (recreate pod)
❯ kubectl apply -f config-validator.yaml # (recreate pod)
```text
pod "config-validator" deleted from default namespace
pod/config-validator created
```

❯ kubectl logs config-validator
```text
=== Configuration Validation ===
OK: DB_HOST = database.local
OK: DB_PORT = 5432
ERROR: Required config missing: DB_NAME
OK: APP_ENV = production
OK: DB_USER is set (length: 7)
OK: DB_PASSWORD is set (length: 10)
OK: CACHE_HOST = redis.local
OK: CACHE_PORT = 6379
OK: LOG_LEVEL = info
WARNING: Optional config not set: API_KEY (using defaults)
VALIDATION FAILED
```

❯ kubectl get pod config-validator -o jsonpath='{.status.containerStatuses[0].state.terminated.exitCode}'
```text
1
```

❯ kubectl apply -f validated-config.yaml # (reset the configmap)
❯ kubectl patch configmap validated-config \
  --type=merge \
  -p '{"data":{"DB_PORT":"invalid"}}'
```text
configmap/validated-config configured
configmap/validated-config patched
```

❯ kubectl delete pod config-validator # (recreate pod)
❯ kubectl apply -f config-validator.yaml # (recreate pod)
```text
pod "config-validator" deleted from default namespace
pod/config-validator created
```

❯ kubectl logs config-validator
```text
=== Configuration Validation ===
OK: DB_HOST = database.local
OK: DB_PORT = invalid
OK: DB_NAME = myapp
OK: APP_ENV = production
OK: DB_USER is set (length: 7)
OK: DB_PASSWORD is set (length: 10)
OK: CACHE_HOST = redis.local
OK: CACHE_PORT = 6379
OK: LOG_LEVEL = info
WARNING: Optional config not set: API_KEY (using defaults)
ERROR: DB_PORT is not a valid number: invalid
VALIDATION FAILED
```

❯ kubectl apply -f validated-config.yaml # (reset the configmap)
❯ kubectl patch configmap validated-config \
  --type=merge \
  -p '{"data":{"APP_ENV":"testing"}}'
```text
configmap/validated-config configured
configmap/validated-config patched
```

❯ kubectl delete pod config-validator # (recreate pod)
❯ kubectl apply -f config-validator.yaml # (recreate pod)
```text
pod "config-validator" deleted from default namespace
pod/config-validator created
```

❯ kubectl logs config-validator
```text
=== Configuration Validation ===
OK: DB_HOST = database.local
OK: DB_PORT = 5432
OK: DB_NAME = myapp
OK: APP_ENV = testing
OK: DB_USER is set (length: 7)
OK: DB_PASSWORD is set (length: 10)
OK: CACHE_HOST = redis.local
OK: CACHE_PORT = 6379
OK: LOG_LEVEL = info
WARNING: Optional config not set: API_KEY (using defaults)
ERROR: APP_ENV is not a valid environment: testing
VALIDATION FAILED
```

❯ kubectl apply -f validated-config.yaml # (reset the configmap)
❯ kubectl delete pod config-validator # (recreate pod)
❯ kubectl apply -f config-validator.yaml # (recreate pod)
```text
configmap/validated-config configured
pod "config-validator" deleted from default namespace
pod/config-validator created
```

❯ kubectl logs config-validator
❯ kubectl get pod config-validator
```text
=== Configuration Validation ===
OK: DB_HOST = database.local
OK: DB_PORT = 5432
OK: DB_NAME = myapp
OK: APP_ENV = production
OK: DB_USER is set (length: 7)
OK: DB_PASSWORD is set (length: 10)
OK: CACHE_HOST = redis.local
OK: CACHE_PORT = 6379
OK: LOG_LEVEL = info
WARNING: Optional config not set: API_KEY (using defaults)
VALIDATION PASSED
NAME               READY   STATUS    RESTARTS   AGE
config-validator   1/1     Running   0          6s
```