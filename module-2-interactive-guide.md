# Module 2: Deploy First Workload and Trace Resources - Interactive Guide

**Learn by Doing with Your AI Coding Agent**

This guide teaches you how OpenChoreo transforms declarative configurations into running applications by deploying a sample app and tracing the entire resource chain.

## Prerequisites

Before starting Module 2, ensure Module 1 is complete:
- Control plane is healthy
- Data plane is healthy and registered
- OpenChoreo console is accessible at http://openchoreo.localhost:8080

## Learning Objectives

By the end of this module, you will:
1. Deploy a containerized application to OpenChoreo
2. Understand the declarative resource model (Component → ReleaseBinding → HTTPRoute → Deployment)
3. Trace how OpenChoreo creates and manages Kubernetes resources
4. Verify application deployment and access it
5. Understand the relationship between OpenChoreo CRDs and native Kubernetes resources

## Time Estimate: 30-40 minutes

---

## Phase 1: Understanding OpenChoreo's Deployment Model (5 minutes)

### Key Concepts

**Ask your AI agent:**
> "Explain the OpenChoreo resource hierarchy: Component, ReleaseBinding, HTTPRoute, Deployment"

**The Chain:**
1. **Component**: Your application definition (what to deploy, which image, ports, etc.)
2. **ReleaseBinding**: Binds a component to an environment (dev/staging/prod) and deployment track
3. **HTTPRoute**: Defines how traffic reaches your application (Gateway API resource)
4. **Deployment**: Standard Kubernetes deployment that runs your pods

**Learning moment**: OpenChoreo uses a declarative model. You define WHAT you want (Component), and OpenChoreo's controllers create the HOW (ReleaseBinding, HTTPRoute, Deployment).

### Architecture Visualization

```
Developer Creates:
  └─ Component (CRD)
       ↓ (Controller watches and reconciles)
  └─ ReleaseBinding (CRD) - one per environment
       ↓ (Controller creates)
  └─ HTTPRoute (Gateway API) - routing rules
  └─ Deployment (Kubernetes) - actual pods
       ↓
  └─ Running Application
```

---

## Phase 2: Deploy the React Starter Application (10 minutes)

### Working Directory

**IMPORTANT**: Ensure you're in the right directory:

```bash
cd /Users/lahirus/work/pet-projects/open-choreo/openchoreo-learning/code/openchoreo
```

---

### Command 1: Examine the Component definition

**What this does**: Shows you the declarative manifest before applying it.

**Ask your AI agent:**
> "Show me the contents of the React starter component definition"

**Command:**
```bash
curl -s https://raw.githubusercontent.com/openchoreo/openchoreo/main/samples/from-image/react-starter-web-app/react-starter.yaml
```

**Or use the local file:**
```bash
cat samples/from-image/react-starter-web-app/react-starter.yaml
```

**What to look for:**
- `kind: Component` - This is an OpenChoreo CRD
- `spec.image` - The container image to deploy
- `spec.port` - The port the app listens on
- Environment bindings (development, staging, production)

**Learning moment**: This single YAML file defines the entire application lifecycle across multiple environments.

---

### Command 2: Deploy the Component

**Ask your AI agent:**
> "Deploy the React starter component to OpenChoreo"

**Command:**
```bash
kubectl apply -f samples/from-image/react-starter-web-app/react-starter.yaml
```

**Expected output:**
```
component.openchoreo.dev/react-starter created
```

**What just happened?**
- You created a Component resource
- OpenChoreo controllers will now automatically create ReleaseBindings for each environment
- The data plane will create HTTPRoutes and Deployments

---

### Command 3: Watch the reconciliation happen

**Ask your AI agent:**
> "Watch the component status until it's ready"

**Command:**
```bash
kubectl get component react-starter -n default -w
```

**What to expect:**
- Status will change as controllers reconcile
- Press `Ctrl+C` to stop watching after 30-60 seconds

**Alternative - check once:**
```bash
kubectl get component react-starter -n default
```

---

## Phase 3: Trace the Resource Chain (15 minutes)

Now we'll follow the trail from Component → Running Application.

### Command 4: Inspect the Component

**Ask your AI agent:**
> "Show me the full Component resource with its status"

