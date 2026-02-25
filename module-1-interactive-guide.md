# Module 1: OpenChoreo Single-Cluster Setup - Interactive Guide

**Learn by Doing with Your AI Coding Agent**

This guide is designed for hands-on learning with an AI coding agent (Claude, GitHub Copilot, etc.). You'll execute commands step-by-step, understand what each does, and verify your progress.

## Prerequisites

Before starting Module 1, ensure Module 0 is complete:
- Rancher Desktop configured with Docker runtime
- All required tools installed (Go, k3d, kubectl, helm, etc.)
- `./check-tools.sh` passes
- Git remotes configured (origin = your fork, upstream = openchoreo/openchoreo)

## Learning Objectives

By the end of this module, you will:
1. Understand OpenChoreo's multi-plane architecture (Control, Data, Build, Observability)
2. Create a local k3d Kubernetes cluster
3. Install and configure the Control Plane
4. Install and configure the Data Plane
5. Verify the system is running correctly
6. Know how to access OpenChoreo services

## Time Estimate: 45-60 minutes

---

## Phase 1: Understanding the Architecture (5 minutes)

### What You'll Learn
OpenChoreo uses a **multi-plane architecture**:
- **Control Plane**: Manages the overall system, hosts the API and UI
- **Data Plane**: Runs your application workloads
- **Build Plane** (optional): Handles building applications from source
- **Observability Plane** (optional): Collects metrics, logs, and traces

In this module, we'll set up Control and Data planes. Build Plane is covered in Module 3.

### Key Concepts
- **k3d**: Creates lightweight Kubernetes clusters using k3s in Docker
- **Helm**: Kubernetes package manager (like npm for Kubernetes)
- **CRDs**: Custom Resource Definitions - extend Kubernetes with new resource types

---

## Phase 2: Create k3d Cluster (5 minutes)

### Working Directory
**IMPORTANT**: All commands must be run from the OpenChoreo source directory:

```bash
cd /Users/lahirus/work/pet-projects/open-choreo/openchoreo-learning/code/openchoreo
```

### Command 1: Verify you're in the right directory

**Ask your AI agent:**
> "Show me the current directory and list the top-level files to confirm we're in the openchoreo source repo"

**Expected output**: You should see directories like `api/`, `install/`, `samples/`, and files like `Makefile`, `go.mod`

---

### Command 2: Create the k3d cluster

**What this does**: Creates a local Kubernetes cluster named "openchoreo" with specific port mappings and configuration.

**Ask your AI agent:**
> "Create the k3d cluster using the config file at install/k3d/single-cluster/config.yaml"

**Command:**
```bash
k3d cluster create --config install/k3d/single-cluster/config.yaml
```

**What to expect:**
- Takes 1-2 minutes
- Creates a cluster with pre-configured port mappings
- Sets up networking for local development

**Learning moment**: Port mappings allow you to access services running inside the cluster from your host machine (e.g., :8080, :19080)

---

### Command 3: Set machine ID (required for some workloads)

**Ask your AI agent:**
> "Execute the machine ID setup command for the k3d server"

**Command:**
```bash
docker exec k3d-openchoreo-server-0 sh -c \
  "cat /proc/sys/kernel/random/uuid | tr -d '-' > /etc/machine-id"
```

**What this does**: Some applications need a unique machine identifier. This creates one inside the k3d container.

---

### Command 4: Verify cluster is running

**Ask your AI agent:**
> "Verify the k3d cluster is running and kubectl can connect to it"

**Commands:**
```bash
k3d cluster list
kubectl cluster-info
kubectl get nodes
```

**Expected output:**
- Cluster "openchoreo" is running
- kubectl shows connection to the cluster
- One node in "Ready" state

---

## Phase 3: Install Prerequisites (15 minutes)

OpenChoreo requires several foundational components. Each serves a specific purpose.

### Command 5: Install Gateway API CRDs

**What this is**: Gateway API is a Kubernetes standard for configuring ingress/egress traffic.

**Ask your AI agent:**
> "Install the Kubernetes Gateway API CRDs version 1.4.1"

