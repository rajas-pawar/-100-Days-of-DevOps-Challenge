# Day 57: Kubernetes Environment Variables and Pod Configuration

## Objective
Configure a Kubernetes pod with environment variables and custom commands to test application prerequisites. This demonstrates how to pass configuration to containers using environment variables—a fundamental pattern for application configuration in Kubernetes.

## Task Requirements
- **Environment:** Kubernetes cluster accessible from jump_host
- **Pod Name:** print-envars-greeting
- **Container Name:** print-env-container
- **Image:** bash
- **Environment Variables:**
  - GREETING = "Welcome to"
  - COMPANY = "DevOps"
  - GROUP = "Datacenter"
- **Command:** `["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']`
- **Restart Policy:** Never
- **Verification:** kubectl logs command
- **Tool:** kubectl (pre-configured on jump_host)

## Understanding Kubernetes Environment Variables

### What are Environment Variables?

**Environment Variables** are key-value pairs that provide configuration to applications running in containers. They're a fundamental mechanism for:
- **Configuration:** Database URLs, API keys, feature flags
- **Secrets:** Credentials, tokens (via Secrets resource)
- **Discovery:** Service endpoints, cluster information
- **Behavior:** Debug modes, log levels, timeouts

### Why Use Environment Variables?

**Without Environment Variables (Hardcoded):**
```dockerfile
# In application code
DATABASE_URL = "mysql://prod-db:3306/app"  ❌ Hardcoded!

Problems:
❌ Must rebuild image for each environment
❌ Secrets exposed in code
❌ No flexibility
❌ Cannot reuse image across dev/staging/prod
```

**With Environment Variables (Configurable):**
```yaml
# In Kubernetes manifest
env:
- name: DATABASE_URL
  value: "mysql://prod-db:3306/app"  ✅ Configurable!

Benefits:
✅ Same image for all environments
✅ Configuration externalized
✅ Secrets separate from code
✅ Easy to modify without rebuilds
```

### The 12-Factor App Principle

**12-Factor App (Config):**
> "Store config in the environment"

**What This Means:**
```
Dev Environment:
├── Same Docker image
├── ENV: DATABASE_URL=mysql://dev-db:3306/app
└── ENV: DEBUG=true

Staging Environment:
├── Same Docker image
├── ENV: DATABASE_URL=mysql://staging-db:3306/app
└── ENV: DEBUG=false

Production Environment:
├── Same Docker image
├── ENV: DATABASE_URL=mysql://prod-db:3306/app
└── ENV: DEBUG=false
```

**Benefits:**
- ✅ **Immutable Images:** Same image across environments
- ✅ **No Secrets in Code:** Credentials injected at runtime
- ✅ **Easy Updates:** Change config without rebuilding
- ✅ **Environment Parity:** Dev/staging/prod consistency

### Environment Variables in Kubernetes

**Four Ways to Set Environment Variables:**

**1. Direct Value (Simple):**
```yaml
env:
- name: GREETING
  value: "Welcome to"  # Hardcoded value
```

**2. From ConfigMap:**
```yaml
env:
- name: APP_CONFIG
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: config.json  # Value from ConfigMap
```

**3. From Secret:**
```yaml
env:
- name: DATABASE_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password  # Value from Secret (encrypted)
```

**4. From Field Reference:**
```yaml
env:
- name: POD_NAME
  valueFrom:
    fieldRef:
      fieldPath: metadata.name  # Pod's own name
```

**For This Task:** We'll use method #1 (direct values)

### Understanding Pod Restart Policies

**Restart Policy** determines what happens when a container exits.

**Three Options:**

**1. Always (Default):**
```yaml
restartPolicy: Always

Behavior:
Container exits → Kubernetes restarts it (forever)
Use case: Long-running services (web servers, APIs)
```

**2. OnFailure:**
```yaml
restartPolicy: OnFailure

Behavior:
Container succeeds (exit 0) → No restart ✅
Container fails (exit 1+) → Restart
Use case: Batch jobs that should retry on failure
```

**3. Never:**
```yaml
restartPolicy: Never

Behavior:
Container exits (any code) → Never restart
Use case: One-time tasks, testing, debug pods
```

**For This Task:** We use `Never` because:
- Pod runs once, prints message, exits
- No need to restart (would create infinite logs)
- Prevents CrashLoopBackOff state

### CrashLoopBackOff Explained

**What is CrashLoopBackOff?**

When a pod keeps restarting with increasing delays:

```
Attempt 1: Container starts → exits → restart after 0s
Attempt 2: Container starts → exits → restart after 10s
Attempt 3: Container starts → exits → restart after 20s
Attempt 4: Container starts → exits → restart after 40s
Attempt 5: Container starts → exits → restart after 80s (max)
...continues with exponential backoff
```

**Why It Happens:**
```yaml
# Pod with restartPolicy: Always (default)
spec:
  restartPolicy: Always
  containers:
  - name: my-container
    command: ["echo", "Hello"]  # Exits immediately

Result:
1. Container runs echo
2. Echo completes and exits
3. Kubernetes sees exit, restarts (due to Always)
4. Repeat infinitely → CrashLoopBackOff ❌
```

**How to Prevent:**
```yaml
# Use restartPolicy: Never for one-time commands
spec:
  restartPolicy: Never  ✅
  containers:
  - name: my-container
    command: ["echo", "Hello"]

Result:
1. Container runs echo
2. Echo completes and exits
3. Kubernetes does NOT restart
4. Pod status: Completed ✅
```

### Command vs Args in Kubernetes

**Understanding the Difference:**

**In Dockerfile:**
```dockerfile
ENTRYPOINT ["python"]  # The executable
CMD ["app.py"]         # Default arguments
```

**In Kubernetes:**
```yaml
command: ["/bin/sh"]   # Overrides ENTRYPOINT
args: ["-c", "echo hi"]  # Overrides CMD
```

**Examples:**

**1. Use Image Defaults:**
```yaml
containers:
- name: nginx
  image: nginx:latest
  # No command or args → uses nginx defaults
  # Starts: nginx -g "daemon off;"
```