**Command:**
```bash
kubectl get component react-starter -n default -o yaml
```

**What to look for:**
- `spec.image`: The container image being used
- `spec.port`: Port 8080
- `spec.bindings`: References to environments (development, staging, production)
- `status.conditions`: Shows if the component is healthy

**Learning moment**: The Component is the "source of truth" - everything else derives from this.

---

### Command 5: Find the ReleaseBinding

**What this is**: A ReleaseBinding connects your Component to a specific environment.

**Ask your AI agent:**
> "List all ReleaseBindings for the react-starter component"

**Command:**
```bash
kubectl get releasebinding -n default -l openchoreo.dev/component=react-starter
```

**Expected output**: You should see ReleaseBindings for each environment:
- `react-starter-development`
- `react-starter-staging`
- `react-starter-production`

**Inspect the development ReleaseBinding:**
```bash
kubectl get releasebinding react-starter-development -n default -o yaml
```

**What to look for:**
- `spec.componentRef`: Points to the Component
- `spec.environmentRef`: Points to the environment (development)
- `status.conditions`: Shows if deployment is ready
- `status.observedGeneration`: Used for tracking updates

**Learning moment**: Each environment gets its own ReleaseBinding. This allows progressive rollout (deploy to dev, then staging, then prod).

---

### Command 6: Find the HTTPRoute

**What this is**: Defines how external traffic reaches your application.

**Ask your AI agent:**
> "Find the HTTPRoute created for the react-starter component"

**Command:**
```bash
kubectl get httproute -A -l openchoreo.dev/component=react-starter
```

**Expected output**: HTTPRoutes in the data plane namespace.

**Inspect one:**
```bash
kubectl get httproute -A -l openchoreo.dev/component=react-starter -o yaml | head -80
```

**What to look for:**
- `spec.hostnames`: The URL(s) for your app
- `spec.rules.backendRefs`: Points to the Kubernetes Service
- Namespace: Deployed in the data plane namespace

**Get the URL:**
```bash
HOSTNAME=$(kubectl get httproute -A -l openchoreo.dev/component=react-starter -o jsonpath='{.items[0].spec.hostnames[0]}')
echo "Application URL: http://${HOSTNAME}:19080"
```

**Learning moment**: HTTPRoute is a Gateway API standard resource. OpenChoreo automatically generates the correct routing rules.

---

### Command 7: Find the Deployment

**What this is**: The actual Kubernetes workload running your containers.

**Ask your AI agent:**
> "Find the Kubernetes Deployment for react-starter"

**Command:**
```bash
kubectl get deployment -A -l openchoreo.dev/component=react-starter
```

**Inspect it:**
```bash
kubectl get deployment -A -l openchoreo.dev/component=react-starter -o yaml | head -100
```

**What to look for:**
- `spec.replicas`: Number of instances
- `spec.template.spec.containers`: Container definitions
- `spec.template.spec.containers[0].image`: The actual image
- Labels: OpenChoreo adds labels for tracking

**Check the pods:**
```bash
kubectl get pods -A -l openchoreo.dev/component=react-starter
```

**Learning moment**: The Deployment is standard Kubernetes. OpenChoreo adds labels and annotations for tracking and management.

---

### Command 8: Visualize the complete chain

**Ask your AI agent:**
> "Create a summary showing the complete resource chain from Component to Pods"

**Commands:**
```bash
echo "=== COMPONENT ==="
kubectl get component react-starter -n default -o jsonpath='{.metadata.name}{"\n  Type: "}{.kind}{"\n  Image: "}{.spec.image}{"\n  Port: "}{.spec.port}{"\n\n"}'

echo "=== RELEASE BINDINGS ==="
kubectl get releasebinding -n default -l openchoreo.dev/component=react-starter -o jsonpath='{range .items[*]}{.metadata.name}{" -> "}{.spec.environmentRef.name}{"\n"}{end}{"\n"}'

echo "=== HTTP ROUTES ==="
kubectl get httproute -A -l openchoreo.dev/component=react-starter -o jsonpath='{range .items[*]}{.metadata.name}{" in "}{.metadata.namespace}{"\n  Hostname: "}{.spec.hostnames[0]}{"\n"}{end}{"\n"}'

echo "=== DEPLOYMENTS ==="
kubectl get deployment -A -l openchoreo.dev/component=react-starter -o jsonpath='{range .items[*]}{.metadata.name}{" in "}{.metadata.namespace}{"\n  Replicas: "}{.spec.replicas}{"\n  Image: "}{.spec.template.spec.containers[0].image}{"\n"}{end}{"\n"}'

echo "=== PODS ==="
kubectl get pods -A -l openchoreo.dev/component=react-starter -o jsonpath='{range .items[*]}{.metadata.name}{" -> "}{.status.phase}{"\n"}{end}'
```

