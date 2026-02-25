# Module 3: Build Plane and Source-to-Deploy Flow - Interactive Guide

**Learn by Doing with Your AI Coding Agent**

This guide teaches you how to set up OpenChoreo's Build Plane and deploy applications from source code, not just pre-built images. You'll watch the complete CI/CD pipeline in action.

## Prerequisites

Before starting Module 3:
- Module 1 complete (Control plane + Data plane running)
- Module 2 complete (Understand Component → Deployment chain)
- Comfortable with kubectl and basic Kubernetes concepts

## Learning Objectives

By the end of this module, you will:
1. Install and configure the Build Plane
2. Understand OpenChoreo's workflow-based build system
3. Deploy an application from source code
4. Trace the build workflow execution
5. Understand ClusterWorkflowTemplates and WorkflowRuns
6. Debug build failures

## Time Estimate: 45-60 minutes

---

## Phase 1: Understanding the Build Plane Architecture (5 minutes)

### What is the Build Plane?

**Ask your AI agent:**
> "Explain what the Build Plane does and how it differs from the Data Plane"

**Key Concepts:**

**Build Plane responsibilities:**
- Clone source code from Git repositories
- Build container images (using Docker, Buildpacks, etc.)
- Push images to container registries
- Generate deployment manifests (Workloads)
- Run on isolated build workers

**Build Plane uses Argo Workflows:**
- Kubernetes-native workflow engine
- Each build is a WorkflowRun (custom resource)
- Workflows are composed of steps (templates)
- Steps run in pods

**The Build Flow:**
```
Component (with source) created
  ↓
Controller triggers WorkflowRun
  ↓
Argo Workflows executes:
  1. Checkout source from Git
  2. Build image (Docker/Buildpacks)
  3. Push to registry
  4. Generate Workload manifest
  ↓
Workload created → triggers deployment
```

**Learning moment**: Build Plane is optional. If you only deploy pre-built images (like Module 2), you don't need it.

---

## Phase 2: Install Build Plane Components (20 minutes)

### Working Directory

```bash
cd /Users/lahirus/work/pet-projects/open-choreo/openchoreo-learning/code/openchoreo
```

---

### Command 1: Create Build Plane namespace and certificates

**What this does**: Sets up the namespace and copies certificates for secure communication.

**Ask your AI agent:**
> "Create the build plane namespace and copy the required certificates from control plane"

**Commands:**
```bash
kubectl create namespace openchoreo-build-plane --dry-run=client -o yaml | kubectl apply -f -

CA_CRT=$(kubectl get configmap cluster-gateway-ca \
  -n openchoreo-control-plane -o jsonpath='{.data.ca\.crt}')

kubectl create configmap cluster-gateway-ca \
  --from-literal=ca.crt="$CA_CRT" \
  -n openchoreo-build-plane

TLS_CRT=$(kubectl get secret cluster-gateway-ca \
  -n openchoreo-control-plane -o jsonpath='{.data.tls\.crt}' | base64 -d)
TLS_KEY=$(kubectl get secret cluster-gateway-ca \
  -n openchoreo-control-plane -o jsonpath='{.data.tls\.key}' | base64 -d)

kubectl create secret generic cluster-gateway-ca \
  --from-literal=tls.crt="$TLS_CRT" \
  --from-literal=tls.key="$TLS_KEY" \
  --from-literal=ca.crt="$CA_CRT" \
  -n openchoreo-build-plane
```

**Verify:**
```bash
kubectl get namespace openchoreo-build-plane
kubectl get configmap cluster-gateway-ca -n openchoreo-build-plane
kubectl get secret cluster-gateway-ca -n openchoreo-build-plane
```

**Learning moment**: Same certificate pattern as Data Plane. All planes need secure communication with Control Plane.

---

### Command 2: Install local container registry

**What this is**: A local Docker registry where built images will be stored.

**Ask your AI agent:**
> "Install a local Docker registry in the build plane namespace"

**Commands:**
```bash
helm repo add twuni https://twuni.github.io/docker-registry.helm
helm repo update

helm install registry twuni/docker-registry \
  --namespace openchoreo-build-plane \
  --create-namespace \
  --values install/k3d/single-cluster/values-registry.yaml
```