**2. Override Command Only:**
```yaml
containers:
- name: bash
  image: bash
  command: ["/bin/bash"]
  # Runs: /bin/bash (no arguments)
```

**3. Override Command and Args:**
```yaml
containers:
- name: bash
  image: bash
  command: ["/bin/sh"]
  args: ["-c", "echo Hello"]
  # Runs: /bin/sh -c "echo Hello"
```

**4. Inline Command (This Task):**
```yaml
containers:
- name: print-env-container
  image: bash
  command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
  # Runs: /bin/sh -c 'echo "$(GREETING) $(COMPANY) $(GROUP)"'
  # All in command field (no separate args)
```

### Environment Variable Substitution

**How $(VAR) Works:**

**Shell Substitution:**
```bash
# In shell script
GREETING="Welcome to"
echo "$(GREETING)"  # Outputs: Welcome to
```

**Kubernetes Environment Variables:**
```yaml
env:
- name: GREETING
  value: "Welcome to"
- name: COMPANY
  value: "DevOps"

command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY)"']

# What happens:
# 1. Kubernetes injects env vars into container
# 2. Shell command runs: /bin/sh -c 'echo "$(GREETING) $(COMPANY)"'
# 3. Shell substitutes: "Welcome to DevOps"
# 4. Output: Welcome to DevOps
```

**Important:** The `$(VAR)` syntax is **shell substitution**, not Kubernetes templating!

**Kubernetes Variable Syntax:**
```yaml
# This is KUBERNETES substitution (only in specific fields):
$(VARIABLE)

# This is SHELL substitution (in command/args):
'echo "$(VARIABLE)"'  # Shell interprets this

# For this task, we use shell substitution
```

## Understanding the Bash Image

### What is the Bash Image?

**Docker Hub:** `bash` (official image)
- Minimal Alpine Linux with Bash shell
- Size: ~13 MB
- Use case: Running bash scripts, shell commands

**Why Use Bash Image?**
```
Regular Images (nginx, python, etc.):
├── Start a service (web server, app)
└── Keep running forever

Bash Image:
├── No service started
├── Just provides bash shell
├── Exits immediately unless given command
└── Perfect for running one-time scripts ✅
```

**For This Task:**
- We need to run a shell command (`echo`)
- Bash image provides the shell
- Command executes and exits
- Perfect for testing environment variables

## Step-by-Step Implementation

### Phase 1: Create Pod with Environment Variables

#### Step 1: Create Pod YAML

```bash
# Create pod manifest
vi print-envars-greeting.yaml
```

**Complete Pod Configuration:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
spec:
  restartPolicy: Never
  containers:
  - name: print-env-container
    image: bash
    env:
    - name: GREETING
      value: "Welcome to"
    - name: COMPANY
      value: "DevOps"
    - name: GROUP
      value: "Datacenter"
    command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
```

**YAML Breakdown:**

**1. API Version and Kind:**
```yaml
apiVersion: v1             # Core API
kind: Pod                  # Resource type
```

**2. Metadata:**
```yaml
metadata:
  name: print-envars-greeting  # Pod name (required)
```

**3. Restart Policy:**
```yaml
spec:
  restartPolicy: Never     # Don't restart after exit (critical!)
```

**Why Never?**
- Command runs once and exits
- Without Never → would restart infinitely
- Prevents CrashLoopBackOff

**4. Container Specification:**
```yaml
  containers:
  - name: print-env-container  # Container name (required)
    image: bash                # Bash image for shell commands
```

**5. Environment Variables:**
```yaml
    env:
    - name: GREETING           # Variable name
      value: "Welcome to"      # Variable value
    - name: COMPANY
      value: "DevOps"
    - name: GROUP
      value: "Datacenter"
```

**How They Work:**
```
Container sees:
├── GREETING="Welcome to"
├── COMPANY="DevOps"
└── GROUP="Datacenter"

Just like running:
export GREETING="Welcome to"
export COMPANY="DevOps"
export GROUP="Datacenter"
```

**6. Command:**
```yaml
    command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
```

**Command Breakdown:**
```
/bin/sh                    # Execute shell
-c                         # Run following string as command
'echo "$(GREETING) $(COMPANY) $(GROUP)"'  # Command to run

Equivalent to:
/bin/sh -c 'echo "$(GREETING) $(COMPANY) $(GROUP)"'

Shell substitutes variables:
echo "Welcome to DevOps Datacenter"

Output:
Welcome to DevOps Datacenter
```

**Alternative YAML Format (Multi-line):**
```yaml
# You can also write command this way:
command:
  - /bin/sh
  - -c
  - 'echo "$(GREETING) $(COMPANY) $(GROUP)"'

# Or with args:
command: ["/bin/sh"]
args:
  - -c
  - 'echo "$(GREETING) $(COMPANY) $(GROUP)"'

# All three formats are equivalent
```

#### Step 2: Validate Pod YAML

```bash
# Dry-run validation
kubectl apply -f print-envars-greeting.yaml --dry-run=client

# Expected Output:
# pod/print-envars-greeting created (dry run)

# Check YAML syntax
cat print-envars-greeting.yaml
```

**Validation Checklist:**
- ✅ apiVersion: v1
- ✅ kind: Pod
- ✅ name: print-envars-greeting
- ✅ restartPolicy: Never
- ✅ container name: print-env-container
- ✅ image: bash
- ✅ env: 3 variables (GREETING, COMPANY, GROUP)
- ✅ command: exact as specified
- ✅ No YAML syntax errors

**Common YAML Errors to Avoid:**

**Error 1: Wrong Indentation:**
```yaml
❌ WRONG:
spec:
  restartPolicy: Never
  containers:
  - name: print-env-container
    image: bash
  env:  # Wrong level! Should be under containers
  - name: GREETING
    value: "Welcome to"

✅ CORRECT:
spec:
  restartPolicy: Never
  containers:
  - name: print-env-container
    image: bash
    env:  # Correct level (under container)
    - name: GREETING
      value: "Welcome to"