**Command:**
```bash
kubectl apply --server-side \
  -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.1/experimental-install.yaml
```

**Verify:**
```bash
kubectl get crd | grep gateway
```

**Expected output**: Several CRDs with names like `gateways.gateway.networking.k8s.io`

---

### Command 6: Install cert-manager

**What this is**: Manages TLS certificates automatically (important for HTTPS).

**Ask your AI agent:**
> "Install cert-manager v1.19.2 using Helm"

**Commands:**
```bash
helm upgrade --install cert-manager oci://quay.io/jetstack/charts/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.19.2 \
  --set crds.enabled=true
```

**Wait for it to be ready:**
```bash
kubectl wait --for=condition=Available deployment/cert-manager \
  -n cert-manager --timeout=180s
```

**Verify:**
```bash
kubectl get pods -n cert-manager
```

**Expected output**: 3 pods in "Running" state (cert-manager, webhook, cainjector)

**Learning moment**: `--create-namespace` automatically creates the namespace if it doesn't exist

---

### Command 7: Install External Secrets Operator

**What this is**: Manages secrets by syncing them from external secret stores.

**Ask your AI agent:**
> "Install External Secrets Operator v1.3.2"

**Commands:**
```bash
helm upgrade --install external-secrets oci://ghcr.io/external-secrets/charts/external-secrets \
  --namespace external-secrets \
  --create-namespace \
  --version 1.3.2 \
  --set installCRDs=true

kubectl wait --for=condition=Available deployment/external-secrets \
  -n external-secrets --timeout=180s
```

**Verify:**
```bash
kubectl get pods -n external-secrets
```

---

### Command 8: Install kgateway

**What this is**: OpenChoreo's gateway implementation for routing HTTP/HTTPS traffic.

**Ask your AI agent:**
> "Install kgateway v2.2.1 with CRDs and experimental features enabled"

**Commands:**
```bash
helm upgrade --install kgateway-crds oci://cr.kgateway.dev/kgateway-dev/charts/kgateway-crds \
  --create-namespace --namespace openchoreo-control-plane \
  --version v2.2.1

helm upgrade --install kgateway oci://cr.kgateway.dev/kgateway-dev/charts/kgateway \
  --namespace openchoreo-control-plane --create-namespace \
  --version v2.2.1 \
  --set controller.extraEnv.KGW_ENABLE_GATEWAY_API_EXPERIMENTAL_FEATURES=true
```

**Verify:**
```bash
kubectl get pods -n openchoreo-control-plane
```

**Expected output**: Gateway controller pod running

---

## Phase 4: Setup Control Plane (15 minutes)

### Command 9: Install Thunder (Identity Provider)

**What this is**: Thunder is OpenChoreo's built-in identity and access management system.

**Ask your AI agent:**
> "Install Thunder v0.23.0 for authentication and authorization"

**Command:**
```bash
helm upgrade --install thunder oci://ghcr.io/asgardeo/helm-charts/thunder \
  --namespace openchoreo-control-plane \
  --create-namespace \
  --version 0.23.0 \
  --values install/k3d/common/values-thunder.yaml
```

**Learning moment**: The values file (`values-thunder.yaml`) contains pre-configured test users and OAuth apps for local development.

---

### Command 10: Setup CoreDNS for local routing

**What this does**: Configures DNS so `*.localhost` domains resolve correctly inside the cluster.

**Ask your AI agent:**
> "Apply the CoreDNS custom configuration for localhost routing"

**Command:**
```bash
kubectl apply -f install/k3d/common/coredns-custom.yaml
```

---

### Command 11: Create Backstage secrets

**What this is**: Backstage is the UI framework OpenChoreo uses for its console.

**Ask your AI agent:**
> "Create the backstage-secrets with required credentials"

**Commands:**
```bash
kubectl create namespace openchoreo-control-plane --dry-run=client -o yaml | kubectl apply -f -

kubectl create secret generic backstage-secrets \
  -n openchoreo-control-plane \
  --from-literal=backend-secret="$(head -c 32 /dev/urandom | base64)" \
  --from-literal=client-secret="backstage-portal-secret" \
  --from-literal=jenkins-api-key="placeholder-not-in-use"
```