**Wait for it to be ready:**
```bash
kubectl wait --for=condition=Available deployment/registry \
  -n openchoreo-build-plane --timeout=180s
```

**Verify:**
```bash
kubectl get pods -n openchoreo-build-plane -l app=docker-registry
```

**Expected output**: Registry pod running

**Learning moment**: For production, you'd use DockerHub, ECR, GCR, etc. For local dev, we use an in-cluster registry.

---

### Command 3: Install Build Plane Helm chart

**What this does**: Installs Argo Workflows and the OpenChoreo build controller.

**Ask your AI agent:**
> "Install the OpenChoreo build plane with all dependencies"

**Command:**
```bash
helm upgrade --install openchoreo-build-plane install/helm/openchoreo-build-plane \
  --dependency-update \
  --namespace openchoreo-build-plane \
  --values install/k3d/single-cluster/values-bp.yaml
```

**This takes 2-3 minutes. Watch the pods come up:**
```bash
kubectl get pods -n openchoreo-build-plane -w
```

Press `Ctrl+C` after all pods are Running.

**Verify:**
```bash
kubectl get pods -n openchoreo-build-plane
```

**Expected output:**
- `cluster-agent-*` - Communicates with control plane
- `workflow-controller-*` - Argo Workflows controller
- `openbao-*` - Secrets management
- `registry-*` - Container registry

**Learning moment**: Build Plane has its own agent that registers with Control Plane, just like Data Plane.

---

### Command 4: Install workflow templates

**What this is**: Reusable build step definitions (checkout, build, publish).

**Ask your AI agent:**
> "Install the ClusterWorkflowTemplates for building and publishing"

**Commands:**
```bash
kubectl apply -f samples/getting-started/workflow-templates/checkout-source.yaml
kubectl apply -f samples/getting-started/workflow-templates.yaml
kubectl apply -f samples/getting-started/workflow-templates/publish-image-k3d.yaml
```

**Verify:**
```bash
kubectl get clusterworkflowtemplate
```

**Expected output**: Templates like:
- `checkout-source`
- `docker-build`
- `react-build`
- `publish-image`
- `generate-workload`

**Learning moment**: These templates are like functions. Workflows call them with parameters (git URL, Dockerfile path, etc.).

---

### Command 5: (Optional) Pre-load buildpack images

**What this does**: Caches buildpack images locally to speed up builds.

**Ask your AI agent:**
> "Pre-load buildpack cache images for faster builds"

**Command:**
```bash
kubectl apply -f install/k3d/common/push-buildpack-cache-images.yaml
```

**Monitor the job:**
```bash
kubectl get jobs -n openchoreo-build-plane -w
```

**This can take 5-10 minutes. You can skip it and continue - builds will just be slower the first time.**

---

### Command 6: Register the Build Plane

**What this does**: Creates a BuildPlane resource so the Control Plane knows about it.

**Ask your AI agent:**
> "Register the build plane with the control plane"

**Command:**
```bash
AGENT_CA=$(kubectl get secret cluster-agent-tls \
  -n openchoreo-build-plane -o jsonpath='{.data.ca\.crt}' | base64 -d)

kubectl apply -f - <<EOF
apiVersion: openchoreo.dev/v1alpha1
kind: BuildPlane
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
    name: openbao
EOF
```

**Verify:**
```bash
kubectl get buildplane -n default
```

**Expected output**: One BuildPlane named "default"

**Check agent connection:**
```bash
kubectl logs -n openchoreo-build-plane -l app=cluster-agent --tail=20
```

**Look for**: "Connected" or "Registered" messages

---

### Command 7: Access Argo Workflows UI

**What this is**: Web UI for monitoring workflow runs.

**Ask your AI agent:**
> "Show me how to access the Argo Workflows UI"

**URL**: http://localhost:10081

**Try it:**
1. Open http://localhost:10081 in your browser
2. You should see the Argo Workflows dashboard
3. No workflows yet - we'll create one next!

