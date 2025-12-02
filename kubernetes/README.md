# 🔐 Cert-Manager ACME Solver - Kubernetes Deployment Guide

Complete Kubernetes deployment guide for the CleanStart Cert-Manager ACME Solver container. The ACME solver is used by cert-manager to solve HTTP-01 challenges for Let's Encrypt and other ACME-compatible certificate authorities.

## 📁 Files

- `deployment.yaml` - Complete deployment manifest (ServiceAccount, RBAC, Deployment, Service)
- `README.md` - This documentation

## 🖼️ Image Details

**Image:** `cleanstart/cert-manager-acmesolver:latest-dev`

**Key Features:**
- **Binary Location:** `/usr/bin/acmesolver`
- **Command:** The deployment uses `command: ["/usr/bin/acmesolver"]` to run the solver
- **User:** `clnstrt` (non-root, UID 1000)
- **Architecture:** `amd64`
- **OS:** `linux`
- **Port:** `8089` (default ACME solver HTTP port)
- **SSL Certificates:** Pre-configured at `/etc/ssl/certs/ca-certificates.crt`

## 🚀 Complete Deployment Steps

### Prerequisites

1. **Kubernetes cluster** (Kind, minikube, k3s, GKE, EKS, AKS, or any other)
2. **kubectl** installed and configured to access your cluster
3. **cert-manager** installed (optional, for full integration testing)

### Step 1: Verify Kubernetes Cluster

```bash
# Check cluster connectivity
kubectl cluster-info

# Verify nodes are ready
kubectl get nodes
```

### Step 2: Deploy the ACME Solver

```bash
# Apply the deployment manifest
kubectl apply -f deployment.yaml

# Verify the pod is running
kubectl get pods -l app=cert-manager-acmesolver -w
```

**Expected Output:**
```
NAME                                      READY   STATUS    RESTARTS   AGE
cert-manager-acmesolver-xxxxxxxxxx-xxxxx  1/1     Running   0          30s
```

### Step 3: Verify Deployment

```bash
# Check pod status
kubectl get pods -l app=cert-manager-acmesolver

# View pod logs
kubectl logs -l app=cert-manager-acmesolver

# Check service
kubectl get svc cert-manager-acmesolver

# Describe the deployment
kubectl describe deployment cert-manager-acmesolver
```

### Step 4: Test the Deployment

The acmesolver is working correctly if the pod is running and listening on port 8089. The acmesolver provides a health check endpoint and responds to ACME HTTP-01 challenge requests.

```bash
# Verify the pod is running and listening
kubectl logs -l app=cert-manager-acmesolver

# You should see output like:
# "starting listener" logger="cert-manager.acmesolver" listen_port=8089

# Port forward to access the acmesolver
kubectl port-forward deployment/cert-manager-acmesolver 8089:8089

# In another terminal, test the health check endpoint
curl http://localhost:8089

# The acmesolver expects ACME challenges at:
# http://localhost:8089/.well-known/acme-challenge/<token>
# This path is used by certificate authorities like Let's Encrypt
```

## 📋 Deployment Components

The `deployment.yaml` includes:

1. **ServiceAccount** - `cert-manager-acmesolver`
   - Provides identity for the pod

2. **Role & RoleBinding** - RBAC permissions
   - Allows the solver to interact with pods, services, configmaps, and secrets
   - Required for cert-manager integration

3. **Deployment** - Main application deployment
   - Runs the acmesolver binary
   - Configured with security best practices (non-root, dropped capabilities)
   - Resource limits and health checks included

4. **Service** - ClusterIP service
   - Exposes port 8089 for internal cluster access
   - Used by cert-manager to route challenge requests

## 🔧 Configuration

### Environment Variables

The deployment sets the following environment variables:
- `SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt` - SSL certificate bundle location
- `PATH` - Standard system PATH
- `POD_NAME` - Automatically set from pod metadata
- `POD_NAMESPACE` - Automatically set from pod metadata

### Ports

- **8089** - HTTP port for ACME challenge requests

### Resource Limits

- **Requests:** CPU: 100m, Memory: 128Mi
- **Limits:** CPU: 500m, Memory: 512Mi

### Security Context

- Runs as non-root user (UID 1000, user: `clnstrt`)
- All capabilities dropped
- No privilege escalation allowed
- Read-only root filesystem: false (may be required for some operations)

## 🧪 Configuring Cert-Manager to Use This Custom Image

To use this custom acmesolver image with cert-manager, you need to configure the cert-manager controller to use your custom image. This is done by adding the `ACME_HTTP01_SOLVER_IMAGE` environment variable to the cert-manager controller deployment.

**Important:** The environment variable must be added to the **cert-manager controller deployment**, not to this acmesolver deployment. When cert-manager needs to solve ACME HTTP-01 challenges, it will create temporary pods using the image specified in this environment variable.

### Step 1: Locate the Cert-Manager Controller Deployment

