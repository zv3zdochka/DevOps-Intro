# Lab 3 — CI/CD with GitHub Actions

## Task 1 — First GitHub Actions Workflow

**Successful run:** https://github.com/zv3zdochka/DevOps-Intro/actions/runs/22070050625

### Evidence (screenshots)

![Actions runs list](screenshots/lab_3/lab_3_0.png)
![Run summary / jobs](screenshots/lab_3/lab_3_1.png)
![Job logs: Basic info](screenshots/lab_3/lab_3_2.png)
![Job logs: Dependent job](screenshots/lab_3/lab_3_3.png)

### What I did (Quickstart implementation)

- Created a workflow YAML file in `.github/workflows/` (`lab3.yml`).
- Configured the workflow to run on the `push` event.
- Defined two jobs:
  - `info` — prints workflow context and basic OS/tooling information.
  - `second-job` — runs only after `info` completes successfully (job dependency).

### Key concepts

- **Workflow**: an automation described in a YAML file stored under `.github/workflows/`.
- **Trigger / Event**: the condition that starts a workflow run (here: `on: push`).
- **Runner**: the execution environment (a GitHub-hosted VM) where jobs run (`ubuntu-latest`).
- **Job**: a set of steps executed on the same runner (here: `info`, `second-job`).
- **Step**: an individual action (`uses`) or shell command (`run`) executed inside a job.
- **Job dependency**: `needs: info` ensures `second-job` starts only after `info` succeeds.

### What triggered the run

The workflow run started automatically because I pushed a commit to branch `feature/lab3`, and the workflow is configured with the `push` trigger.

### Execution analysis

After the `push` event, GitHub Actions matched the workflow by `on: push`, provisioned a GitHub-hosted runner (`ubuntu-latest`) for the `info` job, and executed its steps sequentially (checkout → printing context → OS basics).  
Once `info` finished successfully, the dependency condition `needs: info` was satisfied, so `second-job` started and executed its step, confirming the job order via logs.
