## Task 1 — Web Application Scanning with OWASP ZAP

### Objective

In this task I scanned an intentionally vulnerable web application (OWASP Juice Shop) with OWASP ZAP on my VPS. The goal was to identify common web vulnerabilities, check HTTP security headers, and analyze the scan results.

### 1.1 Starting the target application

First, I started the Juice Shop container locally on the VPS:

```bash
docker run -d --name juice-shop -p 3000:3000 bkimminich/juice-shop
````

Then I waited until the application became available and checked it with `curl -I`:

```bash
curl -I http://127.0.0.1:3000
```

Output:

```text
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Sun, 05 Apr 2026 14:59:33 GMT
ETag: W/"124fa-19d5e28014e"
Content-Type: text/html; charset=UTF-8
Content-Length: 75002
Vary: Accept-Encoding
Date: Sun, 05 Apr 2026 14:59:33 GMT
Connection: keep-alive
Keep-Alive: timeout=5
```

This confirmed that the application was running correctly on port 3000.

### 1.2 Running ZAP on Linux VPS

Because I was running the lab on a Linux VPS, I could not use `host.docker.internal`, so I worked with the local container networking.

At first I tried the baseline scan, but the first attempt failed because ZAP could not write files into the mounted directory. After that I created a writable `out` directory and ran a full ZAP scan from Docker with report generation enabled.

Successful command:

```bash
docker run --name zap-full --network host -u zap -v "$PWD/out":/zap/wrk:rw -i ghcr.io/zaproxy/zaproxy:stable bash -lc '
python3 -u /zap/zap-full-scan.py \
  -t http://127.0.0.1:3000 \
  -m 3 \
  -T 15 \
  -j \
  -g gen-full.conf \
  -r zap-full-report.html \
  -w zap-full-report.md \
  -J zap-full-report.json \
  -I
'
```

ZAP also generated the following files:

```text
-rw-r--r-- 1 1000 1000 8.5K Apr  5 17:46 out/gen-full.conf
-rw-r--r-- 1 1000 1000 187K Apr  5 17:46 out/zap-full-report.html
-rw-r--r-- 1 1000 1000  77K Apr  5 17:46 out/zap-full-report.json
-rw-r--r-- 1 1000 1000  64K Apr  5 17:46 out/zap-full-report.md
-rw-r--r-- 1 root root 104K Apr  5 17:46 zap-full-zap.log
```

### 1.3 Scan results

The final scan completed successfully.

Important part of the console output:

```text
Total of 201 URLs
...
WARN-NEW: Missing Anti-clickjacking Header [10020] x 1
WARN-NEW: X-Content-Type-Options Header Missing [10021] x 3
WARN-NEW: Content Security Policy (CSP) Header Not Set [10038] x 11
WARN-NEW: Deprecated Feature Policy Header Set [10063] x 11
WARN-NEW: Backup File Disclosure [10095] x 31
WARN-NEW: Timestamp Disclosure - Unix [10096] x 9
WARN-NEW: Cross-Domain Misconfiguration [10098] x 11
WARN-NEW: Dangerous JS Functions [10110] x 2
WARN-NEW: Private IP Disclosure [2] x 1
WARN-NEW: Session ID in URL Rewrite [3] x 9
WARN-NEW: SQL Injection [40018] x 1
WARN-NEW: Bypassing 403 [40038] x 6
WARN-NEW: CORS Misconfiguration [40040] x 158
WARN-NEW: Cross-Origin-Embedder-Policy Header Missing or Invalid [90004] x 10