```

**Error 2: Missing Quotes in Command:**
```yaml
❌ WRONG:
command: ["/bin/sh", "-c", echo "$(GREETING) $(COMPANY) $(GROUP)"]
# Missing quotes around echo command

✅ CORRECT:
command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
# Single quotes around entire echo command
```

**Error 3: Wrong restartPolicy Value:**
```yaml
❌ WRONG:
restartPolicy: never  # Lowercase

✅ CORRECT:
restartPolicy: Never  # Capital N
```

#### Step 3: Apply Pod Configuration

```bash
# Create pod
kubectl apply -f print-envars-greeting.yaml

# Expected Output:
# pod/print-envars-greeting created
```

**What Happens Behind the Scenes:**

```
1. kubectl sends YAML to API server
2. API server validates and stores in etcd
3. Scheduler assigns pod to a node
4. Kubelet on node pulls bash image
5. Kubelet creates container with:
   - Environment variables: GREETING, COMPANY, GROUP
   - Command: /bin/sh -c 'echo "..."'
6. Container starts
7. Command executes (prints message)
8. Container exits (exit code 0)
9. Pod status: Completed (due to restartPolicy: Never)
```

#### Step 4: Verify Pod Status

```bash
# Check pod status
kubectl get pods

# Expected Output (initial):
# NAME                     READY   STATUS              RESTARTS   AGE
# print-envars-greeting    0/1     ContainerCreating   0          2s
```

**Wait a few seconds, then check again:**
```bash
kubectl get pods

# Expected Output (after completion):
# NAME                     READY   STATUS      RESTARTS   AGE
# print-envars-greeting    0/1     Completed   0          10s
```

**Status Meanings:**

**ContainerCreating:**
```
Status: ContainerCreating
Meaning: 
├── Image being pulled
├── Container being created
└── Not yet running
Time: Usually 5-30 seconds
```

**Completed:**
```
Status: Completed
Meaning:
├── Container ran successfully
├── Command executed
├── Exited with code 0
├── Won't restart (due to restartPolicy: Never)
└── Logs available ✅
```

**If You See Running (Unexpected):**
```bash
# NAME                     READY   STATUS    RESTARTS   AGE
# print-envars-greeting    1/1     Running   0          5s

# This means command hasn't finished yet
# Wait a moment and check again
kubectl get pods
# Should show Completed
```

#### Step 5: View Pod Details

```bash
# Detailed pod information
kubectl describe pod print-envars-greeting
```

**Expected Output (Key Sections):**
```
Name:             print-envars-greeting
Namespace:        default
Status:           Succeeded
IP:               10.244.1.5
Containers:
  print-env-container:
    Image:          bash
    Command:
      /bin/sh
      -c
      echo "$(GREETING) $(COMPANY) $(GROUP)"
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
    Environment:
      GREETING:   Welcome to
      COMPANY:    DevOps
      GROUP:      Datacenter
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  20s   default-scheduler  Successfully assigned default/print-envars-greeting to node01
  Normal  Pulling    19s   kubelet            Pulling image "bash"
  Normal  Pulled     15s   kubelet            Successfully pulled image "bash"
  Normal  Created    15s   kubelet            Created container print-env-container
  Normal  Started    15s   kubelet            Started container print-env-container
```

**Key Information:**
- **Status:** Succeeded ✅ (pod completed successfully)
- **Command:** Shows our exact command
- **Environment:** All 3 variables present ✅
- **Exit Code:** 0 ✅ (success)
- **Events:** Timeline of pod lifecycle

### Phase 2: View Output

#### Step 6: View Container Logs

```bash
# View logs (output of echo command)
kubectl logs print-envars-greeting

# Expected Output:
# Welcome to DevOps Datacenter
```

**What You Should See:**
```
Welcome to DevOps Datacenter
```

**This confirms:**
- ✅ Environment variables were set correctly
- ✅ Command executed successfully
- ✅ Shell substitution worked ($(VAR) replaced with values)
- ✅ Output matches expected greeting

**Alternative Log Commands:**

```bash
# Follow logs (useful if container still running)
kubectl logs -f print-envars-greeting

# View logs with timestamps
kubectl logs print-envars-greeting --timestamps

# View previous logs (if container restarted)
kubectl logs print-envars-greeting --previous

# Specify container name (useful for multi-container pods)
kubectl logs print-envars-greeting -c print-env-container
```

#### Step 7: Verify Environment Variables

```bash
# Check environment variables in pod spec
kubectl get pod print-envars-greeting -o yaml | grep -A 10 "env:"

# Expected Output:
# env:
# - name: GREETING
#   value: Welcome to
# - name: COMPANY
#   value: DevOps
# - name: GROUP
#   value: Datacenter
```

**Or more detailed:**
```bash
# Get all environment variables in JSON format
kubectl get pod print-envars-greeting -o jsonpath='{.spec.containers[0].env[*]}'

# Pretty print with jq
kubectl get pod print-envars-greeting -o json | jq '.spec.containers[0].env'

# Expected Output:
# [
#   {
#     "name": "GREETING",
#     "value": "Welcome to"
#   },
#   {
#     "name": "COMPANY",
#     "value": "DevOps"
#   },
#   {
#     "name": "GROUP",
#     "value": "Datacenter"
#   }
# ]
```

#### Step 8: Verify Command Execution

```bash
# Check command in pod spec
kubectl get pod print-envars-greeting -o yaml | grep -A 5 "command:"

# Expected Output:
# command:
# - /bin/sh
# - -c
# - echo "$(GREETING) $(COMPANY) $(GROUP)"
```

**Or JSON format:**
```bash
# Get command
kubectl get pod print-envars-greeting -o jsonpath='{.spec.containers[0].command}'

# Expected Output:
# ["/bin/sh","-c","echo \"$(GREETING) $(COMPANY) $(GROUP)\""]
```

### Phase 3: Testing and Validation

#### Step 9: Test with Different Environment Variables

**Create a test pod with different values:**
```bash
vi test-envars.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-envars
spec:
  restartPolicy: Never
  containers:
  - name: test-container
    image: bash
    env:
    - name: GREETING
      value: "Hello from"
    - name: COMPANY
      value: "Kubernetes"
    - name: GROUP
      value: "Cluster"
    command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