**Expected output**: A clear hierarchy showing the entire chain.

---

## Phase 4: Access and Verify the Application (5 minutes)

### Command 9: Get the application URL

**Ask your AI agent:**
> "Give me the URL to access the react-starter application"

**Command:**
```bash
HOSTNAME=$(kubectl get httproute -A -l openchoreo.dev/component=react-starter -o jsonpath='{.items[0].spec.hostnames[0]}')
echo "Access the application at: http://${HOSTNAME}:19080"
```

**Expected output:**
```
Access the application at: http://react-starter-development-default.openchoreoapis.localhost:19080
```

---

### Command 10: Test the application

**Option 1: Use curl**

**Ask your AI agent:**
> "Test the application endpoint using curl"

**Command:**
```bash
HOSTNAME=$(kubectl get httproute -A -l openchoreo.dev/component=react-starter -o jsonpath='{.items[0].spec.hostnames[0]}')
curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" http://${HOSTNAME}:19080
```

**Expected output:** `HTTP Status: 200`

**Option 2: Use your browser**

Open the URL in your browser:
```
http://react-starter-development-default.openchoreoapis.localhost:19080
```

You should see the React starter application.

---

### Command 11: Check application logs

**Ask your AI agent:**
> "Show me the logs from the react-starter application pods"

**Command:**
```bash
kubectl logs -n $(kubectl get deployment -A -l openchoreo.dev/component=react-starter -o jsonpath='{.items[0].metadata.namespace}') \
  -l openchoreo.dev/component=react-starter --tail=20
```

**What to look for:**
- HTTP requests being logged
- No error messages
- Server startup messages

---

## Phase 5: Understanding Controller Behavior (5 minutes)

### Command 12: Watch what happens when you delete a resource

**Learning experiment**: Delete the Deployment and watch OpenChoreo recreate it.

**Ask your AI agent:**
> "Delete the react-starter Deployment and watch OpenChoreo recreate it automatically"

**Commands:**
```bash
# Find and delete the deployment
DEPLOYMENT_NAME=$(kubectl get deployment -A -l openchoreo.dev/component=react-starter -o jsonpath='{.items[0].metadata.name}')
DEPLOYMENT_NS=$(kubectl get deployment -A -l openchoreo.dev/component=react-starter -o jsonpath='{.items[0].metadata.namespace}')

echo "Deleting deployment: $DEPLOYMENT_NAME in namespace $DEPLOYMENT_NS"
kubectl delete deployment $DEPLOYMENT_NAME -n $DEPLOYMENT_NS

# Watch it come back
echo "Watching for recreation..."
kubectl get deployment -n $DEPLOYMENT_NS -l openchoreo.dev/component=react-starter -w
```

**What you'll observe:**
- The Deployment disappears
- Within seconds, the controller recreates it
- This is the "reconciliation loop" in action

**Press Ctrl+C to stop watching**

**Learning moment**: This is the power of declarative systems. The controller constantly ensures reality matches your desired state.

---

### Command 13: View controller logs

**Ask your AI agent:**
> "Show me recent controller logs to see the reconciliation in action"

**Command:**
```bash
kubectl logs -n openchoreo-control-plane -l app=controller-manager --tail=50 | grep -i react-starter
```

**What to look for:**
- Reconcile events
- Resource creation logs
- Status updates

---

## Phase 6: Exploring Resource Relationships (5 minutes)

### Command 14: Understand label selectors

**What this is**: Labels are how Kubernetes (and OpenChoreo) track relationships between resources.