**Learning moment**: `--dry-run=client -o yaml | kubectl apply` is an idempotent way to ensure a namespace exists without failing if it already exists.

---

### Command 12: Install OpenChoreo Control Plane

**This is the big one!** Installs the core OpenChoreo control plane components.

**Ask your AI agent:**
> "Install the OpenChoreo control plane using the single-cluster values"

**Commands:**
```bash
helm upgrade --install openchoreo-control-plane install/helm/openchoreo-control-plane \
  --namespace openchoreo-control-plane \
  --create-namespace \
  --values install/k3d/single-cluster/values-cp.yaml

kubectl wait -n openchoreo-control-plane \
  --for=condition=available --timeout=300s deployment --all
```

**What's happening**: This deploys the API server, controllers, and Backstage console.

**Verify:**
```bash
kubectl get pods -n openchoreo-control-plane
```

**Expected output**: Multiple pods (backstage, controller-manager, api-gateway, etc.) all in "Running" state

**Troubleshooting tip**: If pods are stuck in "Pending" or "ImagePullBackOff", ask your AI agent to describe the pod for details:
```bash
kubectl describe pod <pod-name> -n openchoreo-control-plane
```

---

## Phase 5: Install Default Resources (5 minutes)

### Command 13: Apply default platform resources

**What this does**: Creates default resources like environments, deployment tracks, and component types.

**Ask your AI agent:**
> "Apply the default getting-started resources and label the default namespace"

**Commands:**
```bash
kubectl apply -f samples/getting-started/all.yaml
kubectl label namespace default openchoreo.dev/controlplane-namespace=true
```

**Verify:**
```bash
kubectl get environments -n default
kubectl get componenttypes -n default
```

**Expected output**: Several environments (development, staging, production) and component types

---

## Phase 6: Setup Data Plane (15 minutes)

The Data Plane is where your actual workloads run.

### Command 14: Create namespace and copy certificates

**What this does**: The data plane needs certificates to communicate securely with the control plane.

**Ask your AI agent:**
> "Create the data plane namespace and copy the cluster gateway certificates from control plane"

**Commands:**
```bash
kubectl create namespace openchoreo-data-plane --dry-run=client -o yaml | kubectl apply -f -

CA_CRT=$(kubectl get configmap cluster-gateway-ca \
  -n openchoreo-control-plane -o jsonpath='{.data.ca\.crt}')

kubectl create configmap cluster-gateway-ca \
  --from-literal=ca.crt="$CA_CRT" \
  -n openchoreo-data-plane

TLS_CRT=$(kubectl get secret cluster-gateway-ca \
  -n openchoreo-control-plane -o jsonpath='{.data.tls\.crt}' | base64 -d)
TLS_KEY=$(kubectl get secret cluster-gateway-ca \
  -n openchoreo-control-plane -o jsonpath='{.data.tls\.key}' | base64 -d)

kubectl create secret generic cluster-gateway-ca \
  --from-literal=tls.crt="$TLS_CRT" \
  --from-literal=tls.key="$TLS_KEY" \
  --from-literal=ca.crt="$CA_CRT" \
  -n openchoreo-data-plane
```

**Learning moment**: This pattern is common in Kubernetes - securely sharing certificates between namespaces.

---

### Command 15: Create ClusterSecretStore

**What this is**: Defines where the data plane gets secrets from (for this local setup, we use fake values).

**Ask your AI agent:**
> "Create a fake ClusterSecretStore for local development"

**Command:**
```bash
kubectl apply -f - <<EOF
apiVersion: external-secrets.io/v1
kind: ClusterSecretStore
metadata:
  name: default
spec:
  provider:
    fake:
      data:
      - key: npm-token
        value: "fake-npm-token-for-development"
      - key: docker-username
        value: "dev-user"
      - key: docker-password
        value: "dev-password"
      - key: github-pat
        value: "fake-github-token-for-development"
      - key: username
        value: "dev-user"
      - key: password
        value: "dev-password"
EOF
```

