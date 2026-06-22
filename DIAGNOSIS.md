# CI Failure Diagnosis

## Failure 1 – Dependency Installation

**Step:** Install dependencies

**Error from log:**

```text
run: npm install
```

**Cause:**
The workflow uses `npm install`, which can install different package versions over time and lead to non-reproducible builds. CI pipelines should use `npm ci` to ensure that dependencies are installed exactly as specified in `package-lock.json`.

**Fix Applied:**
Replaced:

```yaml
run: npm install
```

with:

```yaml
run: npm ci
```

---

## Failure 2 – Workflow Sequencing

**Step:** Test job

**Error from log:**

```text
test job starts independently without waiting for install
```

**Cause:**
The `test` job did not contain a `needs: install` dependency. As a result, both jobs ran in parallel, causing the test job to start before the installation job completed.

**Fix Applied:**
Added:

```yaml
needs: install
```

to the test job so that it runs only after the install job succeeds.

---

## Failure 3 – Missing Setup in Test Job

**Step:** Run tests

**Error from log:**

```text
npm test
```

**Cause:**
Each GitHub Actions job runs on a fresh virtual machine. The test job had no `actions/checkout`, no Node.js setup, and no dependency installation. Therefore, the repository contents and `node_modules` were unavailable when `npm test` executed.

**Fix Applied:**
Added the following steps to the test job:

```yaml
- uses: actions/checkout@v4

- uses: actions/setup-node@v4
  with:
    node-version: '18'

- name: Install dependencies
  run: npm ci
```

This ensures the code is checked out, Node.js is installed, and dependencies are available before running tests.
