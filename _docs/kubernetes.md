---
title: Kubernetes
description: Deploy Eltrix on Kubernetes
nav_title: Kubernetes
category: deployment
order: 5
---

Eltrix is designed for Kubernetes from the ground up. This guide covers deploying Eltrix on Kubernetes clusters.

## Prerequisites

- Kubernetes 1.25+
- kubectl configured
- Helm 3 (optional, for Helm chart)
- An ingress controller (nginx-ingress, Traefik, etc.)
- cert-manager (for TLS certificates)

## Quick Start with Kubectl

### Namespace and Secrets

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: eltrix
---
# secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: eltrix-secrets
  namespace: eltrix
type: Opaque
stringData:
  secret-key: "your-secret-key-here"
  database-url: "postgres://eltrix:password@postgres-service/eltrix"
```

### Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eltrix
  namespace: eltrix
spec:
  replicas: 2
  selector:
    matchLabels:
      app: eltrix
  template:
    metadata:
      labels:
        app: eltrix
    spec:
      containers:
        - name: eltrix
          image: ghcr.io/eltrix/eltrix:latest
          ports:
            - containerPort: 8008
              name: client
            - containerPort: 8448
              name: federation
          env:
            - name: ELTRIX_SERVER_NAME
              value: "matrix.example.com"
            - name: ELTRIX_SECRET_KEY
              valueFrom:
                secretKeyRef:
                  name: eltrix-secrets
                  key: secret-key
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: eltrix-secrets
                  key: database-url
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "2Gi"
              cpu: "2"
          livenessProbe:
            httpGet:
              path: /health/live
              port: 8008
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 8008
            initialDelaySeconds: 5
            periodSeconds: 5
          volumeMounts:
            - name: data
              mountPath: /var/lib/eltrix
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: eltrix-data
```

### Service

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: eltrix
  namespace: eltrix
spec:
  selector:
    app: eltrix
  ports:
    - name: client
      port: 8008
      targetPort: 8008
    - name: federation
      port: 8448
      targetPort: 8448
```

### Ingress

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: eltrix
  namespace: eltrix
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - matrix.example.com
      secretName: eltrix-tls
  rules:
    - host: matrix.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: eltrix
                port:
                  number: 8008
```

### Persistent Volume Claim

```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: eltrix-data
  namespace: eltrix
spec:
  accessModes:
    - ReadWriteMany  # Required for multi-replica
  resources:
    requests:
      storage: 50Gi
  storageClassName: standard
```

## Helm Chart

For easier deployment, use the official Helm chart:

```bash
# Add the Eltrix Helm repository
helm repo add eltrix https://charts.eltrix.dev
helm repo update

# Install with custom values
helm install eltrix eltrix/eltrix \
  --namespace eltrix \
  --create-namespace \
  --set serverName=matrix.example.com \
  --set secretKey=$(openssl rand -base64 32) \
  --set postgresql.enabled=true
```

### Custom Values

```yaml
# values.yaml
serverName: matrix.example.com

replicaCount: 3

image:
  repository: ghcr.io/eltrix/eltrix
  tag: latest
  pullPolicy: IfNotPresent

resources:
  requests:
    memory: "1Gi"
    cpu: "500m"
  limits:
    memory: "4Gi"
    cpu: "4"

postgresql:
  enabled: true
  auth:
    database: eltrix
    postgresPassword: "your-postgres-password"
  primary:
    persistence:
      size: 50Gi

persistence:
  enabled: true
  size: 100Gi
  storageClass: standard
  accessMode: ReadWriteMany

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: matrix.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: eltrix-tls
      hosts:
        - matrix.example.com

config:
  logLevel: info
  logFormat: json
  federationEnabled: true
  registrationEnabled: false
```

## Horizontal Pod Autoscaling

Scale based on CPU/memory:

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: eltrix
  namespace: eltrix
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: eltrix
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

## Pod Disruption Budget

Ensure availability during updates:

```yaml
# pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: eltrix
  namespace: eltrix
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: eltrix
```

## Federation Service

For federation, you need a separate service for port 8448:

```yaml
# federation-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: eltrix-federation
  namespace: eltrix
  annotations:
    # For AWS NLB
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
spec:
  type: LoadBalancer
  selector:
    app: eltrix
  ports:
    - name: federation
      port: 8448
      targetPort: 8448
```

## Monitoring

### ServiceMonitor for Prometheus

```yaml
# servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: eltrix
  namespace: eltrix
spec:
  selector:
    matchLabels:
      app: eltrix
  endpoints:
    - port: client
      path: /metrics
      interval: 30s
```

## Next Steps

- [Scaling Guide](/docs/scaling/) - Advanced scaling strategies
- [Federation Guide](/docs/federation/) - Configure federation
- [Configuration](/docs/configuration/) - All configuration options