FAIL-NEW: 0     FAIL-INPROG: 0  WARN-NEW: 14    WARN-INPROG: 0  INFO: 0 IGNORE: 0       PASS: 127
ZAP_EXIT_CODE=0
```

**Number of Medium risk vulnerabilities found:**
`[replace with the exact Medium number from the HTML report overview]`

Even from the console output it is clear that the application has multiple security issues, including SQL Injection, 403 bypass, backup file disclosure, missing security headers, and CORS misconfiguration.

### 1.4 Two most interesting vulnerabilities

**1. SQL Injection**

One of the most important findings was SQL Injection:

```text
WARN-NEW: SQL Injection [40018] x 1
http://127.0.0.1:3000/rest/products/search?q=%27%28 (500 Internal Server Error)
```

This means user input in the search request was handled in an unsafe way and caused a server-side error during injection testing. SQL Injection is dangerous because it can allow an attacker to read or modify database data, bypass application logic, or in some cases get deeper access to the system.

**2. Backup File Disclosure**

Another interesting issue was Backup File Disclosure:

```text
WARN-NEW: Backup File Disclosure [10095] x 31
http://127.0.0.1:3000/ftp/quarantine.bak (200 OK)
http://127.0.0.1:3000/ftp/quarantine.backup (200 OK)
http://127.0.0.1:3000/ftp/quarantine.bac (200 OK)
http://127.0.0.1:3000/ftp/quarantine.zip (200 OK)
http://127.0.0.1:3000/ftp/quarantine.tar (200 OK)
```

This means backup copies of files were accessible directly from the web application. This is dangerous because backup files may contain source code, credentials, configuration, logs, or other sensitive information that should never be public.

### 1.5 Security headers status

From the initial HTTP check of the main page, some headers were already present:

* `X-Content-Type-Options: nosniff`
* `X-Frame-Options: SAMEORIGIN`
* `Feature-Policy: payment 'self'`

However, the full ZAP scan showed that security headers were not configured consistently across all responses.

**Present on the main page:**

* `X-Frame-Options`
* `X-Content-Type-Options`
* `Feature-Policy`

**Missing or problematic on other responses according to ZAP:**

* Missing Anti-clickjacking Header
* X-Content-Type-Options Header Missing
* Content Security Policy (CSP) Header Not Set
* Cross-Origin-Embedder-Policy Header Missing or Invalid
* Deprecated Feature Policy Header Set

Why they matter:

* **X-Frame-Options / anti-clickjacking protection** helps prevent the site from being embedded into a malicious page.
* **X-Content-Type-Options** helps prevent MIME-type confusion.
* **Content-Security-Policy (CSP)** reduces the risk of XSS and unsafe resource loading.
* **Cross-Origin-Embedder-Policy** is part of stronger browser isolation controls.
* **Deprecated Feature-Policy** shows that an older header is still being used instead of a modern policy approach.

So, the application had some protection on the main page, but overall the header configuration was incomplete and inconsistent.

### 1.6 Screenshot

Add screenshot of the generated HTML report overview here.

Example:

```md
![OWASP ZAP report overview](images/zap-full-report-overview.png)
```

### 1.7 Analysis

The most common issues in this scan were configuration and exposure problems: missing security headers, cross-origin policy issues, backup file disclosure, and access control weaknesses. These are common in real web applications because they often come from default settings, forgotten files, weak deployment hygiene, or inconsistent server configuration.

At the same time, the scan also found a more serious vulnerability class — SQL Injection. This shows that web applications can have both simple misconfigurations and deeper input-handling flaws at the same time.

In my opinion, the main lesson from this task is that even a locally deployed training application quickly shows how many different problem categories a web scanner can detect: input validation issues, missing browser protections, information disclosure, and broken access control.


## Task 2 — Container Vulnerability Scanning with Trivy

### Objective

In this task I scanned the `bkimminich/juice-shop` container image with Trivy in order to identify high and critical vulnerabilities before deployment. The goal was to check the image for known package-level security issues and understand what kinds of risks exist inside the container.

### 2.1 Running the Trivy scan

I ran the scan on my VPS using Docker. The original lab command uses `aquasec/trivy:latest`, but on my side that image tag was not available, so I used the current working Trivy image from GHCR and scanned the same target image with the required severity filter.

Command used:

```bash id="srfhw8"
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$PWD":/work \
  -w /work \
  ghcr.io/aquasecurity/trivy:0.69.3 image \
  --scanners vuln \
  --severity HIGH,CRITICAL \
  --format table \
  --output trivy-report.txt \
  bkimminich/juice-shop
````

I also generated a JSON report for easier review:

```bash id="bwndx3"
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$PWD":/work \
  -w /work \
  ghcr.io/aquasecurity/trivy:0.69.3 image \
  --scanners vuln \
  --severity HIGH,CRITICAL \
  --format json \
  --output trivy-report.json \
  bkimminich/juice-shop
```