**Ask your AI agent:**
> "Show me all the labels on the react-starter component and related resources"

**Commands:**
```bash
echo "=== Component Labels ==="
kubectl get component react-starter -n default --show-labels

echo -e "\n=== ReleaseBinding Labels ==="
kubectl get releasebinding -n default -l openchoreo.dev/component=react-starter --show-labels

echo -e "\n=== HTTPRoute Labels ==="
kubectl get httproute -A -l openchoreo.dev/component=react-starter --show-labels

echo -e "\n=== Deployment Labels ==="
kubectl get deployment -A -l openchoreo.dev/component=react-starter --show-labels

echo -e "\n=== Pod Labels ==="
kubectl get pods -A -l openchoreo.dev/component=react-starter --show-labels
```

**Learning moment**: The label `openchoreo.dev/component=react-starter` is how all these resources are connected. Controllers use label selectors to find related resources.

---

### Command 15: Trace ownership references

**Ask your AI agent:**
> "Show me the ownership chain for react-starter resources"

**Command:**
```bash
kubectl get releasebinding react-starter-development -n default -o jsonpath='{.metadata.ownerReferences}' | jq .
```

**What to look for:**
- `ownerReferences` field shows which resource "owns" this one
- When an owner is deleted, owned resources are automatically deleted (cascade deletion)

**Learning moment**: Kubernetes uses owner references for garbage collection. Delete a Component, and all its ReleaseBindings are automatically deleted.

---

## Phase 7: Exploring Multiple Environments (5 minutes)

### Command 16: Compare development vs production deployments

**Ask your AI agent:**
> "Compare the ReleaseBindings for different environments"

**Commands:**
```bash
echo "=== Development ReleaseBinding ==="
kubectl get releasebinding react-starter-development -n default -o yaml | grep -A 10 "spec:"

echo -e "\n=== Production ReleaseBinding ==="
kubectl get releasebinding react-starter-production -n default -o yaml | grep -A 10 "spec:"
```

**What to look for:**
- Different `environmentRef` values
- Same `componentRef` (same app, different environments)
- Potentially different configurations (replicas, resources, etc.)

---

### Command 17: Check status across environments

**Ask your AI agent:**
> "Show me the status of react-starter across all environments"

**Command:**
```bash
kubectl get releasebinding -n default -l openchoreo.dev/component=react-starter -o jsonpath='{range .items[*]}{.metadata.name}{": "}{.status.conditions[?(@.type=="Ready")].status}{"\n"}{end}'
```

**Expected output:**
```
react-starter-development: True
react-starter-staging: True
react-starter-production: True
```

**Learning moment**: Each environment is independently managed. You could update dev without touching production.

---

## Module 2 Exit Criteria

Before moving to Module 3, verify:

- [ ] Successfully deployed the react-starter component
- [ ] Can trace the chain: Component → ReleaseBinding → HTTPRoute → Deployment → Pods
- [ ] Application is accessible at the generated URL
- [ ] Can explain what each resource type does
- [ ] Understand how labels connect resources
- [ ] Understand the controller reconciliation loop
- [ ] Can explain the difference between declarative and imperative systems

### Knowledge Check

**Ask your AI agent these questions to test your understanding:**

1. "What happens if I delete a Component? Walk me through the cascade."
2. "Why does OpenChoreo create separate ReleaseBindings for each environment?"
3. "What Kubernetes resource actually runs the application containers?"
4. "How does traffic reach my application? Trace the path from external request to pod."
5. "What label selector would I use to find all resources for a specific component?"

---

## Advanced Exploration (Optional)

### Command 18: Edit a Component and watch the update propagate

**Ask your AI agent:**
> "Help me update the react-starter component to use 2 replicas and watch the change propagate"

**Steps:**
1. Edit the Component:
```bash
kubectl edit component react-starter -n default
```

2. Find the `spec:` section and add:
```yaml
spec:
  replicas: 2  # Add this line
```

3. Save and exit (`:wq` in vim)

4. Watch the Deployment update:
```bash
kubectl get deployment -A -l openchoreo.dev/component=react-starter -w
```

5. Verify 2 pods are running:
```bash
kubectl get pods -A -l openchoreo.dev/component=react-starter
```

