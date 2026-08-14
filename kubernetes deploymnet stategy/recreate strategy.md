# Recreate Strategy

## What is it?
A Kubernetes deployment strategy where **all old pods are stopped first**, and only then are **new pods created**.

## How it works
1. Stop all old pods completely
2. Wait until they are fully gone
3. Start new pods with the new version
4. Traffic resumes once new pods are ready

## Key Point
Old and new pods **never run together**. This causes **downtime**.

## YAML
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v2
```

## When to use
- Dev/testing environments
- When old and new versions can't run together (e.g. breaking DB changes)

## Downside
- App is completely unavailable during update (downtime)