```

```bash
# Apply test pod
kubectl apply -f test-envars.yaml

# Wait for completion
kubectl get pods test-envars

# View output
kubectl logs test-envars

# Expected Output:
# Hello from Kubernetes Cluster
```

**Cleanup test pod:**
```bash
kubectl delete pod test-envars
```

#### Step 10: Test Variable Interpolation

**Create pod that shows variable usage:**
```bash
kubectl run env-test --image=bash --restart=Never \
  --env="VAR1=First" \
  --env="VAR2=Second" \
  --env="VAR3=Third" \
  -- /bin/sh -c 'echo "Var1: $VAR1, Var2: $VAR2, Var3: $VAR3"'

# Wait for completion
kubectl get pods env-test

# View output
kubectl logs env-test

# Expected Output:
# Var1: First, Var2: Second, Var3: Third

# Cleanup
kubectl delete pod env-test
```

#### Step 11: Test Without restartPolicy

**Create pod WITHOUT restartPolicy: Never (to see CrashLoopBackOff):**

```bash
vi crash-loop-test.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: crash-loop-test
spec:
  # restartPolicy: Always (default - commented out to show default behavior)
  containers:
  - name: test-container
    image: bash
    command: ["/bin/sh", "-c", "echo 'This will loop'; exit 0"]
```

```bash
# Apply
kubectl apply -f crash-loop-test.yaml

# Watch pod status
kubectl get pods crash-loop-test -w

# You'll see:
# NAME              READY   STATUS              RESTARTS   AGE
# crash-loop-test   0/1     ContainerCreating   0          2s
# crash-loop-test   0/1     Completed           0          5s
# crash-loop-test   0/1     Completed           1          6s
# crash-loop-test   0/1     CrashLoopBackOff    1          7s
# crash-loop-test   0/1     Completed           2          18s
# crash-loop-test   0/1     CrashLoopBackOff    2          19s
# ...continues with increasing delays

# Press Ctrl+C to stop watching

# View logs (will keep repeating)
kubectl logs crash-loop-test

# Cleanup
kubectl delete pod crash-loop-test
```

**What We Learned:**
- Without `restartPolicy: Never`, pod keeps restarting
- Each restart increases backoff delay
- This is why `Never` is critical for one-time commands

#### Step 12: Verify Pod Restart Count

```bash
# Check original pod restart count
kubectl get pod print-envars-greeting

# Expected Output:
# NAME                     READY   STATUS      RESTARTS   AGE
# print-envars-greeting    0/1     Completed   0          5m
#                                              ↑ Should be 0 (no restarts)
```

**If RESTARTS > 0:**
- Pod restarted due to failure
- Check logs: `kubectl logs print-envars-greeting --previous`
- Check events: `kubectl describe pod print-envars-greeting`

#### Step 13: Test Command Variations

**Test 1: Multiple echo statements**
```bash
kubectl run multi-echo --image=bash --restart=Never \
  --env="MSG=Hello" \
  -- /bin/sh -c 'echo "Message 1: $MSG"; echo "Message 2: World"'

kubectl logs multi-echo
# Expected Output:
# Message 1: Hello
# Message 2: World

kubectl delete pod multi-echo
```

**Test 2: Combine multiple variables**
```bash
kubectl run combine-vars --image=bash --restart=Never \
  --env="FIRST=John" \
  --env="LAST=Doe" \
  -- /bin/sh -c 'FULL="$FIRST $LAST"; echo "Full Name: $FULL"'

kubectl logs combine-vars
# Expected Output:
# Full Name: John Doe

kubectl delete pod combine-vars
```

**Test 3: Use variables in calculations**
```bash
kubectl run math-test --image=bash --restart=Never \
  --env="NUM1=10" \
  --env="NUM2=20" \
  -- /bin/sh -c 'SUM=$((NUM1 + NUM2)); echo "Sum: $SUM"'

kubectl logs math-test
# Expected Output:
# Sum: 30

kubectl delete pod math-test
```

### Phase 4: Advanced Environment Variable Patterns

#### Step 14: Environment Variables from ConfigMap

**Create ConfigMap:**
```bash
kubectl create configmap app-config \
  --from-literal=database.host=mysql.example.com \
  --from-literal=database.port=3306 \
  --from-literal=app.name=MyApp

# Verify
kubectl get configmap app-config -o yaml
```

**Create pod using ConfigMap:**
```bash
vi pod-with-configmap.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-configmap
spec:
  restartPolicy: Never
  containers:
  - name: app-container
    image: bash
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database.host
    - name: DB_PORT
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database.port
    - name: APP_NAME
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: app.name
    command: ["/bin/sh", "-c", 'echo "App: $APP_NAME, DB: $DB_HOST:$DB_PORT"']
```

```bash
# Apply
kubectl apply -f pod-with-configmap.yaml

# View output
kubectl logs pod-with-configmap

# Expected Output:
# App: MyApp, DB: mysql.example.com:3306

# Cleanup
kubectl delete pod pod-with-configmap
kubectl delete configmap app-config
```

#### Step 15: All Environment Variables from ConfigMap

**Create ConfigMap:**
```bash
kubectl create configmap full-config \
  --from-literal=VAR1=Value1 \
  --from-literal=VAR2=Value2 \
  --from-literal=VAR3=Value3
```

**Create pod that loads all ConfigMap values:**
```bash
vi pod-envfrom-configmap.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-envfrom-configmap
spec:
  restartPolicy: Never
  containers:
  - name: app-container
    image: bash
    envFrom:  # Load ALL keys from ConfigMap
    - configMapRef:
        name: full-config
    command: ["/bin/sh", "-c", 'echo "VAR1=$VAR1, VAR2=$VAR2, VAR3=$VAR3"']
```

```bash
# Apply
kubectl apply -f pod-envfrom-configmap.yaml

# View output
kubectl logs pod-envfrom-configmap

# Expected Output:
# VAR1=Value1, VAR2=Value2, VAR3=Value3