**Learning moment**: Changes to the Component propagate through the entire chain automatically.

---

### Command 19: View the OpenChoreo console

**Ask your AI agent:**
> "Show me how to view the react-starter component in the OpenChoreo console"

**Steps:**
1. Open http://openchoreo.localhost:8080
2. Log in (credentials from `install/k3d/common/values-thunder.yaml`)
3. Navigate to Components → react-starter
4. Explore the UI view of what we just did via CLI

**Learning moment**: The console provides a visual representation of everything we traced via `kubectl`.

---

## Cleanup (Optional)

If you want to remove the react-starter component:

**Ask your AI agent:**
> "Clean up the react-starter component and all related resources"

**Command:**
```bash
kubectl delete component react-starter -n default
```

**Verify cleanup:**
```bash
# These should all return empty
kubectl get releasebinding -n default -l openchoreo.dev/component=react-starter
kubectl get httproute -A -l openchoreo.dev/component=react-starter
kubectl get deployment -A -l openchoreo.dev/component=react-starter
kubectl get pods -A -l openchoreo.dev/component=react-starter
```

**Learning moment**: Deleting the Component cascades to all owned resources. This is Kubernetes garbage collection in action.

---

## Important Commands Reference

**Ask your AI agent to save these for you:**

### Tracing Resources
```bash
# Find all resources for a component
kubectl get all -A -l openchoreo.dev/component=<component-name>

# Get component status
kubectl get component <name> -n default -o yaml

# Get release bindings
kubectl get releasebinding -n default -l openchoreo.dev/component=<name>

# Get HTTP routes
kubectl get httproute -A -l openchoreo.dev/component=<name>

# Get deployments
kubectl get deployment -A -l openchoreo.dev/component=<name>
```

### Debugging
```bash
# Check component conditions
kubectl get component <name> -n default -o jsonpath='{.status.conditions}' | jq .

# Check release binding status
kubectl get releasebinding <name> -n default -o jsonpath='{.status.conditions}' | jq .

# View pod logs
kubectl logs -n <namespace> -l openchoreo.dev/component=<name> --tail=50

# Describe a resource
kubectl describe component <name> -n default
```

---

## Troubleshooting Common Issues

### Issue: Application not accessible
**Diagnosis:**
```bash
# Check ReleaseBinding status
kubectl get releasebinding react-starter-development -n default -o jsonpath='{.status.conditions}' | jq .

# Check HTTPRoute
kubectl get httproute -A -l openchoreo.dev/component=react-starter -o yaml

# Check deployment
kubectl get deployment -A -l openchoreo.dev/component=react-starter

# Check pods
kubectl get pods -A -l openchoreo.dev/component=react-starter
```

**Common causes:**
- Deployment not ready (check pod status)
- HTTPRoute misconfigured (check hostname and backend refs)
- Gateway not running (check kgateway pods)

---

### Issue: Pods in CrashLoopBackOff
**Diagnosis:**
```bash
# Get pod logs
kubectl logs -n <namespace> <pod-name>

# Describe pod for events
kubectl describe pod <pod-name> -n <namespace>
```

**Common causes:**
- Container image not found
- Application error on startup
- Port mismatch (app listens on different port than specified)

---

### Issue: Cannot find resources
**Cause**: Wrong namespace or label selector

**Solution:**
```bash
# Search all namespaces
kubectl get all -A -l openchoreo.dev/component=react-starter

# List all components
kubectl get components -A
```

---

## Next Steps: Module 3

Once Module 2 is complete, you're ready for **Module 3: Build Plane + Source Build/Deploy**.

In Module 3, you will:
- Install the Build Plane
- Deploy an application from source code (not pre-built image)
- Watch the build workflow execute
- Understand how OpenChoreo builds and publishes container images

---

## Update Your Progress Log

**Ask your AI agent:**
> "Help me update the progress-log.md file with my Module 2 completion"

Add to [progress-log.md](progress-log.md):
- Date completed
- Component deployed
- Resources traced
- Key learnings about the resource chain

---

**Congratulations on completing Module 2!** You now understand how OpenChoreo transforms declarative components into running applications and can trace the entire resource chain. This knowledge is essential for contributing to the controller codebase.