**Learning moment**: Argo UI is helpful for visual debugging of build failures.

---

## Phase 3: Deploy Application from Source (15 minutes)

Now let's build and deploy a Go service from source code.

### Command 8: Examine the source-based component

**Ask your AI agent:**
> "Show me the greeting service component definition with source code reference"

**Command:**
```bash
cat samples/from-source/services/go-docker-greeter/greeting-service.yaml
```

**What to look for:**
- `spec.source` - Git repository URL and path
- `spec.build` - Build configuration (Dockerfile location)
- `spec.workflow` - Which workflow template to use
- No `spec.image` - Image will be built, not pulled

**Learning moment**: Instead of `spec.image`, we have `spec.source` and `spec.build`. OpenChoreo will build the image for us.

---

### Command 9: Deploy the component

**Ask your AI agent:**
> "Deploy the greeting service component that builds from source"

**Command:**
```bash
kubectl apply -f samples/from-source/services/go-docker-greeter/greeting-service.yaml
```

**Expected output:**
```
component.openchoreo.dev/greeting-service created
```

**What just happened?**
- Component created
- Controller detected `spec.source` and triggered a WorkflowRun
- Argo Workflows started executing the build

---

### Command 10: Watch the WorkflowRun

**Ask your AI agent:**
> "Show me the WorkflowRun that was created and watch its progress"

**Commands:**
```bash
# List WorkflowRuns
kubectl get workflowrun -n default

# Watch the specific run
kubectl get workflowrun greeting-service-build-01 -n default -w
```

**What to expect:**
- Status starts as `Running`
- After 3-5 minutes, changes to `Succeeded`

**Press Ctrl+C when done**

---

### Command 11: Monitor build pods

**Ask your AI agent:**
> "Show me the pods that are running the build workflow"

**Commands:**
```bash
# List build namespace pods
kubectl get pods -n openchoreo-ci-default

# Watch them
kubectl get pods -n openchoreo-ci-default -w
```

**What you'll see:**
- Pods named like `greeting-service-build-01-*`
- Different pods for each step (checkout, build, publish)
- They complete sequentially

**Press Ctrl+C when done**

**Learning moment**: Each workflow step runs in its own pod. This isolation prevents one failing step from affecting others.

---

### Command 12: View workflow logs in real-time

**Ask your AI agent:**
> "Show me the logs from the build workflow as it runs"

**Command:**
```bash
# Get the latest workflow pod
POD_NAME=$(kubectl get pods -n openchoreo-ci-default \
  -l component-workflows.argoproj.io/workflow=greeting-service-build-01 \
  --sort-by=.metadata.creationTimestamp \
  -o jsonpath='{.items[-1].metadata.name}')

# Follow logs
kubectl logs -n openchoreo-ci-default $POD_NAME -f
```

**What you'll see:**
- Git clone output
- Docker build layers
- Image push confirmation
- Workload generation

**Press Ctrl+C to stop following**

**Learning moment**: These logs are crucial for debugging build failures.

---

### Command 13: Check WorkflowRun status and conditions

**Ask your AI agent:**
> "Show me the detailed status of the WorkflowRun"

**Command:**
```bash
kubectl get workflowrun greeting-service-build-01 -n default -o jsonpath='{.status.conditions}' | jq .
```

**Expected output (when complete):**
```json
[
  {
    "type": "WorkflowCompleted",
    "status": "True"
  },
  {
    "type": "WorkflowSucceeded",
    "status": "True"
  },
  {
    "type": "WorkloadUpdated",
    "status": "True"
  }
]
```

**What these mean:**
- `WorkflowCompleted`: Workflow finished (not stuck)
- `WorkflowSucceeded`: No errors occurred
- `WorkloadUpdated`: Generated Workload was created/updated

---

### Command 14: Verify the Workload was created

**What this is**: The workflow's output - a Workload resource defining the deployment.

**Ask your AI agent:**
> "Show me the Workload that was generated by the build"

**Commands:**
```bash
# List Workloads
kubectl get workload -n default

# Inspect the greeting service workload
kubectl get workload greeting-service-development-default -n default -o yaml
```

