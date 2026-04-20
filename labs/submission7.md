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


## Task 2 — GitOps Health Monitoring

### 2.1 Create Health Check Script

Commands

```bash
cd ~/gitops-lab
rm -f health.log
cat > healthcheck.sh <<'EOF'
#!/bin/bash
# healthcheck.sh - Monitor GitOps sync health

DESIRED_MD5=$(md5sum desired-state.txt | awk '{print $1}')
CURRENT_MD5=$(md5sum current-state.txt | awk '{print $1}')

if [ "$DESIRED_MD5" != "$CURRENT_MD5" ]; then
    echo "$(date) - ❌ CRITICAL: State mismatch detected!" | tee -a health.log
    echo "  Desired MD5: $DESIRED_MD5" | tee -a health.log
    echo "  Current MD5: $CURRENT_MD5" | tee -a health.log
else
    echo "$(date) - ✅ OK: States synchronized" | tee -a health.log
fi
EOF
chmod +x healthcheck.sh
echo "===== healthcheck.sh ====="
cat healthcheck.sh
````

Output

```text
===== healthcheck.sh =====
#!/bin/bash
# healthcheck.sh - Monitor GitOps sync health

DESIRED_MD5=$(md5sum desired-state.txt | awk '{print $1}')
CURRENT_MD5=$(md5sum current-state.txt | awk '{print $1}')

if [ "$DESIRED_MD5" != "$CURRENT_MD5" ]; then
    echo "$(date) - ❌ CRITICAL: State mismatch detected!" | tee -a health.log
    echo "  Desired MD5: $DESIRED_MD5" | tee -a health.log
    echo "  Current MD5: $CURRENT_MD5" | tee -a health.log
else
    echo "$(date) - ✅ OK: States synchronized" | tee -a health.log
fi
```

### 2.2 Test Health Monitoring

Healthy state check

Commands

```bash
./healthcheck.sh
cat health.log
```

Output

```text
Thu Mar 19 16:33:20 UTC 2026 - ✅ OK: States synchronized

Thu Mar 19 16:33:20 UTC 2026 - ✅ OK: States synchronized
```

Drift simulation

Commands

```bash
echo "unapproved-change: true" >> current-state.txt
cat current-state.txt
./healthcheck.sh
cat health.log
```

Output

```text
version: 1.0
app: myapp
replicas: 3
unapproved-change: true

Thu Mar 19 16:33:20 UTC 2026 - ❌ CRITICAL: State mismatch detected!
  Desired MD5: a15a1a4f965ecd8f9e23a33a6b543155
  Current MD5: 48168ff3ab5ffc0214e81c7e2ee356f5

Thu Mar 19 16:33:20 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:20 UTC 2026 - ❌ CRITICAL: State mismatch detected!
  Desired MD5: a15a1a4f965ecd8f9e23a33a6b543155
  Current MD5: 48168ff3ab5ffc0214e81c7e2ee356f5
```

Fix drift and verify

Commands

```bash
./reconcile.sh
./healthcheck.sh
cat health.log
```

Output

```text
Thu Mar 19 16:33:21 UTC 2026 - ⚠️  DRIFT DETECTED!
Reconciling current state with desired state...
Thu Mar 19 16:33:21 UTC 2026 - ✅ Reconciliation complete
Thu Mar 19 16:33:21 UTC 2026 - ✅ OK: States synchronized

Thu Mar 19 16:33:20 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:20 UTC 2026 - ❌ CRITICAL: State mismatch detected!
  Desired MD5: a15a1a4f965ecd8f9e23a33a6b543155
  Current MD5: 48168ff3ab5ffc0214e81c7e2ee356f5
Thu Mar 19 16:33:21 UTC 2026 - ✅ OK: States synchronized
```

### 2.3 Continuous Health Monitoring

Commands

```bash
cat > monitor.sh <<'EOF'
#!/bin/bash
# monitor.sh - Combined reconciliation and health monitoring

echo "Starting GitOps monitoring..."
for i in {1..10}; do
    echo
    echo "--- Check #$i ---"
    ./healthcheck.sh
    ./reconcile.sh
    sleep 3
