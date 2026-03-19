## Task 1 — Git State Reconciliation

### 1.1 Setup Desired State Configuration

Commands

```bash
mkdir -p gitops-lab
cd gitops-lab
printf "version: 1.0\napp: myapp\nreplicas: 3\n" > desired-state.txt
cp desired-state.txt current-state.txt
echo "Initial state synchronized"
echo "===== desired-state.txt ====="
cat desired-state.txt
echo
echo "===== current-state.txt ====="
cat current-state.txt
````

Output

```text
Initial state synchronized
===== desired-state.txt =====
version: 1.0
app: myapp
replicas: 3

===== current-state.txt =====
version: 1.0
app: myapp
replicas: 3
```

Initial desired-state.txt

```text
version: 1.0
app: myapp
replicas: 3
```

Initial current-state.txt

```text
version: 1.0
app: myapp
replicas: 3
```

### 1.2 Create Reconciliation Loop

Commands

```bash
cat > reconcile.sh <<'EOF'
#!/bin/bash
# reconcile.sh - GitOps reconciliation loop

DESIRED=$(cat desired-state.txt)
CURRENT=$(cat current-state.txt)

if [ "$DESIRED" != "$CURRENT" ]; then
    echo "$(date) - ⚠️  DRIFT DETECTED!"
    echo "Reconciling current state with desired state..."
    cp desired-state.txt current-state.txt
    echo "$(date) - ✅ Reconciliation complete"
else
    echo "$(date) - ✅ States synchronized"
fi
EOF
chmod +x reconcile.sh
echo "===== reconcile.sh ====="
cat reconcile.sh
```

Output

```text
===== reconcile.sh =====
#!/bin/bash
# reconcile.sh - GitOps reconciliation loop

DESIRED=$(cat desired-state.txt)
CURRENT=$(cat current-state.txt)

if [ "$DESIRED" != "$CURRENT" ]; then
    echo "$(date) - ⚠️  DRIFT DETECTED!"
    echo "Reconciling current state with desired state..."
    cp desired-state.txt current-state.txt
    echo "$(date) - ✅ Reconciliation complete"
else
    echo "$(date) - ✅ States synchronized"
fi
```

### 1.3 Test Manual Drift Detection

Commands

```bash
printf "version: 2.0\napp: myapp\nreplicas: 5\n" > current-state.txt
echo "===== current-state.txt BEFORE reconciliation ====="
cat current-state.txt
echo
echo "===== manual reconciliation output ====="
./reconcile.sh
echo
echo "===== diff desired vs current AFTER reconciliation ====="
diff desired-state.txt current-state.txt
echo "diff exit code: $?"
echo
echo "===== current-state.txt AFTER reconciliation ====="
cat current-state.txt
```

Output

```text
===== current-state.txt BEFORE reconciliation =====
version: 2.0
app: myapp
replicas: 5

===== manual reconciliation output =====
Thu Mar 19 16:21:02 UTC 2026 - ⚠️  DRIFT DETECTED!
Reconciling current state with desired state...
Thu Mar 19 16:21:02 UTC 2026 - ✅ Reconciliation complete

===== diff desired vs current AFTER reconciliation =====
diff exit code: 0

===== current-state.txt AFTER reconciliation =====
version: 1.0
app: myapp
replicas: 3
```

Synchronized state after reconciliation

```text
version: 1.0
app: myapp
replicas: 3
```

### 1.4 Automated Continuous Reconciliation

Commands

```bash
cd ~/gitops-lab
watch -n 5 ./reconcile.sh
```

In another terminal

```bash
cd ~/gitops-lab
echo "replicas: 10" >> current-state.txt
cat current-state.txt
```

Continuous reconciliation output

```text
Every 5.0s: ./reconcile.sh                                                                gnow: Thu Mar 19 16:28:37 2026

Thu Mar 19 16:28:37 UTC 2026 - ⚠️  DRIFT DETECTED!
Reconciling current state with desired state...
Thu Mar 19 16:28:37 UTC 2026 - ✅ Reconciliation complete
```

State after auto-healing

```text
version: 1.0
app: myapp
replicas: 3
```

### Analysis

The reconciliation loop compares the desired state from Git with the current state in the system. If the files are different, it detects drift and restores the current state from the desired state. This prevents configuration drift because any manual or accidental change is automatically overwritten by the declared source of truth.

### Reflection

Declarative configuration is better than imperative commands in production because the expected final state is written explicitly and stored in Git. It is easier to review, version, audit, and restore. It also reduces human error because the system does not depend on manually repeating commands in the correct order.



