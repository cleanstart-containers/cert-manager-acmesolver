# CleanStart Container for Cert-Manager ACME Solver

A security-hardened container image for cert-manager's ACME HTTP-01 challenge solver.  
Enables automated TLS certificate provisioning from ACME-compatible certificate authorities such as Let’s Encrypt.  
Optimized for Kubernetes, with non-root execution, restricted permissions, and minimal attack surface.

- --

## Key Features

- Automated HTTP-01 ACME challenge solving  
- Domain ownership verification through HTTP challenge responses  
- Native integration with cert-manager  
- Lightweight and resource-efficient  
- Built-in readiness and liveness health checks  
- Secure token handling with ephemeral pod lifecycle  
- Supports Let’s Encrypt and all ACME-compatible certificate authorities  

- --

## Common Use Cases

- Automated TLS certificate provisioning  
- Let's Encrypt certificate automation  
- Domain-validated SSL/TLS certificate management  
- Multi-domain HTTP-01 validation  
- Certificate renewal automation  
- CI/CD pipeline certificate provisioning  
- Microservices certificate automation in Kubernetes  

- --

## Image Details

- *Image:** `ghcr.io/cleanstart-containers/cert-manager-acmesolver:latest`  
- *Port:** `8089` (ACME solver HTTP port)  
- *User:** Non-root (`clnstrt`, UID 1000)  
- *Binary:** `/usr/bin/acmesolver`  
- *SSL Certificates:** Included at `/etc/ssl/certs/ca-certificates.crt`  

- --

## Getting Started

### Pull Commands

Download the runtime container images:
```bash
docker pull ghcr.io/cleanstart-containers/cert-manager-acmesolver:latest
```
```bash
docker pull ghcr.io/cleanstart-containers/cert-manager-acmesolver:latest-dev
```

### Basic Test Run

Run the container with a basic test command:
```bash
kubectl run acmesolver-test \
  --image=ghcr.io/cleanstart-containers/cert-manager-acmesolver:latest-dev \
  --restart=Never \
  -- /usr/bin/acmesolver --help
```

### Production Deployment

Recommended production deployment with hardened security:
```bash
docker run -d --name cert-manager-acmesolver-prod \
  --read-only \
  --security-opt=no-new-privileges \
  --user 1000:1000 \
  ghcr.io/cleanstart-containers/cert-manager-acmesolver:latest
```

## How It Works

- Serves ACME HTTP-01 challenge responses at:  
- Validates domain ownership by replying to CA probes  
- Receives configuration from cert-manager  
- Provides health checks at `/` for readiness/liveness  
- Pod is created on-demand and removed after challenge completion  

- --

## Kubernetes Deployment

### Important Environment Variables

- `SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt`  
- `POD_NAME` (auto-populated by the downward API)  
- `POD_NAMESPACE` (auto-populated by the downward API)  

### Required RBAC Permissions  

Namespace-scoped access to:
- Pods  
- Services  
- Endpoints  
- ConfigMaps  
- Secrets  
- Events  

- --

## Best Practices

- Configure cert-manager using:  
`ACME_HTTP01_SOLVER_IMAGE=ghcr.io/cleanstart-containers/cert-manager-acmesolver`  
- Set CPU and memory limits for predictable behavior  
- Use Kubernetes readiness & liveness probes  
- Apply network policies if needed  
- Keep the container updated regularly  
- Validate ACME Issuer and Certificate configuration in cert-manager  

- --

## Resources

- **Official Documentation:** https://cert-manager.io/docs/
- **Provenance / SBOM / Signature:** https://images.cleanstart.com/images/cert-manager-acmesolver
- **Docker Hub:** https://hub.docker.com/r/cleanstart/cert-manager-acmesolver
- **CleanStart All Images:** https://images.cleanstart.com
- **CleanStart Community Images:** https://hub.docker.com/u/cleanstart

## Vulnerability Disclaimer

CleanStart provides Docker images that include third-party open-source components maintained by independent contributors. While CleanStart applies industry-standard security practices, it cannot guarantee the security or integrity of upstream components.

Users acknowledge that open-source software may contain undiscovered vulnerabilities or introduce risks through updates. CleanStart is not liable for security issues originating from third-party libraries, including zero-day exploits or supply-chain attacks.

Security is a shared responsibility — CleanStart provides secure updated images, while users must apply proper deployment and security controls.