**What to look for:**
- `spec.image` - The image that was built and pushed
- `spec.replicas` - Number of instances
- This Workload triggers the deployment (like Module 2)

**Learning moment**: Workflow outputs a Workload. The Workload controller then creates the Deployment, just like in Module 2.

---

## Phase 4: Verify Deployment and Test Application (10 minutes)

### Command 15: Check the ReleaseBinding

**Ask your AI agent:**
> "Verify the ReleaseBinding is ready after the build"

**Command:**
```bash
kubectl get releasebinding greeting-service-development -n default -o jsonpath='{.status.conditions}' | jq .
```

**Expected output**: `Ready: True`

---

### Command 16: Find the running deployment

**Ask your AI agent:**
> "Show me the deployment created for the greeting service"

**Command:**
```bash
kubectl get deployment -A -l openchoreo.dev/component=greeting-service
```

**Verify pods are running:**
```bash
kubectl get pods -A -l openchoreo.dev/component=greeting-service
```

---

### Command 17: Get the service URL

**Ask your AI agent:**
> "Get the URL for the greeting service API"

**Command:**
```bash
HOSTNAME=$(kubectl get httproute -A -l openchoreo.dev/component=greeting-service -o jsonpath='{.items[0].spec.hostnames[0]}')
PATH_PREFIX=$(kubectl get httproute -A -l openchoreo.dev/component=greeting-service -o jsonpath='{.items[0].spec.rules[0].matches[0].path.value}')

echo "Base URL: http://${HOSTNAME}:19080${PATH_PREFIX}"
```

**Expected output:**
```
Base URL: http://development-default.openchoreoapis.localhost:19080/greeting-service
```

---

### Command 18: Test the API

**Ask your AI agent:**
> "Test the greeting service endpoints"

**Commands:**
```bash
HOSTNAME=$(kubectl get httproute -A -l openchoreo.dev/component=greeting-service -o jsonpath='{.items[0].spec.hostnames[0]}')
PATH_PREFIX=$(kubectl get httproute -A -l openchoreo.dev/component=greeting-service -o jsonpath='{.items[0].spec.rules[0].matches[0].path.value}')

# Test basic greet
echo "Testing basic greet:"
curl "http://${HOSTNAME}:19080${PATH_PREFIX}/greeter/greet"
echo -e "\n"

# Test greet with name
echo "Testing greet with name:"
curl "http://${HOSTNAME}:19080${PATH_PREFIX}/greeter/greet?name=Alice"
echo -e "\n"
```

**Expected output:**
```
Testing basic greet:
{"message":"Hello, World!"}

Testing greet with name:
{"message":"Hello, Alice!"}
```

**Success!** You've built and deployed from source code.

---

## Phase 5: Understanding Workflow Templates (10 minutes)

### Command 19: Inspect ClusterWorkflowTemplates

**Ask your AI agent:**
> "Show me the structure of the docker-build workflow template"

**Command:**
```bash
kubectl get clusterworkflowtemplate docker-build -o yaml
```

**What to look for:**
- `spec.templates` - Defines the steps
- `inputs.parameters` - Parameters passed to the template
- `container` - The Docker image used to run the step
- `script` - The actual build commands

**Learning moment**: Templates are reusable. You can create custom templates for different build tools (Maven, npm, etc.).

---

### Command 20: Examine the full workflow composition

**Ask your AI agent:**
> "Show me how the greeting service workflow composes multiple templates"

**Command:**
```bash
kubectl get workflowrun greeting-service-build-01 -n default -o yaml
```

**What to look for:**
- `spec.workflowSpec.templates` - The workflow steps
- References to ClusterWorkflowTemplates:
  - `checkout-source`
  - `docker-build`
  - `publish-image`
  - `generate-workload`

**Trace the flow:**
1. Checkout source from Git
2. Build with Docker
3. Push to registry
4. Generate Workload manifest

**Learning moment**: Workflows are composable. You can mix and match templates to create custom build pipelines.

---

### Command 21: View the Argo Workflows UI