### 2.2 Important terminal output

Relevant part of the Trivy output:

```text id="8mu1xh"
2026-04-05T19:05:02Z    INFO    [vulndb] Need to update DB
2026-04-05T19:05:02Z    INFO    [vulndb] Downloading vulnerability DB...
...
2026-04-05T19:05:42Z    INFO    Detected OS     family="debian" version="13.4"
2026-04-05T19:05:42Z    INFO    [debian] Detecting vulnerabilities...   os_version="13" pkg_num=13
2026-04-05T19:05:42Z    INFO    Number of language-specific files       num=1
2026-04-05T19:05:42Z    INFO    [node-pkg] Detecting vulnerabilities...
2026-04-05T19:05:42Z    WARN    Using severities from other vendors for some vulnerabilities.
2026-04-05T19:05:43Z    INFO    Table result includes only package filenames. Use '--format json' option to get the full path to the package file.
```

Report summary:

```text id="qm0i60"
Report Summary

Target                                               Type      Vulnerabilities
bkimminich/juice-shop (debian 13.4)                  debian    0
...
Node.js (node-pkg)
==================
Total: 53 (HIGH: 44, CRITICAL: 9)
```

Generated files:

```text id="4ewmbt"
-rw-r--r-- 1 root root 855K Apr  5 19:06 trivy-report.json
-rw-r--r-- 1 root root 578K Apr  5 19:05 trivy-report.txt
```

### 2.3 Key findings

**Total number of CRITICAL vulnerabilities:** 9
**Total number of HIGH vulnerabilities:** 44 

The scan showed that the Debian base layer had no high or critical issues, while the vulnerable findings came from Node.js dependencies inside the application.  

Examples of vulnerable packages with CVE IDs:

1. **braces** — `CVE-2024-4068` — HIGH
   Installed version: `2.3.2`
   Fixed version: `3.0.3`
   Issue: the package fails to limit the number of characters it can handle. 

2. **lodash** — `CVE-2019-10744` — CRITICAL
   Installed version: `2.4.2`
   Fixed version: `4.17.12`
   Issue: prototype pollution in `defaultsDeep`, which can lead to modification of object properties. 

Another critical example from the scan:

* **vm2** — `CVE-2023-32314` — CRITICAL
  Installed version: `3.9.17`
  Fixed version: `3.9.18`
  Issue: sandbox escape. 

### 2.4 Most common vulnerability type

The dominant problem in this image was not the OS layer, but vulnerable **Node.js third-party dependencies**. The scan summary shows `0` vulnerabilities in the Debian base image and `53` in `node-pkg`, so the main risk clearly came from application libraries rather than the container base system.  

From the visible findings, the recurring vulnerability patterns included:

* prototype pollution,
* sandbox escape,
* denial of service,
* path traversal / file overwrite issues.

So the most common issue category in practice was **insecure application dependencies in the Node.js ecosystem**.

### 2.5 Screenshot

Add a screenshot of the terminal output with the vulnerability summary and critical findings here.

Example:

```md id="v2b0c9"
![Trivy scan output](images/trivy-critical-findings.png)
```

### 2.6 Analysis

Container image scanning is important before production deployment because an application can inherit security problems from both the base image and its dependencies. In this case, the operating system layer was clean, but the Node.js dependency tree still contained many serious issues. This means that checking only the base image is not enough.

A vulnerable container can be deployed successfully and still contain known exploitable packages. If such an image reaches production, an attacker may take advantage of dependency flaws such as prototype pollution, sandbox escape, path traversal, or denial of service. Scanning early helps detect these problems before release and gives the team a chance to upgrade or replace risky packages.

### 2.7 Reflection: CI/CD integration

I would integrate Trivy into CI/CD in a simple way:

1. Run Trivy automatically on every build or pull request.
2. Save the scan results as artifacts (`table` or `json` reports).
3. Fail the pipeline if any CRITICAL vulnerabilities are found.
4. Optionally fail or warn on HIGH vulnerabilities depending on project policy.
5. Add a regular scheduled scan for base images and dependencies, because new CVEs can appear even when the code itself has not changed.

This would make vulnerability scanning part of the normal delivery process instead of a manual step at the very end.