**Learning moment**: In production, this would connect to HashiCorp Vault, AWS Secrets Manager, etc.

---

### Command 16: Install Data Plane

**Ask your AI agent:**
> "Install the OpenChoreo data plane with dependencies"

**Command:**
```bash
helm upgrade --install openchoreo-data-plane install/helm/openchoreo-data-plane \
  --dependency-update \
  --namespace openchoreo-data-plane \
  --create-namespace \
  --values install/k3d/single-cluster/values-dp.yaml
```

**Verify:**
```bash
kubectl get pods -n openchoreo-data-plane
```

**Expected output**: cluster-agent and other data plane components running

---

### Command 17: Register the Data Plane

**What this does**: Creates a DataPlane resource that tells the control plane about this data plane.

**Ask your AI agent:**
> "Register the data plane with the control plane by creating a DataPlane resource"

**Command:**
```bash
AGENT_CA=$(kubectl get secret cluster-agent-tls \
  -n openchoreo-data-plane -o jsonpath='{.data.ca\.crt}' | base64 -d)

kubectl apply -f - <<EOF
apiVersion: openchoreo.dev/v1alpha1
kind: DataPlane
metadata:
  name: default
  namespace: default
spec:
  planeID: default
  clusterAgent:
    clientCA:
      value: |
$(echo "$AGENT_CA" | sed 's/^/        /')
  secretStoreRef:
    name: default
  gateway:
    publicVirtualHost: openchoreoapis.localhost
    publicHTTPPort: 19080
    publicHTTPSPort: 19443
EOF
```

**Verify:**
```bash
kubectl get dataplane -n default
```

**Expected output**: One DataPlane named "default"

---

## Phase 7: Verification and Access (5 minutes)

### Command 18: Verify all components

**Ask your AI agent:**
> "Run a comprehensive verification of all installed components"

**Commands:**
```bash
echo "=== Control Plane Pods ==="
kubectl get pods -n openchoreo-control-plane

echo ""
echo "=== Data Plane Pods ==="
kubectl get pods -n openchoreo-data-plane

echo ""
echo "=== Plane Resources ==="
kubectl get dataplane -n default

echo ""
echo "=== Check Data Plane Agent Logs ==="
kubectl logs -n openchoreo-data-plane -l app=cluster-agent --tail=10
```

**Expected output**: All pods in "Running" state, DataPlane resource exists, agent logs show successful connection

---

### Command 19: Access the OpenChoreo Console

**Ask your AI agent:**
> "Check if the OpenChoreo console is accessible"

**URL**: http://openchoreo.localhost:8080

**Try in your browser:**
1. Open http://openchoreo.localhost:8080
2. You should see the OpenChoreo login page
3. Default test credentials are in `install/k3d/common/values-thunder.yaml`

**Learning moment**: The console runs on port 8080 (mapped from the cluster) and uses `*.localhost` domains for routing.

---

### Command 20: Check the API

**Ask your AI agent:**
> "Test the OpenChoreo API endpoint"

**Command:**
```bash
curl -s http://api.openchoreo.localhost:8080/health | jq .
```

**Expected output**: Health check response (JSON)

**Note**: If `jq` is not installed, you can omit it: `curl -s http://api.openchoreo.localhost:8080/health`

---

## Phase 8: Understanding What You Built

### Key Resources Created

**Ask your AI agent:**
> "Show me a summary of all the key Kubernetes resources we created"

**Commands:**
```bash
kubectl get namespaces | grep openchoreo
kubectl get crd | grep openchoreo
kubectl get all -n openchoreo-control-plane
kubectl get dataplane,environments,componenttypes -n default
```

### Architecture Review

You now have:
1. **Control Plane** (port 8080/8443)
   - API server for managing resources
   - Backstage console for UI
   - Thunder for authentication
   - Controllers watching for resource changes

2. **Data Plane** (port 19080/19443)
   - Cluster agent connecting to control plane
   - Gateway for routing traffic to workloads
   - Namespace where your apps will run

