# Kubernetes ☸️

[![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestration-blue)](https://kubernetes.io/)
[![Go](https://img.shields.io/badge/Go-1.19+-blue)](https://golang.org/)
[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](https://opensource.org/licenses/Apache-2.0)

> **Container Orchestration Platform** - Kubernetes is an open-source container orchestration system for automating software deployment, scaling, and management of containerized applications.

## 🎯 Overview

Kubernetes is a powerful container orchestration platform that:
- **Automates Deployment** - Deploy applications across clusters
- **Manages Scaling** - Automatically scale applications based on demand
- **Ensures High Availability** - Self-healing and fault tolerance
- **Provides Service Discovery** - Automatic service registration and discovery
- **Manages Configuration** - Centralized configuration management
- **Enables Rolling Updates** - Zero-downtime deployments

## 🚀 Key Features

### Core Capabilities
- **Container Orchestration** - Manage containerized applications
- **Service Discovery** - Automatic service registration and load balancing
- **Load Balancing** - Distribute traffic across multiple instances
- **Self-Healing** - Automatically restart failed containers
- **Horizontal Scaling** - Scale applications based on metrics
- **Rolling Updates** - Deploy updates without downtime

### Advanced Features
- **ConfigMaps and Secrets** - Configuration and secret management
- **Persistent Volumes** - Persistent storage for stateful applications
- **Ingress Controllers** - External access to services
- **RBAC** - Role-based access control
- **Network Policies** - Network security and isolation
- **Custom Resources** - Extend Kubernetes with custom resources

## 🏗️ Architecture

### Cluster Components
```
Kubernetes Cluster
├── 🎛️ Control Plane
│   ├── API Server
│   ├── etcd
│   ├── Scheduler
│   ├── Controller Manager
│   └── Cloud Controller Manager
├── 🖥️ Worker Nodes
│   ├── kubelet
│   ├── kube-proxy
│   ├── Container Runtime
│   └── Pods
├── 🌐 Networking
│   ├── CNI Plugins
│   ├── Service Mesh
│   └── Ingress Controllers
└── 💾 Storage
    ├── Persistent Volumes
    ├── Storage Classes
    └── Volume Claims
```

## 🛠️ Installation & Setup

### Minikube (Local Development)
```bash
# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start Minikube
minikube start

# Enable addons
minikube addons enable ingress
minikube addons enable dashboard

# Check status
minikube status
kubectl get nodes
```

### kubeadm (Production Setup)
```bash
# Install kubeadm, kubelet, and kubectl
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Initialize cluster
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Configure kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install CNI plugin
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### Managed Kubernetes (EKS, GKE, AKS)
```bash
# AWS EKS
aws eks create-cluster --name my-cluster --role-arn arn:aws:iam::123456789012:role/EKSClusterRole --resources-vpc-config subnetIds=subnet-12345,subnet-67890

# Google GKE
gcloud container clusters create my-cluster --zone us-central1-a --num-nodes 3

# Azure AKS
az aks create --resource-group myResourceGroup --name myAKSCluster --node-count 3 --enable-addons monitoring --generate-ssh-keys
```

## 📦 Basic Kubernetes Resources

### Pods
```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Deployments
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### Services
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

### ConfigMaps
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://localhost:5432/mydb"
  log_level: "info"
  max_connections: "100"
```

### Secrets
```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: cGFzc3dvcmQ=  # base64 encoded
```

## 🔧 Advanced Kubernetes Resources

### StatefulSets
```yaml
# statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
spec:
  serviceName: mysql-service
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

### DaemonSets
```yaml
# daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-daemonset
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch
        env:
        - name: FLUENT_ELASTICSEARCH_HOST
          value: "elasticsearch.logging.svc.cluster.local"
        - name: FLUENT_ELASTICSEARCH_PORT
          value: "9200"
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

### Jobs and CronJobs
```yaml
# job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-job
spec:
  template:
    spec:
      containers:
      - name: backup
        image: postgres:13
        command:
        - /bin/bash
        - -c
        - |
          pg_dump -h postgres-service -U postgres mydb > /backup/backup.sql
        env:
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        volumeMounts:
        - name: backup-storage
          mountPath: /backup
      volumes:
      - name: backup-storage
        persistentVolumeClaim:
          claimName: backup-pvc
      restartPolicy: Never
  backoffLimit: 3
```

```yaml
# cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-cronjob
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:13
            command:
            - /bin/bash
            - -c
            - |
              pg_dump -h postgres-service -U postgres mydb > /backup/backup-$(date +%Y%m%d).sql
            env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
            volumeMounts:
            - name: backup-storage
              mountPath: /backup
          volumes:
          - name: backup-storage
            persistentVolumeClaim:
              claimName: backup-pvc
          restartPolicy: OnFailure
```

## 🌐 Networking

### Ingress
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

### Network Policies
```yaml
# networkpolicy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-network-policy
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: database
    ports:
    - protocol: TCP
      port: 5432
```

## 💾 Storage

### Persistent Volumes
```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-ssd
  hostPath:
    path: /data/mysql
```

### Persistent Volume Claims
```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast-ssd
```

### Storage Classes
```yaml
# storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  fsType: ext4
reclaimPolicy: Delete
allowVolumeExpansion: true
```

## 🔐 Security

### RBAC (Role-Based Access Control)
```yaml
# rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: app-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: default
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: default
roleRef:
  kind: Role
  name: app-role
  apiGroup: rbac.authorization.k8s.io
```

### Pod Security Policies
```yaml
# psp.yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted-psp
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
```

## 🚀 Advanced Features

### Horizontal Pod Autoscaler
```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
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

### Vertical Pod Autoscaler
```yaml
# vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: app
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 1
        memory: 1Gi
```

### Custom Resources
```yaml
# crd.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              databaseName:
                type: string
              replicas:
                type: integer
                minimum: 1
                maximum: 5
          status:
            type: object
            properties:
              phase:
                type: string
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
```

## 🔧 Best Practices

### Resource Management
1. **Set Resource Limits** - Always set CPU and memory limits
2. **Use Resource Requests** - Specify minimum resource requirements
3. **Monitor Resource Usage** - Track resource consumption
4. **Optimize Images** - Use lightweight base images
5. **Implement Health Checks** - Use liveness and readiness probes

### Security Best Practices
1. **Use RBAC** - Implement role-based access control
2. **Network Policies** - Restrict network traffic
3. **Pod Security** - Use non-root containers
4. **Secrets Management** - Secure sensitive data
5. **Image Scanning** - Scan container images for vulnerabilities

### Deployment Best Practices
1. **Use Deployments** - Don't create pods directly
2. **Implement Rolling Updates** - Zero-downtime deployments
3. **Use ConfigMaps** - Externalize configuration
4. **Health Checks** - Implement proper health checks
5. **Monitoring** - Set up comprehensive monitoring

## 🚀 CI/CD Integration

### GitLab CI
```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  KUBE_NAMESPACE: production

build:
  stage: build
  script:
    - docker build -t $DOCKER_IMAGE .
    - docker push $DOCKER_IMAGE

test:
  stage: test
  script:
    - kubectl apply -f k8s/test/
    - kubectl wait --for=condition=ready pod -l app=test --timeout=300s
    - kubectl exec -it $(kubectl get pods -l app=test -o jsonpath='{.items[0].metadata.name}') -- npm test

deploy:
  stage: deploy
  script:
    - kubectl set image deployment/app-deployment app=$DOCKER_IMAGE
    - kubectl rollout status deployment/app-deployment
  only:
    - main
```

### GitHub Actions
```yaml
# .github/workflows/deploy.yml
name: Deploy to Kubernetes

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Build Docker image
      run: |
        docker build -t myapp:${{ github.sha }} .
        docker tag myapp:${{ github.sha }} myapp:latest
    
    - name: Deploy to Kubernetes
      uses: azure/k8s-deploy@v1
      with:
        manifests: |
          k8s/deployment.yaml
          k8s/service.yaml
        images: |
          myapp:${{ github.sha }}
        namespace: production
```

## 🔍 Monitoring and Debugging

### Monitoring Setup
```yaml
# monitoring.yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
spec:
  selector:
    app: prometheus
  ports:
  - port: 9090
    targetPort: 9090
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: prometheus-config
          mountPath: /etc/prometheus
      volumes:
      - name: prometheus-config
        configMap:
          name: prometheus-config
```

### Debugging Commands
```bash
# View pod logs
kubectl logs pod-name
kubectl logs -f pod-name

# Execute commands in pod
kubectl exec -it pod-name -- /bin/bash

# Describe resources
kubectl describe pod pod-name
kubectl describe service service-name

# Get events
kubectl get events --sort-by=.metadata.creationTimestamp

# Port forward
kubectl port-forward pod-name 8080:80

# Debug deployment
kubectl rollout status deployment/deployment-name
kubectl rollout history deployment/deployment-name
```

## 🐛 Troubleshooting

### Common Issues
1. **Pod Stuck in Pending** - Check resource availability and node capacity
2. **Image Pull Errors** - Verify image name and registry access
3. **Service Not Accessible** - Check service selector and pod labels
4. **Persistent Volume Issues** - Verify storage class and access modes
5. **Network Connectivity** - Check network policies and DNS resolution

### Debugging Tools
```bash
# Check cluster status
kubectl cluster-info
kubectl get nodes
kubectl get pods --all-namespaces

# Check resource usage
kubectl top nodes
kubectl top pods

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Check logs
kubectl logs -l app=myapp --tail=100
```

## 📚 Learning Resources

### Official Documentation
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/)
- [Kubernetes Concepts](https://kubernetes.io/docs/concepts/)

### Community Resources
- [GitHub Repository](https://github.com/kubernetes/kubernetes)
- [Kubernetes Community](https://kubernetes.io/community/)
- [CNCF Landscape](https://landscape.cncf.io/)

## 🔄 Updates and Roadmap

### Recent Updates
- **v1.28.0** - Enhanced security features and performance improvements
- **v1.27.0** - New API features and improved resource management
- **v1.26.0** - Enhanced networking and storage capabilities

### Upcoming Features
- **v1.29.0** - Advanced scheduling and resource optimization
- **v1.30.0** - Enhanced security and compliance features
- **v1.31.0** - Improved developer experience and tooling

## 🤝 Contributing

### How to Contribute
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests and documentation
5. Submit a pull request

### Contribution Guidelines
- Follow Go coding standards
- Write comprehensive tests
- Update documentation
- Ensure backward compatibility

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](https://github.com/kubernetes/kubernetes/blob/master/LICENSE) file for details.

## 🙏 Acknowledgments

- **Google** for originally creating Kubernetes
- **CNCF** for maintaining and evolving the project
- **Open Source Community** for contributions and feedback
- **Users** for providing real-world use cases and improvements

---

<div align="center">

**🌟 Star this repository if you find it helpful!**

[⬆ Back to Top](#kubernetes-)

</div>