```bash
# Verify cert-manager is installed
kubectl get deployment -n cert-manager

# You should see a deployment named 'cert-manager'
```

### Step 2: Add the Environment Variable

You have two options:

#### Option A: Edit the Deployment (Interactive)

```bash
# Open the cert-manager controller deployment in your default editor
kubectl edit deployment cert-manager -n cert-manager
```

In the editor, find the `containers` section for the `cert-manager` container and add the environment variable to the `env` array:

```yaml
spec:
  template:
    spec:
      containers:
      - name: cert-manager
        # ... existing configuration ...
        env:
        # ... existing environment variables ...
        - name: ACME_HTTP01_SOLVER_IMAGE
          value: "cleanstart/cert-manager-acmesolver:latest-dev"
```

Save and close the editor. Kubernetes will automatically restart the cert-manager controller pod with the new configuration.

#### Option B: Patch the Deployment (Non-Interactive)

```bash
# Patch the deployment directly without opening an editor
kubectl set env deployment/cert-manager \
  -n cert-manager \
  ACME_HTTP01_SOLVER_IMAGE=cleanstart/cert-manager-acmesolver:latest-dev
```

This command will:
1. Add the environment variable to the cert-manager controller deployment
2. Trigger a rolling update to restart the controller pod
3. Apply the change immediately

### Step 3: Verify the Configuration

After adding the environment variable, verify it was applied correctly:

```bash
# Check the cert-manager controller deployment
kubectl get deployment cert-manager -n cert-manager -o yaml | grep -A 5 ACME_HTTP01_SOLVER_IMAGE

# Or describe the deployment
kubectl describe deployment cert-manager -n cert-manager | grep ACME_HTTP01_SOLVER_IMAGE

# Check the running pod's environment
kubectl get pods -n cert-manager -l app.kubernetes.io/name=cert-manager
kubectl exec -n cert-manager <cert-manager-pod-name> -- env | grep ACME_HTTP01_SOLVER_IMAGE
```

### Step 4: Test with an ACME Certificate Request

When cert-manager creates an ACME challenge pod, verify it uses your custom image:

```bash
# Create a test Certificate resource that triggers an ACME challenge
# (This requires a properly configured ACME Issuer/ClusterIssuer)

# Watch for ACME solver pods
kubectl get pods --all-namespaces -l acme.cert-manager.io/http01-solver=true -w

# When a solver pod is created, check its image
kubectl describe pod <solver-pod-name> -n <namespace> | grep Image
```

The solver pod should show:
```
Image: cleanstart/cert-manager-acmesolver:latest-dev
```

### How It Works

1. **Cert-Manager Controller** reads the `ACME_HTTP01_SOLVER_IMAGE` environment variable
2. When an ACME HTTP-01 challenge is needed, cert-manager creates a temporary pod
3. The pod uses the image specified in `ACME_HTTP01_SOLVER_IMAGE`
4. The solver pod handles the ACME challenge request
5. After the challenge completes, cert-manager deletes the temporary pod

### If cert-manager is not using your custom image:

```bash
# Verify the environment variable is set
kubectl get deployment cert-manager -n cert-manager -o jsonpath='{.spec.template.spec.containers[0].env[?(@.name=="ACME_HTTP01_SOLVER_IMAGE")]}'

# Check cert-manager controller logs for errors
kubectl logs -n cert-manager deployment/cert-manager | grep -i acme

# Ensure the image is accessible from your cluster
kubectl run test --image=cleanstart/cert-manager-acmesolver:latest-dev --rm -it --restart=Never -- /bin/sh
```

### Health Check Failures

The acmesolver uses TCP socket checks for health probes. If the pod is not becoming ready, check the logs to see if the acmesolver is listening on port 8089.

### Empty Web Page in Browser

**This is expected!** The acmesolver responds to GET requests at "/" with an OK health check response (you'll see `"responding OK to health check"` in the logs). However, browsers may display this as an empty page since it's not HTML content. The acmesolver is working correctly.

The acmesolver is designed to:
- Respond to health checks at `GET /` (returns OK)
- Handle ACME HTTP-01 challenges at `/.well-known/acme-challenge/<token>`

To verify it's working:
- Check the pod logs: `kubectl logs -l app=cert-manager-acmesolver`
- You should see: `"responding OK to health check"` for GET requests to "/"
- You should see: `"starting listener" listen_port=8089` when it starts
- Test with curl: `curl http://localhost:8089` (should return OK or no response)

## 🗑️ Cleanup

To remove the deployment:

```bash
kubectl delete -f deployment.yaml
```

Or delete individual resources:

```bash
kubectl delete deployment cert-manager-acmesolver
kubectl delete service cert-manager-acmesolver
kubectl delete serviceaccount cert-manager-acmesolver
kubectl delete role cert-manager-acmesolver
kubectl delete rolebinding cert-manager-acmesolver
```