# Cleanup
kubectl delete pod pod-envfrom-configmap
kubectl delete configmap full-config
```

#### Step 16: Environment Variables from Pod Fields

**Pod that references its own metadata:**
```bash
vi pod-with-fieldref.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-fieldref
  labels:
    app: test
    tier: backend
spec:
  restartPolicy: Never
  containers:
  - name: app-container
    image: bash
    env:
    - name: MY_POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: MY_POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: MY_POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: MY_NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    command: ["/bin/sh", "-c", 'echo "Pod: $MY_POD_NAME in $MY_POD_NAMESPACE on $MY_NODE_NAME (IP: $MY_POD_IP)"']
```

```bash
# Apply
kubectl apply -f pod-with-fieldref.yaml

# View output
kubectl logs pod-with-fieldref

# Expected Output:
# Pod: pod-with-fieldref in default on node01 (IP: 10.244.1.5)

# Cleanup
kubectl delete pod pod-with-fieldref
```

## Complete Command Summary

### Original Task Commands
```bash
# Create pod YAML
vi print-envars-greeting.yaml

# Paste content:
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
spec:
  restartPolicy: Never
  containers:
  - name: print-env-container
    image: bash
    env:
    - name: GREETING
      value: "Welcome to"
    - name: COMPANY
      value: "DevOps"
    - name: GROUP
      value: "Datacenter"
    command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']

# Validate
kubectl apply -f print-envars-greeting.yaml --dry-run=client

# Apply
kubectl apply -f print-envars-greeting.yaml

# Verify pod status
kubectl get pods
kubectl get pod print-envars-greeting

# View logs (output)
kubectl logs print-envars-greeting

# Expected Output: "Welcome to DevOps Datacenter"

# Follow logs (if still running)
kubectl logs -f print-envars-greeting

# Describe pod
kubectl describe pod print-envars-greeting
```

### Quick One-Liner (Alternative Method)
```bash
# Create pod using kubectl run (without YAML file)
kubectl run print-envars-greeting \
  --image=bash \
  --restart=Never \
  --env="GREETING=Welcome to" \
  --env="COMPANY=DevOps" \
  --env="GROUP=Datacenter" \
  -- /bin/sh -c 'echo "$(GREETING) $(COMPANY) $(GROUP)"'

# View logs
kubectl logs print-envars-greeting

# Expected Output: "Welcome to DevOps Datacenter"
```

### Verification Commands
```bash
# Check pod status
kubectl get pods print-envars-greeting

# View pod YAML
kubectl get pod print-envars-greeting -o yaml

# View environment variables
kubectl get pod print-envars-greeting -o yaml | grep -A 10 "env:"

# View command
kubectl get pod print-envars-greeting -o yaml | grep -A 5 "command:"

# View logs with timestamps
kubectl logs print-envars-greeting --timestamps

# Check restart policy
kubectl get pod print-envars-greeting -o jsonpath='{.spec.restartPolicy}'

# Check restart count
kubectl get pod print-envars-greeting -o jsonpath='{.status.containerStatuses[0].restartCount}'

# View pod events
kubectl get events --field-selector involvedObject.name=print-envars-greeting
```

### Cleanup Commands
```bash
# Delete pod
kubectl delete pod print-envars-greeting

# Delete pod using YAML file
kubectl delete -f print-envars-greeting.yaml

# Force delete (if stuck)
kubectl delete pod print-envars-greeting --force --grace-period=0

# Verify deletion
kubectl get pods

# Delete YAML file (optional)
rm print-envars-greeting.yaml
```

## Troubleshooting Common Issues

### Issue 1: Pod Stuck in CrashLoopBackOff

**Symptoms:**
```bash
kubectl get pods
# NAME                     READY   STATUS             RESTARTS   AGE
# print-envars-greeting    0/1     CrashLoopBackOff   5          3m
```

**Diagnosis:**
```bash
# Check if restartPolicy is set correctly
kubectl get pod print-envars-greeting -o jsonpath='{.spec.restartPolicy}'

# Should output: Never
# If output is: Always or OnFailure → That's the problem!
```

**Cause:**
```yaml
# Missing or wrong restartPolicy
spec:
  restartPolicy: Always  # ❌ Wrong! Command exits immediately, then restarts infinitely
```

**Fix:**
```yaml
spec:
  restartPolicy: Never   # ✅ Correct! Command runs once and stops
```

**Solution:**
1. Delete the pod: `kubectl delete pod print-envars-greeting`
2. Edit YAML to add `restartPolicy: Never`
3. Reapply: `kubectl apply -f print-envars-greeting.yaml`

### Issue 2: Environment Variables Not Substituted

**Symptoms:**
```bash
kubectl logs print-envars-greeting
# Output: $(GREETING) $(COMPANY) $(GROUP)
# ❌ Variables not replaced, shows literal $(VAR) syntax
```

**Cause 1: Using Double Quotes in YAML**
```yaml
❌ WRONG:
command: ["/bin/sh", "-c", "echo \"$(GREETING) $(COMPANY) $(GROUP)\""]
# Kubernetes might try to substitute variables (but they don't exist at YAML parsing time)

✅ CORRECT:
command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
# Single quotes in YAML, double quotes inside shell command
```

**Cause 2: Wrong Variable Syntax**
```yaml
❌ WRONG:
command: ["/bin/sh", "-c", 'echo "${GREETING} ${COMPANY} ${GROUP}"']
# ${VAR} syntax might not work depending on shell

✅ CORRECT:
command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
# $(VAR) syntax is more reliable
```

**Cause 3: Variables Not Defined**
```bash
# Check if environment variables are actually set
kubectl get pod print-envars-greeting -o yaml | grep -A 10 "env:"

# If empty or wrong names → Fix env section
```

### Issue 3: Pod Shows "ImagePullBackOff"

**Symptoms:**
```bash
kubectl get pods
# NAME                     READY   STATUS             RESTARTS   AGE
# print-envars-greeting    0/1     ImagePullBackOff   0          2m
```

**Diagnosis:**
```bash
kubectl describe pod print-envars-greeting

