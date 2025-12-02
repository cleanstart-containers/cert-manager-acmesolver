# Cert-Manager ACME Solver - CleanStart Container

A security-hardened container image for cert-manager's ACME HTTP-01 challenge solver, enabling automated TLS certificate provisioning from ACME-compatible certificate authorities.

## Overview

The cert-manager ACME solver is a specialized HTTP server component that handles HTTP-01 challenges for cert-manager, enabling automated TLS certificate provisioning from ACME-compatible certificate authorities like Let's Encrypt. The solver responds to challenge requests at the `/.well-known/acme-challenge/` path, serving challenge tokens that prove domain ownership and enable automated certificate issuance without manual intervention.

**Key Features:**
* HTTP-01 challenge solving for ACME protocol certificate authorities
* Automatic domain ownership verification through HTTP challenge responses
* Integration with cert-manager for automated certificate lifecycle management
* Support for Let's Encrypt and other ACME-compatible certificate authorities
* Health check endpoint for monitoring and readiness verification
* Lightweight and efficient operation with minimal resource requirements
* Secure challenge token handling and validation
* Ephemeral pod design - created on-demand and cleaned up after challenge completion

**Common Use Cases:**
* Automated TLS certificate provisioning for Kubernetes services
* Let's Encrypt certificate automation in Kubernetes clusters
* Domain-validated SSL/TLS certificate management
* Automated certificate renewal for web applications
* Multi-domain certificate management with HTTP-01 validation
* Development and staging environment certificate automation
* CI/CD pipeline certificate provisioning
* Microservices architecture with automated certificate management

## What Cert-Manager ACME Solver Does

The ACME solver operates as a temporary pod created by cert-manager when an ACME HTTP-01 challenge is required:

1. **Handles Challenge Requests**: Listens on port 8089 and serves challenge tokens at the `/.well-known/acme-challenge/<token>` path
2. **Validates Domain Ownership**: Responds to HTTP GET requests from certificate authorities with challenge tokens, proving domain control
3. **Integrates with Cert-Manager**: Receives challenge configuration from cert-manager through environment variables or mounted secrets
4. **Provides Health Checks**: Responds to GET requests at `/` with an OK status for Kubernetes health probes
5. **Automatic Cleanup**: Pods are ephemeral - created when needed and deleted after challenge completion

## Image Details

**Image:** `cleanstart/cert-manager-acmesolver:latest-dev`

**Key Specifications:**
* **Binary Location:** `/usr/bin/acmesolver`
* **User:** `clnstrt` (non-root, UID 1000)
* **Architecture:** `amd64`
* **OS:** `linux`
* **Port:** `8089` (default ACME solver HTTP port)
* **SSL Certificates:** Pre-configured at `/etc/ssl/certs/ca-certificates.crt`

## How It Works

The CleanStart cert-manager ACME solver image provides a security-hardened implementation of the cert-manager acmesolver component:

1. **Security Hardening**: Runs as non-root user (`clnstrt`, UID 1000) with all Linux capabilities dropped, following the principle of least privilege
2. **Privilege Escalation Prevention**: Configured with `allowPrivilegeEscalation: false` to prevent privilege escalation attacks
3. **Pre-Configured SSL/TLS**: Includes complete SSL certificate bundle for secure connections to certificate authorities
4. **Kubernetes Integration**: Automatically receives pod metadata (POD_NAME, POD_NAMESPACE) through downward API for proper logging
5. **Resource Management**: Deployed with conservative resource requests and limits for predictable resource usage
6. **Health Monitoring**: Built-in health check support enables Kubernetes liveness and readiness probes

When cert-manager is configured to use this CleanStart image via the `ACME_HTTP01_SOLVER_IMAGE` environment variable, all ACME HTTP-01 challenge pods automatically benefit from these security enhancements without requiring changes to certificate management workflows.

## Kubernetes Deployment

The `kubernetes/` directory contains a complete, production-ready Kubernetes deployment:

## Configuration

### Ports

* **8089** - HTTP port (TCP) - Serves ACME challenge requests at `/.well-known/acme-challenge/<token>`

### Environment Variables

* `SSL_CERT_FILE=/etc/ssl/certs/ca-certificates.crt` - SSL certificate path for secure communication with certificate authorities
* `PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin` - Standard PATH environment variable
* `POD_NAME` - Automatically populated from Kubernetes pod metadata via downward API
* `POD_NAMESPACE` - Automatically populated from Kubernetes pod metadata via downward API

### RBAC Permissions

The solver requires namespace-scoped permissions to:
* Get, list, watch, create, update, and patch Pods, Services, Endpoints, ConfigMaps, Events, and Secrets
* Interact with resources necessary for ACME challenge completion

## Best Practices

* Configure cert-manager to use the CleanStart image via `ACME_HTTP01_SOLVER_IMAGE` environment variable
* Deploy solver pods with proper resource limits to ensure predictable resource usage
* Monitor solver pod logs for challenge success rates and troubleshooting
* Use the health check endpoint for Kubernetes liveness and readiness probes
* Keep the container image updated with the latest security patches
* Implement proper network policies if required by your security policies
* Review cert-manager Certificate resources to ensure proper ACME issuer configuration

## Security Notes

* The container runs as non-root user (`clnstrt`, UID 1000)
* All Linux capabilities are dropped for security
* Privilege escalation is prevented (`allowPrivilegeEscalation: false`)
* SSL certificates are pre-configured for secure communication with certificate authorities
* Ephemeral pod design minimizes attack surface - pods are deleted after challenge completion
* Security context prevents privilege escalation and enforces non-root execution
* Challenge tokens are handled securely and not stored persistently

## Observability

The solver provides structured logging that includes:
* Challenge token validation events
* HTTP request handling (paths, headers, response codes)
* Error conditions and failure reasons
* Listener startup and port binding status

Health check functionality allows Kubernetes to monitor solver readiness through TCP socket checks on port 8089. The solver responds to GET requests at the root path (`/`) with an OK response, enabling proper integration with Kubernetes liveness and readiness probes.

While the solver itself doesn't expose metrics endpoints, its integration with cert-manager provides observability through cert-manager's metrics and logging. Cert-manager tracks solver pod creation, challenge status, and certificate issuance success rates.

For deployment instructions, configuration details, and troubleshooting guides, see the `kubernetes/README.md` file in the `kubernetes/` directory.