done
EOF
chmod +x monitor.sh
echo "===== monitor.sh ====="
cat monitor.sh
./monitor.sh
cat health.log
```

monitor.sh contents

```text
#!/bin/bash
# monitor.sh - Combined reconciliation and health monitoring

echo "Starting GitOps monitoring..."
for i in {1..10}; do
    echo
    echo "--- Check #$i ---"
    ./healthcheck.sh
    ./reconcile.sh
    sleep 3
done
```

monitor.sh output

```text
Starting GitOps monitoring...

--- Check #1 ---
Thu Mar 19 16:33:21 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:21 UTC 2026 - ✅ States synchronized

--- Check #2 ---
Thu Mar 19 16:33:24 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:24 UTC 2026 - ✅ States synchronized

--- Check #3 ---
Thu Mar 19 16:33:27 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:27 UTC 2026 - ✅ States synchronized

--- Check #4 ---
Thu Mar 19 16:33:30 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:30 UTC 2026 - ✅ States synchronized

--- Check #5 ---
Thu Mar 19 16:33:33 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:33 UTC 2026 - ✅ States synchronized

--- Check #6 ---
Thu Mar 19 16:33:36 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:36 UTC 2026 - ✅ States synchronized

--- Check #7 ---
Thu Mar 19 16:33:39 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:39 UTC 2026 - ✅ States synchronized

--- Check #8 ---
Thu Mar 19 16:33:42 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:42 UTC 2026 - ✅ States synchronized

--- Check #9 ---
Thu Mar 19 16:33:45 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:45 UTC 2026 - ✅ States synchronized

--- Check #10 ---
Thu Mar 19 16:33:48 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:48 UTC 2026 - ✅ States synchronized
```

Complete health.log

```text
Thu Mar 19 16:33:20 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:20 UTC 2026 - ❌ CRITICAL: State mismatch detected!
  Desired MD5: a15a1a4f965ecd8f9e23a33a6b543155
  Current MD5: 48168ff3ab5ffc0214e81c7e2ee356f5
Thu Mar 19 16:33:21 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:21 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:24 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:27 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:30 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:33 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:36 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:39 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:42 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:45 UTC 2026 - ✅ OK: States synchronized
Thu Mar 19 16:33:48 UTC 2026 - ✅ OK: States synchronized
```

### Analysis

MD5 checksums help detect configuration changes because even a small change in the file produces a different hash value. Instead of comparing the whole file line by line, the script compares two compact fingerprints and quickly determines whether the desired state and current state are identical or not.

### Comparison

This is similar to GitOps tools like ArgoCD and their Sync Status. ArgoCD compares the desired state stored in Git with the live state in the cluster and marks the application as synced or out of sync. In this lab, the MD5 comparison plays the same role in a simplified form: matching hashes mean synced state, and different hashes mean drift.

## Report

In this lab, I simulated the basic GitOps workflow with simple Linux tools. First, I created a desired state file and a current state file. After that, I wrote a reconciliation script that compares both files and restores the current state if drift is detected.

The results showed that the reconciliation loop works as expected. When I changed `current-state.txt` manually, the script detected the difference and replaced the drifted state with the correct state from `desired-state.txt`. This demonstrates the main GitOps idea: the declared state is the source of truth, and the system should always return to it.

The continuous reconciliation test also showed self-healing behavior. After I introduced an extra change into the current state, the loop detected it automatically and fixed it during the next check. This means that even if someone changes the live state manually, the system can recover without manual correction.

In the second part of the lab, I added health monitoring with MD5 checksums. This made it possible to detect whether the desired state and the current state were identical. When both files matched, the script reported `OK`. When the state was changed, it reported `CRITICAL` and wrote the mismatch to `health.log`.

The monitoring script combined health checks and reconciliation into one loop. This showed how GitOps systems can both detect problems and recover from them continuously. In a real environment, tools like ArgoCD and Flux use the same general idea, but with Kubernetes resources instead of simple text files.

Overall, this lab helped me understand the core GitOps principles in a practical way. Declarative configuration is useful because the expected state is written clearly and can be versioned in Git. Reconciliation helps prevent configuration drift, and health checks make the sync status visible. Together, these mechanisms create a simple self-healing system.