**Ask your AI agent:**
> "Let's view the workflow in the Argo UI for a visual representation"

**Steps:**
1. Open http://localhost:10081
2. Click on `greeting-service-build-01`
3. See the visual DAG (Directed Acyclic Graph)
4. Click each node to see logs, inputs, outputs

**Learning moment**: The UI provides a much better visualization of complex workflows than kubectl.

---

## Phase 6: Triggering Rebuilds (5 minutes)

### Command 22: Trigger a new build

**What this does**: Create a new WorkflowRun to rebuild the application.

**Ask your AI agent:**
> "Show me how to trigger a new build of the greeting service"

**Option 1: Create a new WorkflowRun manually**
```bash
kubectl apply -f - <<EOF
apiVersion: openchoreo.dev/v1alpha1
kind: WorkflowRun
metadata:
  name: greeting-service-build-02
  namespace: default
spec:
  componentRef:
    name: greeting-service
  buildPlaneRef:
    name: default
EOF
```

**Option 2: Edit the Component to trigger automatic rebuild**
```bash
# Add an annotation to trigger rebuild
kubectl annotate component greeting-service -n default \
  openchoreo.dev/rebuild="$(date +%s)" --overwrite
```

**Watch the new build:**
```bash
kubectl get workflowrun -n default -w
```

**Learning moment**: In production, rebuilds are typically triggered by Git webhooks or CI/CD pipelines.

---

## Module 3 Exit Criteria

Before moving to Module 4, verify:

- [ ] Build Plane is installed and healthy
- [ ] BuildPlane resource registered with Control Plane
- [ ] Container registry is running
- [ ] ClusterWorkflowTemplates are installed
- [ ] Successfully built and deployed greeting-service from source
- [ ] WorkflowRun completed with `WorkflowSucceeded: True`
- [ ] Can access the greeting service API and get responses
- [ ] Understand the workflow composition (checkout → build → publish → generate)
- [ ] Can view workflows in Argo UI

### Knowledge Check

**Ask your AI agent these questions:**

1. "What's the difference between building from source (Module 3) vs deploying a pre-built image (Module 2)?"
2. "What are the four main steps in the build workflow?"
3. "Where does the built container image get stored in our local setup?"
4. "What resource does the workflow generate as its final output?"
5. "How would I add a custom build step to the workflow?"

---

## Advanced Exploration (Optional)

### Command 23: Build a different application type

**Try building a React app:**

**Ask your AI agent:**
> "Deploy the React starter app from source using the react-build workflow"

**Command:**
```bash
kubectl apply -f samples/from-source/web-apps/react-starter/react-starter.yaml
```

**Watch the build:**
```bash
kubectl get workflowrun -n default -w
```

**Learning moment**: Different component types use different workflow templates. React uses `react-build`, Go uses `docker-build`.

---

### Command 24: Inspect the local registry

**Ask your AI agent:**
> "Show me the images stored in the local container registry"

**Command:**
```bash
# Port-forward the registry
kubectl port-forward -n openchoreo-build-plane svc/registry 5000:5000 &

# List images (requires curl)
curl -s http://localhost:5000/v2/_catalog | jq .

# Kill the port-forward
pkill -f "port-forward.*registry"
```

**Expected output**: List of built images

**Learning moment**: The registry stores all your built images. In production, use external registries with authentication.

---

### Command 25: Explore build artifacts

**Ask your AI agent:**
> "Show me the workflow outputs and artifacts"

**Command:**
```bash
kubectl get workflowrun greeting-service-build-01 -n default -o jsonpath='{.status.outputs}' | jq .
```

**What to look for:**
- Image digest/tag
- Workload reference
- Build timestamp

---

## Troubleshooting Common Issues

### Issue: WorkflowRun stuck in "Running"

**Diagnosis:**
```bash
# Check workflow pods
kubectl get pods -n openchoreo-ci-default

# Check for pending pods
kubectl describe pods -n openchoreo-ci-default | grep -A 10 Events

# Check workflow controller logs
kubectl logs -n openchoreo-build-plane -l app=workflow-controller --tail=50
```