3. **Default Resources**
   - Environments (dev, staging, prod)
   - Component types (service, web-app, etc.)
   - Deployment tracks (defining deployment pipeline)

---

## Important Commands Reference

**Ask your AI agent to save these for you:**

### Cluster Management
```bash
# List clusters
k3d cluster list

# Stop cluster (preserves data)
k3d cluster stop openchoreo

# Start cluster
k3d cluster start openchoreo

# Delete cluster (destroys everything)
k3d cluster delete openchoreo
```

### Debugging
```bash
# View control plane logs
kubectl logs -n openchoreo-control-plane -l app=controller-manager --tail=50

# View data plane agent logs
kubectl logs -n openchoreo-data-plane -l app=cluster-agent --tail=50

# Describe a pod (troubleshooting)
kubectl describe pod <pod-name> -n <namespace>

# Get events (recent cluster activity)
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

### Port Forwarding (if direct access doesn't work)
```bash
# Forward console to local port
kubectl port-forward -n openchoreo-control-plane svc/backstage 7007:7007

# Forward API
kubectl port-forward -n openchoreo-control-plane svc/api-gateway 8080:8080
```

---

## Module 1 Exit Criteria

Before moving to Module 2, verify:

- [ ] Control plane pods are all healthy and running
- [ ] Data plane pods are all healthy and running
- [ ] DataPlane resource exists in the default namespace
- [ ] OpenChoreo console is reachable at http://openchoreo.localhost:8080
- [ ] API health endpoint returns success
- [ ] You can explain the purpose of each plane (Control, Data)
- [ ] You understand what CRDs, Helm charts, and k3d are

### Update Your Progress Log

**Ask your AI agent:**
> "Help me update the progress-log.md file with my Module 1 completion"

Add to [progress-log.md](progress-log.md):
- Date completed
- Commands that worked
- Any blockers encountered
- Key learnings

---

## Common Issues and Solutions

### Issue: Pods stuck in "Pending"
**Cause**: Not enough resources or image pull issues
**Solution**:
```bash
kubectl describe pod <pod-name> -n <namespace>
# Look for "Events" section for details
```

### Issue: Cannot access console at openchoreo.localhost:8080
**Cause**: Port mapping or DNS issue
**Solution**:
```bash
# Verify ports are mapped
docker ps | grep openchoreo

# Try port-forward instead
kubectl port-forward -n openchoreo-control-plane svc/backstage 7007:7007
# Then access http://localhost:7007
```

### Issue: "connection refused" errors between planes
**Cause**: Certificates not copied correctly
**Solution**: Re-run certificate copy commands (Command 14)

### Issue: Helm chart not found
**Cause**: Network issue or chart repository problem
**Solution**:
```bash
# Verify you have internet connectivity
# Try pulling a specific chart
helm pull oci://quay.io/jetstack/charts/cert-manager --version v1.19.2
```

---

## Next Steps: Module 2

Once Module 1 is complete, you're ready for **Module 2: Deploy First Workload and Trace Runtime Resources**.

In Module 2, you will:
- Deploy a sample application
- Trace how OpenChoreo creates Kubernetes resources
- Understand the Component → ReleaseBinding → Deployment chain
- Access your deployed application

---

## Learning Checkpoint

**Discuss with your AI agent:**

1. "Explain to me why we need both a control plane and a data plane"
2. "What would happen if the cluster-agent in the data plane couldn't connect to the control plane?"
3. "Show me how to find the IP address and ports for the data plane gateway"
4. "What CRDs did OpenChoreo install? List them"

Understanding these concepts deeply will help you in later modules when you start modifying controller behavior and contributing code.

---

## Additional Resources

- [OpenChoreo Architecture Docs](https://openchoreo.dev/docs/architecture/)
- [k3d Documentation](https://k3d.io/)
- [Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/)
- `code/openchoreo/install/k3d/single-cluster/README.md` - Source reference

---

**Congratulations on completing Module 1!** You now have a fully functional OpenChoreo development environment. Take a break, then continue to Module 2 to deploy your first application.