# Look for:
# Events:
#   Failed to pull image "bash": rpc error: code = Unknown desc = Error response from daemon: manifest for bash:latest not found
```

**Common Causes:**

**1. Wrong Image Name:**
```yaml
❌ WRONG:
image: bahs  # Typo!

✅ CORRECT:
image: bash
```

**2. Image Doesn't Exist:**
```yaml
# Bash image should exist on Docker Hub
# Verify: https://hub.docker.com/_/bash

# If using private registry, ensure:
# - Image name includes registry
# - imagePullSecrets configured
```

**3. Network Issues:**
- Node can't reach Docker Hub
- Check node internet connectivity
- Check proxy settings

**Fix:**
1. Correct image name in YAML
2. Reapply: `kubectl apply -f print-envars-greeting.yaml`

### Issue 4: Pod Status Stuck in "Pending"

**Symptoms:**
```bash
kubectl get pods
# NAME                     READY   STATUS    RESTARTS   AGE
# print-envars-greeting    0/1     Pending   0          5m
```

**Diagnosis:**
```bash
kubectl describe pod print-envars-greeting

# Look at Events section for scheduling issues
```

**Common Causes:**

**1. No Available Nodes:**
```
Events:
  Warning  FailedScheduling  5m  default-scheduler  0/0 nodes are available
```

**Fix:** Ensure cluster has at least one ready node

**2. Insufficient Resources:**
```
Events:
  Warning  FailedScheduling  5m  default-scheduler  0/1 nodes available: insufficient cpu
```

**Fix:** 
- Remove resource-heavy pods
- Add more nodes
- Reduce resource requests in pod spec

**3. Node Taints:**
```
Events:
  Warning  FailedScheduling  5m  default-scheduler  0/1 nodes available: node(s) had untolerated taint
```

**Fix:** Add tolerations or remove taints from nodes

### Issue 5: Wrong Output in Logs

**Symptoms:**
```bash
kubectl logs print-envars-greeting
# Output: Welcome to  
# ❌ Missing COMPANY and GROUP values
```

**Diagnosis:**
```bash
# Check environment variables are defined correctly
kubectl get pod print-envars-greeting -o yaml | grep -A 15 "env:"

# Verify values
kubectl get pod print-envars-greeting -o jsonpath='{.spec.containers[0].env[*].name}'
# Should show: GREETING COMPANY GROUP

kubectl get pod print-envars-greeting -o jsonpath='{.spec.containers[0].env[*].value}'
# Should show: Welcome to DevOps Datacenter
```

**Common Causes:**

**1. Missing Environment Variables:**
```yaml
❌ INCOMPLETE:
env:
- name: GREETING
  value: "Welcome to"
# Missing COMPANY and GROUP!

✅ COMPLETE:
env:
- name: GREETING
  value: "Welcome to"
- name: COMPANY
  value: "DevOps"
- name: GROUP
  value: "Datacenter"
```

**2. Empty Values:**
```yaml
❌ WRONG:
- name: COMPANY
  value: ""  # Empty string!

✅ CORRECT:
- name: COMPANY
  value: "DevOps"
```

**3. Wrong Indentation:**
```yaml
❌ WRONG:
containers:
- name: print-env-container
  image: bash
  env:
  - name: GREETING
    value: "Welcome to"
  - name: COMPANY
  value: "DevOps"  # Wrong indentation!

✅ CORRECT:
containers:
- name: print-env-container
  image: bash
  env:
  - name: GREETING
    value: "Welcome to"
  - name: COMPANY
    value: "DevOps"  # Aligned with GREETING
```

### Issue 6: Command Syntax Errors

**Symptoms:**
```bash
kubectl logs print-envars-greeting
# Shows error like:
# /bin/sh: syntax error: unterminated quoted string
```

**Diagnosis:**
Check command quotes and escaping

**Common Causes:**

**1. Mismatched Quotes:**
```yaml
❌ WRONG:
command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)']
# Missing closing double quote!

✅ CORRECT:
command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
```

**2. Wrong Quote Type:**
```yaml
❌ PROBLEMATIC:
command: ["/bin/sh", "-c", "echo \"$(GREETING) $(COMPANY) $(GROUP)\""]
# Escaped quotes can cause issues

✅ BETTER:
command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
# Single quotes in YAML, double quotes in shell command
```

**3. Special Characters Not Escaped:**
```yaml
# If your values have special characters:
- name: MESSAGE
  value: "Hello! It's great"  # Apostrophe can cause issues

# Wrap in quotes properly
```

## Environment Variable Best Practices

### 1. Naming Conventions

**✅ Good Names:**
```yaml
env:
- name: DATABASE_HOST           # Clear, descriptive
- name: API_KEY                 # Uppercase with underscores
- name: MAX_CONNECTIONS         # Descriptive
- name: LOG_LEVEL               # Standard format
```

**❌ Bad Names:**
```yaml
env:
- name: db                      # Too short, unclear
- name: Key                     # Inconsistent case
- name: x                       # Not descriptive
- name: the-database-host       # Dashes (use underscores)
```

**Convention:**
- Use UPPERCASE
- Separate words with underscores
- Be descriptive
- Follow language/framework conventions

### 2. Value Quoting

**When to Use Quotes:**

```yaml
✅ ALWAYS quote strings with special characters:
- name: MESSAGE
  value: "Hello, World!"        # Has comma and space

- name: SQL_QUERY
  value: "SELECT * FROM users"  # Has spaces

- name: JSON_DATA
  value: '{"key": "value"}'     # Has special chars

✅ Quote numbers if you want them as strings:
- name: PORT
  value: "8080"                 # String

✅ Quote to preserve leading/trailing spaces:
- name: PREFIX
  value: "  INDENT  "

⚠️ Can omit quotes for simple values:
- name: ENVIRONMENT
  value: production             # Simple word (but quoting is safer)
```

### 3. Sensitive Data (Secrets)

**❌ DON'T store secrets in env directly:**
```yaml
env:
- name: DATABASE_PASSWORD
  value: "super-secret-password"  # ❌ Visible in pod spec!
```

**✅ DO use Kubernetes Secrets:**
```yaml
# Create secret first:
# kubectl create secret generic db-secret --from-literal=password=super-secret-password

