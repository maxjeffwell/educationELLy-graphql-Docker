# Kubernetes Deployment Guide

This guide explains how to deploy the educationELLy GraphQL application to Kubernetes, with specific instructions for Linode Kubernetes Engine (LKE).

## Table of Contents

- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Linode Setup](#linode-setup)
- [Configuration](#configuration)
- [Deployment](#deployment)
- [Monitoring and Management](#monitoring-and-management)
- [Scaling](#scaling)
- [SSL/TLS Setup](#ssltls-setup)
- [Troubleshooting](#troubleshooting)
- [Production Checklist](#production-checklist)

## Prerequisites

### Required Tools

```bash
# kubectl (Kubernetes CLI)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Verify installation
kubectl version --client

# Linode CLI (optional but recommended)
pip3 install linode-cli
linode-cli configure
```

### Required Resources

- Kubernetes cluster (Linode LKE or any K8s provider)
- `kubectl` configured to access your cluster
- Domain name pointing to your cluster
- MongoDB Atlas or external MongoDB instance
- Docker Hub images (already published)

## Quick Start

```bash
# 1. Configure secrets
cp k8s/secrets.yaml.example k8s/secrets.yaml
# Edit k8s/secrets.yaml with your actual base64-encoded values

# 2. Update configuration
vim k8s/configmap.yaml  # Update ALLOWED_ORIGINS with your domain
vim k8s/ingress.yaml    # Update domain names

# 3. Deploy everything
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secrets.yaml
kubectl apply -f k8s/server-deployment.yaml
kubectl apply -f k8s/server-service.yaml
kubectl apply -f k8s/client-deployment.yaml
kubectl apply -f k8s/client-service.yaml
kubectl apply -f k8s/ingress.yaml
kubectl apply -f k8s/hpa.yaml

# 4. Check status
kubectl get all -n educationelly
```

## Linode Setup

### 1. Create Linode Kubernetes Cluster

#### Via Linode Cloud Manager (Web UI)

1. Log in to [Linode Cloud Manager](https://cloud.linode.com/)
2. Click **Kubernetes** in the left sidebar
3. Click **Create Cluster**
4. Configure:
   - **Cluster Label**: `educationelly-prod`
   - **Region**: Choose closest to your users (e.g., `us-east`)
   - **Kubernetes Version**: Latest stable (1.28+)
   - **Add Node Pools**:
     - **Dedicated 4GB**: 2 nodes (minimum for production)
     - Or **Dedicated 8GB**: 2 nodes (better performance)
5. Click **Create Cluster**
6. Wait 5-10 minutes for cluster provisioning

#### Via Linode CLI

```bash
# List available regions
linode-cli regions list

# List available Kubernetes versions
linode-cli lke versions-list

# Create cluster
linode-cli lke cluster-create \
  --label educationelly-prod \
  --region us-east \
  --k8s_version 1.28 \
  --node_pools.type g6-dedicated-4 \
  --node_pools.count 2
```

### 2. Download kubeconfig

#### Via Web UI

1. Go to your cluster in Linode Cloud Manager
2. Click **Download kubeconfig**
3. Save to `~/.kube/config` or merge with existing config

#### Via CLI

```bash
# Get cluster ID
linode-cli lke clusters-list

# Download kubeconfig
linode-cli lke kubeconfig-view <CLUSTER_ID> --text | base64 -d > ~/.kube/config-linode

# Use the config
export KUBECONFIG=~/.kube/config-linode

# Or merge with existing config
KUBECONFIG=~/.kube/config:~/.kube/config-linode kubectl config view --flatten > ~/.kube/config.new
mv ~/.kube/config.new ~/.kube/config
```

### 3. Verify Cluster Access

```bash
# Check cluster info
kubectl cluster-info

# List nodes
kubectl get nodes

# Should show 2+ nodes in Ready state
```

### 4. Install Nginx Ingress Controller

Linode LKE doesn't include an ingress controller by default.

```bash
# Install Nginx Ingress Controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.4/deploy/static/provider/cloud/deploy.yaml

# Wait for LoadBalancer to be assigned
kubectl get svc -n ingress-nginx ingress-nginx-controller --watch

# Get external IP (this is your cluster's public IP)
kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

**Important**: Point your domain's A record to this IP address.

### 5. Install cert-manager (Optional - for SSL/TLS)

```bash
# Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.2/cert-manager.yaml

# Wait for cert-manager pods to be ready
kubectl get pods -n cert-manager --watch

# Create ClusterIssuer for Let's Encrypt
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com  # Change this!
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
```

## Configuration

### 1. Create Secrets

Secrets contain sensitive data like database credentials and JWT secrets.

```bash
# Copy template
cp k8s/secrets.yaml.example k8s/secrets.yaml

# Generate base64-encoded values
echo -n "mongodb+srv://user:password@cluster.mongodb.net/database" | base64
echo -n "your-super-secret-jwt-key-min-64-chars" | base64

# Edit secrets.yaml with the base64 values
vim k8s/secrets.yaml

# IMPORTANT: Add secrets.yaml to .gitignore
echo "k8s/secrets.yaml" >> .gitignore
```

### 2. Update ConfigMap

Edit `k8s/configmap.yaml`:

```yaml
data:
  ALLOWED_ORIGINS: "https://yourdomain.com,https://www.yourdomain.com"
```

### 3. Update Ingress

Edit `k8s/ingress.yaml`:

```yaml
spec:
  tls:
  - hosts:
    - yourdomain.com           # Change to your domain
    - www.yourdomain.com       # Change to your domain
  rules:
  - host: yourdomain.com       # Change to your domain
  - host: www.yourdomain.com   # Change to your domain
```

Also update the cert-manager email:
```yaml
metadata:
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"  # Ensure this matches
```

## Deployment

### Deploy Application

```bash
# 1. Create namespace
kubectl apply -f k8s/namespace.yaml

# 2. Create ConfigMap
kubectl apply -f k8s/configmap.yaml

# 3. Create Secrets
kubectl apply -f k8s/secrets.yaml

# 4. Deploy server
kubectl apply -f k8s/server-deployment.yaml
kubectl apply -f k8s/server-service.yaml

# 5. Deploy client
kubectl apply -f k8s/client-deployment.yaml
kubectl apply -f k8s/client-service.yaml

# 6. Create Ingress
kubectl apply -f k8s/ingress.yaml

# 7. Enable autoscaling (optional)
kubectl apply -f k8s/hpa.yaml

# 8. Verify deployment
kubectl get all -n educationelly
```

### Verify Deployment

```bash
# Check pods are running
kubectl get pods -n educationelly

# Should show:
# educationelly-server-xxx   1/1   Running
# educationelly-client-xxx   1/1   Running

# Check services
kubectl get svc -n educationelly

# Check ingress
kubectl get ingress -n educationelly

# View logs
kubectl logs -n educationelly deployment/educationelly-server
kubectl logs -n educationelly deployment/educationelly-client

# Check pod health
kubectl describe pod -n educationelly <pod-name>
```

## Monitoring and Management

### View Application Logs

```bash
# Stream server logs
kubectl logs -f -n educationelly deployment/educationelly-server

# Stream client logs
kubectl logs -f -n educationelly deployment/educationelly-client

# View logs from all pods
kubectl logs -n educationelly -l app=educationelly-graphql --all-containers=true

# View logs from specific time
kubectl logs -n educationelly deployment/educationelly-server --since=1h
```

### Check Application Health

```bash
# Check pod status
kubectl get pods -n educationelly -o wide

# Check deployment status
kubectl rollout status deployment/educationelly-server -n educationelly
kubectl rollout status deployment/educationelly-client -n educationelly

# Describe resources for detailed info
kubectl describe deployment educationelly-server -n educationelly
kubectl describe pod <pod-name> -n educationelly

# Check resource usage
kubectl top pods -n educationelly
kubectl top nodes
```

### Access Application

```bash
# Get ingress IP
kubectl get ingress -n educationelly

# Test endpoints
curl https://yourdomain.com/health
curl https://yourdomain.com/api/health
curl https://yourdomain.com/graphql -X POST -H "Content-Type: application/json" -d '{"query":"{ __typename }"}'
```

### Update Application

```bash
# Update to new version
kubectl set image deployment/educationelly-server server=maxjeffwell/educationelly-graphql-server:v1.1.0 -n educationelly
kubectl set image deployment/educationelly-client client=maxjeffwell/educationelly-graphql-client:v1.1.0 -n educationelly

# Watch rollout
kubectl rollout status deployment/educationelly-server -n educationelly

# Rollback if needed
kubectl rollout undo deployment/educationelly-server -n educationelly

# Check rollout history
kubectl rollout history deployment/educationelly-server -n educationelly
```

## Scaling

### Manual Scaling

```bash
# Scale server deployment
kubectl scale deployment educationelly-server --replicas=5 -n educationelly

# Scale client deployment
kubectl scale deployment educationelly-client --replicas=3 -n educationelly

# Verify
kubectl get deployment -n educationelly
```

### Autoscaling (HPA)

HPA is already configured in `k8s/hpa.yaml`.

```bash
# Check HPA status
kubectl get hpa -n educationelly

# Describe HPA
kubectl describe hpa educationelly-server-hpa -n educationelly

# View autoscaling events
kubectl get events -n educationelly --field-selector involvedObject.name=educationelly-server-hpa

# Modify HPA
kubectl edit hpa educationelly-server-hpa -n educationelly
```

**HPA Configuration:**
- **Server**: 2-10 replicas, scales at 70% CPU or 80% memory
- **Client**: 2-5 replicas, scales at 70% CPU or 80% memory

## SSL/TLS Setup

SSL/TLS is automatically handled by cert-manager if configured.

### Verify Certificate

```bash
# Check certificate status
kubectl get certificate -n educationelly

# Describe certificate
kubectl describe certificate educationelly-tls -n educationelly

# Check cert-manager logs
kubectl logs -n cert-manager deployment/cert-manager

# Manual certificate request (if needed)
kubectl delete certificate educationelly-tls -n educationelly
kubectl apply -f k8s/ingress.yaml
```

### Troubleshoot SSL Issues

```bash
# Check certificate request
kubectl get certificaterequest -n educationelly

# Check challenges
kubectl get challenges -n educationelly

# View detailed challenge info
kubectl describe challenge <challenge-name> -n educationelly
```

## Troubleshooting

### Pods Not Starting

```bash
# Check pod status
kubectl describe pod <pod-name> -n educationelly

# Common issues:
# - ImagePullBackOff: Check image name and Docker Hub access
# - CrashLoopBackOff: Check logs for application errors
# - Pending: Check resource availability

# Check events
kubectl get events -n educationelly --sort-by='.lastTimestamp'
```

### Database Connection Issues

```bash
# Check secrets are correctly set
kubectl get secret educationelly-secrets -n educationelly -o yaml

# Decode and verify MongoDB URI
kubectl get secret educationelly-secrets -n educationelly -o jsonpath='{.data.MONGODB_URI}' | base64 -d

# Check server logs for connection errors
kubectl logs -n educationelly deployment/educationelly-server | grep -i mongo
```

### Ingress Not Working

```bash
# Check ingress status
kubectl describe ingress educationelly-ingress -n educationelly

# Check ingress controller logs
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller

# Verify DNS points to LoadBalancer IP
nslookup yourdomain.com

# Test without Ingress (port-forward)
kubectl port-forward -n educationelly svc/educationelly-client 8080:80
# Access http://localhost:8080
```

### High Memory/CPU Usage

```bash
# Check resource usage
kubectl top pods -n educationelly

# Check HPA status
kubectl get hpa -n educationelly

# Increase resource limits
kubectl edit deployment educationelly-server -n educationelly
# Update resources.limits
```

## Production Checklist

### Before Deployment

- [ ] Domain DNS pointing to LoadBalancer IP
- [ ] MongoDB Atlas allows K8s cluster IPs
- [ ] Secrets created with strong values
- [ ] ConfigMap updated with production domain
- [ ] Ingress configured with correct domain
- [ ] Resource limits set appropriately
- [ ] HPA enabled and configured

### Security

- [ ] Use strong JWT secret (64+ characters)
- [ ] MongoDB uses strong password
- [ ] Secrets not committed to version control
- [ ] CORS configured for production domain only
- [ ] Network policies configured (optional)
- [ ] Pod security policies enabled (optional)

### Monitoring

- [ ] Set up logging (e.g., ELK, Loki)
- [ ] Set up monitoring (e.g., Prometheus, Grafana)
- [ ] Configure alerts for pod failures
- [ ] Configure alerts for high resource usage
- [ ] Set up uptime monitoring

### Backup

- [ ] MongoDB automated backups configured
- [ ] Kubernetes manifests in version control
- [ ] Secrets backed up securely (not in git!)
- [ ] Disaster recovery plan documented

## Cost Optimization (Linode)

### Recommended Node Configuration

**Development/Staging:**
- 2x Linode 4GB ($36/month each) = **$72/month**

**Production:**
- 3x Linode 8GB ($72/month each) = **$216/month**
- Or 2x Linode 8GB + HPA for bursts = **$144/month**

### Cost Saving Tips

1. **Use HPA**: Let Kubernetes scale down during low traffic
2. **Resource Limits**: Set appropriate limits to pack pods efficiently
3. **Shared Nodes**: Run multiple services on same nodes
4. **Spot/Preemptible**: Use if available (not on Linode LKE)
5. **Reserved Capacity**: Commit to 1-3 years for discounts

## Useful Commands Cheat Sheet

```bash
# Get all resources in namespace
kubectl get all -n educationelly

# Delete all resources
kubectl delete namespace educationelly

# Restart deployment
kubectl rollout restart deployment/educationelly-server -n educationelly

# Shell into pod
kubectl exec -it <pod-name> -n educationelly -- /bin/sh

# Copy files from pod
kubectl cp educationelly/<pod-name>:/path/to/file ./local-file

# View resource usage
kubectl top pods -n educationelly
kubectl top nodes

# Get pod YAML
kubectl get pod <pod-name> -n educationelly -o yaml

# Apply all manifests
kubectl apply -f k8s/

# Delete all manifests
kubectl delete -f k8s/
```

## Next Steps

- Set up continuous deployment from GitHub Actions
- Configure monitoring and alerting
- Implement backup strategy
- Set up staging environment
- Load testing and optimization

## Additional Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Linode Kubernetes Engine Docs](https://www.linode.com/docs/kubernetes/)
- [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [cert-manager Documentation](https://cert-manager.io/docs/)
