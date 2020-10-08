# sample-ol-stack-monitoring

This repository contains a custom Dockerfile and deployment yaml which can be used to build and deploy an Open Liberty stack application and connect it to Prometheus for monitoring.

## quick-start-security

The default quick-start-security configuration is removed and replaced with a configuration that has environment variables for the username and password.

```xml
<quickStartSecurity userName="${stack.username}" userPassword="${stack.password}" />
```

## Secret Injection

To set the username and password environment variables, the app is injecting values from a pre-defined Secret called "mySecret". This must exist in the cluster namespace before deploying.

```yaml
env:
  - name: STACK_USERNAME
  valueFrom:
    secretKeyRef:
      key: username
      name: mySecret
  - name: STACK_PASSWORD
  valueFrom:
    secretKeyRef:
      key: password
      name: mySecret
```

<!-- ## HTTPS Port Configuration

You will notice that the app-deploy.yaml is configured to use port 9443 rather than port 9080.

```yaml
port: 9443
```
-->

## Service Monitor

To allow for easy detection from Prometheus, this app defines a Service Monitor that will use the credentials from "mySecret" and the HTTPS port to connect to the deployment's /metrics endpoint.

```yaml
monitoring:
  endpoints:
  - basicAuth:
      password:
        key: password
        name: mysecret
      username:
        key: username
        name: mysecret
    interval: 5s
    port: HTTPS
    scheme: HTTPS
    tlsConfig:
      insecureSkipVerify: true
  labels:
    app-monitoring: ''
```

Prometheus may need to be configured to watch the namespace where this app is deployed, and the namespace will need to have the label "app-monitoring."