**Common causes:**
- Not enough resources (CPU/memory)
- Image pull failures
- Network issues

---

### Issue: WorkflowRun failed

**Diagnosis:**
```bash
# Get failure message
kubectl get workflowrun greeting-service-build-01 -n default -o jsonpath='{.status.message}'

# Check conditions
kubectl get workflowrun greeting-service-build-01 -n default -o jsonpath='{.status.conditions}' | jq .

# View failed step logs
kubectl logs -n openchoreo-ci-default <failed-pod-name>
```

**Common causes:**
- Git clone failed (bad URL, private repo without auth)
- Build failed (compilation errors, missing dependencies)
- Image push failed (registry unavailable, auth issues)

---

### Issue: Deployment doesn't start after successful build

**Diagnosis:**
```bash
# Check if Workload was created
kubectl get workload -n default

# Check ReleaseBinding status
kubectl get releasebinding greeting-service-development -n default -o jsonpath='{.status.conditions}' | jq .

# Check controller logs
kubectl logs -n openchoreo-control-plane -l app=controller-manager --tail=50 | grep greeting-service
```

**Common causes:**
- Workload generation step failed
- Controller not watching the Workload
- Image pull failed (registry unreachable from data plane)

---

### Issue: Can't access Argo UI

**Solution:**
```bash
# Verify port mapping
docker ps | grep openchoreo

# Check if argo server is running
kubectl get pods -n openchoreo-build-plane -l app=argo-server

# Try port-forward instead
kubectl port-forward -n openchoreo-build-plane svc/argo-workflows-server 10081:2746
```

---

## Important Commands Reference

**Ask your AI agent to save these:**

### Build Plane Management
```bash
# List WorkflowRuns
kubectl get workflowrun -n default

# Watch a workflow
kubectl get workflowrun <name> -n default -w

# Get workflow status
kubectl get workflowrun <name> -n default -o jsonpath='{.status.conditions}' | jq .

# View workflow pods
kubectl get pods -n openchoreo-ci-default

# View workflow logs
kubectl logs -n openchoreo-ci-default <pod-name>

# List workflow templates
kubectl get clusterworkflowtemplate
```

### Debugging Builds
```bash
# Get latest workflow pod
POD_NAME=$(kubectl get pods -n openchoreo-ci-default \
  --sort-by=.metadata.creationTimestamp \
  -o jsonpath='{.items[-1].metadata.name}')

# Follow logs
kubectl logs -n openchoreo-ci-default $POD_NAME -f

# Check workflow controller
kubectl logs -n openchoreo-build-plane -l app=workflow-controller --tail=50
```

### Registry Operations
```bash
# Port-forward registry
kubectl port-forward -n openchoreo-build-plane svc/registry 5000:5000

# List images
curl http://localhost:5000/v2/_catalog

# Get image tags
curl http://localhost:5000/v2/<image-name>/tags/list
```

---

## Cleanup (Optional)

**Ask your AI agent:**
> "Clean up the greeting service and its build artifacts"

**Commands:**
```bash
# Delete the component (cascades to everything)
kubectl delete component greeting-service -n default

# Delete WorkflowRuns
kubectl delete workflowrun greeting-service-build-01 -n default

# Verify cleanup
kubectl get component,workflowrun,workload,releasebinding -n default | grep greeting
```

---

## Next Steps: Module 4

Once Module 3 is complete, you're ready for **Module 4: Controller/API Architecture Deep Dive**.

In Module 4, you will:
- Explore the OpenChoreo codebase structure
- Understand the controller reconciliation pattern
- Trace how controllers process resources
- Learn the API types and service layer
- Prepare for contributing to controller code

---

## Update Your Progress Log

**Ask your AI agent:**
> "Help me update the progress-log.md file with my Module 3 completion"

Add to [progress-log.md](progress-log.md):
- Date completed
- Successful build and deployment from source
- WorkflowRun details
- Any build issues encountered
- Key learnings about build workflows

---

**Congratulations on completing Module 3!** You now understand how OpenChoreo builds applications from source code using workflow-based CI/CD. This is the foundation for understanding the build controller and contributing to build-related features.