env:
- name: DATABASE_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password             # ✅ Not visible in pod spec
```

### 4. ConfigMap vs Direct Values

**Use Direct Values For:**
- Simple, static configuration
- Non-sensitive data
- Testing/development

```yaml
env:
- name: APP_MODE
  value: "debug"
- name: LOG_LEVEL
  value: "info"
```

**Use ConfigMap For:**
- Shared configuration across pods
- Configuration that might change
- Many environment variables

```yaml
envFrom:
- configMapRef:
    name: app-config  # Loads all keys from ConfigMap
```

### 5. Environment Variable Precedence

**If same variable defined multiple times:**

```yaml
containers:
- name: app
  image: myapp
  env:
  - name: DEBUG
    value: "false"              # Defined directly
  envFrom:
  - configMapRef:
      name: config              # ConfigMap also has DEBUG=true

# Result: Direct env (false) takes precedence over envFrom
```

**Order:**
1. Direct `env` values (highest priority)
2. `envFrom` ConfigMap
3. `envFrom` Secret
4. Image defaults (lowest priority)

### 6. Variable Substitution Patterns

**Shell Substitution (What We Used):**
```yaml
command: ["/bin/sh", "-c", 'echo "Value: $(MY_VAR)"']
# Shell interprets $(MY_VAR) at runtime
```

**Alternative Syntax:**
```yaml
command: ["/bin/sh", "-c", 'echo "Value: ${MY_VAR}"']
# Also works, slightly different shell behavior
```

**With Default Values:**
```yaml
command: ["/bin/sh", "-c", 'echo "Value: ${MY_VAR:-default}"']
# If MY_VAR empty/unset, use "default"
```

**Kubernetes Variable Substitution (Limited):**
```yaml
# Only works in specific fields like args with $(VAR_NAME)
args:
- "$(MY_POD_NAME)"  # Kubernetes substitutes before passing to container
```

### 7. Multi-line Values

**For long values:**
```yaml
env:
- name: CONFIG_JSON
  value: |
    {
      "database": {
        "host": "mysql.example.com",
        "port": 3306
      },
      "cache": {
        "enabled": true
      }
    }
```

**Or use ConfigMap for complex config:**
```bash
kubectl create configmap app-config --from-file=config.json
```

### 8. Environment Variables for Different Environments

**Pattern: Use same image, different configs**

**Development:**
```yaml
env:
- name: ENVIRONMENT
  value: "development"
- name: DEBUG
  value: "true"
- name: DATABASE_HOST
  value: "dev-db.local"
```

**Production:**
```yaml
env:
- name: ENVIRONMENT
  value: "production"
- name: DEBUG
  value: "false"
- name: DATABASE_HOST
  value: "prod-db.example.com"
```

**Better: Use ConfigMaps per environment**
```bash
# Development
kubectl create configmap app-config \
  --from-literal=ENVIRONMENT=development \
  --from-literal=DEBUG=true \
  --namespace=dev

# Production
kubectl create configmap app-config \
  --from-literal=ENVIRONMENT=production \
  --from-literal=DEBUG=false \
  --namespace=prod
```

### 9. Debugging Environment Variables

**View all env vars in running pod:**
```bash
kubectl exec print-envars-greeting -- env
# Shows all environment variables

kubectl exec print-envars-greeting -- printenv
# Alternative command
```

**View specific variable:**
```bash
kubectl exec print-envars-greeting -- printenv GREETING
# Shows: Welcome to
```

**View in pod spec:**
```bash
kubectl get pod print-envars-greeting -o json | jq '.spec.containers[0].env'
```

### 10. Testing Environment Variables Locally

**Before deploying to Kubernetes:**
```bash
# Test with Docker
docker run --rm \
  -e GREETING="Welcome to" \
  -e COMPANY="DevOps" \
  -e GROUP="Datacenter" \
  bash \
  /bin/sh -c 'echo "$(GREETING) $(COMPANY) $(GROUP)"'

# Expected Output:
# Welcome to DevOps Datacenter
```

## Completion Checklist

Verify your setup with these commands:

```bash
# 1. Pod exists with correct name
kubectl get pod print-envars-greeting
# NAME: print-envars-greeting ✅

# 2. Pod status is Completed (not Running or CrashLoopBackOff)
kubectl get pod print-envars-greeting -o jsonpath='{.status.phase}'
# Output: Succeeded ✅

# 3. Restart count is 0 (no restarts)
kubectl get pod print-envars-greeting -o jsonpath='{.status.containerStatuses[0].restartCount}'
# Output: 0 ✅

# 4. Container name is correct
kubectl get pod print-envars-greeting -o jsonpath='{.spec.containers[0].name}'
# Output: print-env-container ✅

# 5. Image is bash
kubectl get pod print-envars-greeting -o jsonpath='{.spec.containers[0].image}'
# Output: bash ✅

# 6. Restart policy is Never
kubectl get pod print-envars-greeting -o jsonpath='{.spec.restartPolicy}'
# Output: Never ✅

# 7. All three environment variables are set
kubectl get pod print-envars-greeting -o jsonpath='{.spec.containers[0].env[*].name}'
# Output: GREETING COMPANY GROUP ✅

# 8. GREETING value is correct
kubectl get pod print-envars-greeting -o jsonpath='{.spec.containers[0].env[?(@.name=="GREETING")].value}'
# Output: Welcome to ✅

# 9. COMPANY value is correct
kubectl get pod print-envars-greeting -o jsonpath='{.spec.containers[0].env[?(@.name=="COMPANY")].value}'
# Output: DevOps ✅

# 10. GROUP value is correct
kubectl get pod print-envars-greeting -o jsonpath='{.spec.containers[0].env[?(@.name=="GROUP")].value}'
# Output: Datacenter ✅

# 11. Command is set correctly
kubectl get pod print-envars-greeting -o jsonpath='{.spec.containers[0].command}'
# Output: ["/bin/sh","-c","echo \"$(GREETING) $(COMPANY) $(GROUP)\""] ✅

