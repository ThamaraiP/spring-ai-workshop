# Kubernetes Deployment Config

Typical manifest structure:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-service
spec:
  replicas: 2
  template:
    spec:
      containers:
        - name: backend
          image: backend-service:latest
          ports:
            - containerPort: 8080
```
Common issues:
- Incorrect image tag.
- Missing ConfigMap references.