# 12. Logs show correct output
kubectl logs print-envars-greeting
# Output: Welcome to DevOps Datacenter ✅

# 13. Exit code is 0 (success)
kubectl get pod print-envars-greeting -o jsonpath='{.status.containerStatuses[0].state.terminated.exitCode}'
# Output: 0 ✅

# 14. No error messages in events
kubectl get events --field-selector involvedObject.name=print-envars-greeting,type=Warning
# Output: No resources found ✅
```

**All checks passing = Task complete! ✅**

## Summary

### What We Accomplished
1. ✅ Created pod with environment variables
2. ✅ Used bash image for shell commands
3. ✅ Set three environment variables (GREETING, COMPANY, GROUP)
4. ✅ Executed command with variable substitution
5. ✅ Set restartPolicy to Never (prevented CrashLoopBackOff)
6. ✅ Verified output using kubectl logs
7. ✅ Demonstrated one-time task pattern
8. ✅ Learned environment variable best practices

### Key Concepts Learned

**Environment Variables:**
- Externalize configuration from code
- 12-Factor App principle: "Config in environment"
- Four ways to set: direct, ConfigMap, Secret, fieldRef
- Variable substitution with $(VAR) syntax
- Shell interprets variables at runtime

**Restart Policies:**
- **Always:** Restart forever (services)
- **OnFailure:** Restart on failure (batch jobs)
- **Never:** No restart (one-time tasks) ← Used in this task
- Prevents CrashLoopBackOff for short-lived pods

**Command Execution:**
- `command` overrides ENTRYPOINT
- `args` overrides CMD
- Inline array format: `["/bin/sh", "-c", "echo hello"]`
- Shell substitution for variables

**Pod Lifecycle:**
- ContainerCreating → Running → Completed
- Exit code 0 = Success
- Logs available after completion
- restartPolicy determines post-exit behavior

### Critical Commands
```bash
# Create pod with environment variables (YAML)
kubectl apply -f print-envars-greeting.yaml

# Create pod with environment variables (one-liner)
kubectl run print-envars-greeting \
  --image=bash \
  --restart=Never \
  --env="GREETING=Welcome to" \
  --env="COMPANY=DevOps" \
  --env="GROUP=Datacenter" \
  -- /bin/sh -c 'echo "$(GREETING) $(COMPANY) $(GROUP)"'

# View logs (output)
kubectl logs print-envars-greeting
# Expected: Welcome to DevOps Datacenter

# Follow logs
kubectl logs -f print-envars-greeting

# Verify environment variables
kubectl get pod print-envars-greeting -o yaml | grep -A 10 "env:"

# Check pod status
kubectl get pod print-envars-greeting

# Describe pod
kubectl describe pod print-envars-greeting

# Delete pod
kubectl delete pod print-envars-greeting
```

### Configuration Pattern

```yaml
Complete Pod Configuration:
├── apiVersion: v1
├── kind: Pod
├── metadata:
│   └── name: print-envars-greeting
└── spec:
    ├── restartPolicy: Never          # One-time execution
    └── containers:
        ├── name: print-env-container
        ├── image: bash               # Shell for commands
        ├── env:                      # Environment variables
        │   ├── GREETING="Welcome to"
        │   ├── COMPANY="DevOps"
        │   └── GROUP="Datacenter"
        └── command:                  # Shell command
            └── /bin/sh -c 'echo "$(GREETING) $(COMPANY) $(GROUP)"'
```

### Real-World Applications

**1. Application Configuration:**
```yaml
env:
- name: DATABASE_URL
  value: "postgresql://db:5432/myapp"
- name: REDIS_HOST
  value: "redis-service"
- name: LOG_LEVEL
  value: "info"
```

**2. Feature Flags:**
```yaml
env:
- name: FEATURE_NEW_UI
  value: "true"
- name: FEATURE_BETA_API
  value: "false"
```

**3. Service Discovery:**
```yaml
env:
- name: API_ENDPOINT
  value: "http://api-service:8080"
- name: CACHE_SERVERS
  value: "cache1:11211,cache2:11211"
```

**4. Runtime Behavior:**
```yaml
env:
- name: DEBUG_MODE
  value: "false"
- name: MAX_WORKERS
  value: "4"
- name: TIMEOUT_SECONDS
  value: "30"
```

### Best Practices Applied
1. ✅ Externalized configuration (environment variables)
2. ✅ Used appropriate restart policy (Never)
3. ✅ Clear, descriptive variable names (GREETING, COMPANY, GROUP)
4. ✅ Proper command syntax with quotes
5. ✅ Verified output with kubectl logs
6. ✅ Used shell substitution correctly $(VAR)
7. ✅ Documented expected output
8. ✅ Provided troubleshooting guidance

### What's Next?

**Day 58+:** Continue with Kubernetes configuration:
- **ConfigMaps:** Manage configuration files
- **Secrets:** Store sensitive data (passwords, tokens)
- **Volumes:** Mount ConfigMaps and Secrets as files
- **Init Containers:** Run setup tasks before main container
- **Resource Limits:** CPU and memory constraints
- **Probes:** Health checks (liveness, readiness, startup)

**Learning Path:**
```
Day 57: Environment Variables ← YOU ARE HERE
    ↓
Day 58+: ConfigMaps (configuration files)
    ↓
Day 59+: Secrets (sensitive data)
    ↓
Day 60+: Volumes (persistent storage)
    ↓
Day 61+: Init Containers (initialization)
    ↓
Day 62+: Resource Management
```

### Environment Variable Progression

**Today (Day 57):**
```yaml
env:  # Direct values
- name: VAR
  value: "value"
```

**Next Steps:**
```yaml
# ConfigMap reference
- name: VAR
  valueFrom:
    configMapKeyRef:
      name: config
      key: var

# Secret reference
- name: PASSWORD
  valueFrom:
    secretKeyRef:
      name: secret
      key: password

# Field reference
- name: POD_IP
  valueFrom:
    fieldRef:
      fieldPath: status.podIP
```

🎉 **Day 57 Complete!** You've mastered Kubernetes environment variables and pod configuration with custom commands—essential skills for configuring applications in Kubernetes!
