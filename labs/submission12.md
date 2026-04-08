## Preparation

All work for this lab was done on the Ubuntu server directly in `labs/lab12/`.

First, I checked that the lab directory and all required files were already present.

### Check lab directory

```bash
cd /root/labs/lab12
ls -la
````

Output:

```text
total 24
drwxr-xr-x 2 root root 4096 Apr  8 08:59 .
drwxr-xr-x 4 root root 4096 Apr  8 08:58 ..
-rw-r--r-- 1 root root  432 Apr  8 08:59 Dockerfile
-rw-r--r-- 1 root root   79 Apr  8 08:59 Dockerfile.wasm
-rw-r--r-- 1 root root 3456 Apr  8 08:59 main.go
-rw-r--r-- 1 root root  305 Apr  8 08:59 spin.toml
```

This confirmed that the lab files were already available:

* `main.go`
* `Dockerfile`
* `Dockerfile.wasm`
* `spin.toml`

### Check installed tooling

I also verified that the required container tooling was available on the server.

```bash
docker --version
containerd --version
ctr version
```

Output:

```text
Docker version 28.5.2, build ecc6942
containerd containerd.io v1.7.28 b98a3aace656320842a23f4a392a33f46af97866
Client:
  Version:  v1.7.28
  Revision: b98a3aace656320842a23f4a392a33f46af97866
  Go version: go1.24.9

Server:
  Version:  v1.7.28
  Revision: b98a3aace656320842a23f4a392a33f46af97866
  UUID: 23b78f2a-9585-43f1-92bc-e86ada0f9c8f
```

### Install missing packages

Before preparing the WASM workflow, I installed the helper packages needed for downloading, unpacking, and verifying binaries.

```bash
apt-get update
apt-get install -y ca-certificates curl jq file time git golang-go tar gzip unzip
```

Output:

```text
Hit:1 https://deb.nodesource.com/node_20.x nodistro InRelease
Hit:2 https://download.docker.com/linux/ubuntu noble InRelease
Hit:3 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:4 http://nova.clouds.archive.ubuntu.com/ubuntu noble InRelease
Hit:5 http://nova.clouds.archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:6 http://nova.clouds.archive.ubuntu.com/ubuntu noble-backports InRelease
Reading package lists... Done
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ca-certificates is already the newest version (20240203).
curl is already the newest version (8.5.0-2ubuntu10.8).
jq is already the newest version (1.7.1-3ubuntu0.24.04.1).
file is already the newest version (1:5.45-3build1).
time is already the newest version (1.9-0.2build1).
git is already the newest version (1:2.43.0-1ubuntu7.3).
golang-go is already the newest version (2:1.22~2build1).
tar is already the newest version (1.35+dfsg-3build1).
gzip is already the newest version (1.12-1ubuntu3.1).
unzip is already the newest version (6.0-28ubuntu4.1).
0 upgraded, 0 newly installed, 0 to remove and 183 not upgraded.
```

### Install the Wasmtime shim for containerd

To prepare the WASM container runtime, I downloaded the prebuilt `containerd-shim-wasmtime-v1` binary from the runwasi release and installed it into `/usr/local/bin/`.

```bash
curl -fL "https://github.com/containerd/runwasi/releases/download/containerd-shim-wasmtime/v0.6.0/containerd-shim-wasmtime-x86_64-linux-musl.tar.gz" -o containerd-shim-wasmtime-x86_64-linux-musl.tar.gz
tar -xzf containerd-shim-wasmtime-x86_64-linux-musl.tar.gz
install -m 0755 containerd-shim-wasmtime-v1 /usr/local/bin/containerd-shim-wasmtime-v1
ls -l /usr/local/bin/containerd-shim-wasmtime-v1
```

Output:

```text
Downloading: https://github.com/containerd/runwasi/releases/download/containerd-shim-wasmtime/v0.6.0/containerd-shim-wasmtime-x86_64-linux-musl.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
100  9.8M  100  9.8M    0     0   474k      0  0:00:21  0:00:21 --:--:-- 1222k

-rwxr-xr-x 1 root root 28765376 Apr  8 18:29 /usr/local/bin/containerd-shim-wasmtime-v1
```

### Verify that the WASM runtime works with `ctr`

After installing the shim, I pulled the demo WASM image and tested it with `ctr`.

```bash
ctr images pull ghcr.io/containerd/runwasi/wasi-demo-app:latest
ctr run --rm --runtime=io.containerd.wasmtime.v1 ghcr.io/containerd/runwasi/wasi-demo-app:latest testwasm
```

Output:

```text
ghcr.io/containerd/runwasi/wasi-demo-app:latest:                                  resolved       |++++++++++++++++++++++++++++++++++++++|
manifest-sha256:1a5ef678e7425a98de8166d9e289e09e21d8a82312ad7e5c8bf9b961bb1f2666: done           |++++++++++++++++++++++++++++++++++++++|
layer-sha256:5e274030b46dbf5e38a5443ccb220d1023a0a9d1f654fa733c2b3dba62ae98ac:    done           |++++++++++++++++++++++++++++++++++++++|
config-sha256:329a0ea93324142d5c7af24210a3d9a20642157e9d427be98f775e130a0f59eb:   done           |++++++++++++++++++++++++++++++++++++++|
elapsed: 1.6 s                                                                    total:  2.2 Mi (1.3 MiB/s)
unpacking linux/amd64 sha256:1a5ef678e7425a98de8166d9e289e09e21d8a82312ad7e5c8bf9b961bb1f2666...
done: 132.197574ms

This is a song that never ends.
Yes, it goes on and on my friends.
Some people started singing it not knowing what it was,
So they'll continue singing it forever just because...

This is a song that never ends.
Yes, it goes on and on my friends.
Some people started singing it not knowing what it was,
So they'll continue singing it forever just because...

...
```

The demo application kept running and printing output continuously, which showed that the Wasmtime-based runtime path was working.

To make the verification shorter and cleaner, I ran the same image again with an explicit `echo` command:

```bash
ctr run --rm \
  --runtime=io.containerd.wasmtime.v1 \
  ghcr.io/containerd/runwasi/wasi-demo-app:latest testwasm-echo \
  /wasi-demo-app.wasm echo hello
```

Output:

```text
hello
exiting
```

This was the final confirmation that `ctr` + `io.containerd.wasmtime.v1` worked correctly on the server.

### Install Spin for the bonus WASM server mode

For the bonus part, I also installed Spin.

```bash
curl -fsSL https://developer.fermyon.com/downloads/install.sh | bash
install -m 0755 /root/spin /usr/local/bin/spin
spin --version
```

Output:

```text
Step 1: Downloading: https://github.com/spinframework/spin/releases/download/v3.6.2/spin-v3.6.2-linux-amd64.tar.gz
Done...

Step 2: Decompressing: spin-v3.6.2-linux-amd64.tar.gz
crt.pem
spin.sig
README.md
LICENSE
spin
spin 3.6.2 (c0fc970 2026-02-25)
Done...

Step 3: Removing the downloaded tarball
Done...

Step 4: Installing default templates
Copying remote template source
Installing template http-grain...
Installing template http-rust...
Installing template http-zig...
Installing template static-fileserver...
Installing template http-php...
Installing template redis-rust...
Installing template http-rust-wasip3-unstable...
Installing template http-empty...
Installing template redis-go...
Installing template http-go...
Installing template http-c...
Installing template redirect...
Installed 12 template(s)

+------------------------------------------------------------------------------------------+
| Name                        Description                                                  |
+==========================================================================================+
| http-c                      HTTP request handler using C and the Zig toolchain           |
| http-empty                  HTTP application with no components                          |
| http-go                     HTTP request handler using (Tiny)Go                          |
| http-grain                  HTTP request handler using Grain                             |
| http-php                    HTTP request handler using PHP                               |
| http-rust                   HTTP request handler using Rust                              |
| http-rust-wasip3-unstable   WASIp3 HTTP request handler using Rust (0.3.0-rc-2025-09-16) |
| http-zig                    HTTP request handler using Zig                               |
| redirect                    Redirects a HTTP route                                       |
| redis-go                    Redis message handler using (Tiny)Go                         |
| redis-rust                  Redis message handler using Rust                             |
| static-fileserver           Serves static files from an asset directory                  |
+------------------------------------------------------------------------------------------+

Copying remote template source
Installing template http-py...
Installed 1 template(s)

+---------------------------------------------+
| Name      Description                       |
+=============================================+
| http-py   HTTP request handler using Python |
+---------------------------------------------+

Copying remote template source
Installing template http-ts...
Installing template redis-js...
Installing template http-js...
Installing template redis-ts...
Installed 4 template(s)

+---------------------------------------------------+
| Name       Description                            |
+===================================================+
| http-js    HTTP request handler using JavaScript  |
| http-ts    HTTP request handler using TypeScript  |
| redis-js   Redis message handler using JavaScript |
| redis-ts   Redis message handler using TypeScript |
+---------------------------------------------------+

Step 5: Installing default plugins
Plugin information updated successfully
Plugin 'cloud' was installed successfully!

Description:
        Commands for publishing applications to the Fermyon Cloud.

Homepage:
        https://github.com/fermyon/cloud-plugin
You're good to go. Check here for the next steps: https://spinframework.dev/quickstart
Run './spin' to get started

spin 3.6.2 (c0fc970 2026-02-25)
```

### Final readiness check

Finally, I verified that Spin was installed, the WASM demo worked with `ctr`, and the lab files were still present.

```bash
spin --version
ctr run --rm --runtime=io.containerd.wasmtime.v1 ghcr.io/containerd/runwasi/wasi-demo-app:latest testwasm-echo /wasi-demo-app.wasm echo hello
cd /root/labs/lab12 && ls -la
```

Output:

```text
spin 3.6.2 (c0fc970 2026-02-25)
hello
exiting

total 24
drwxr-xr-x 2 root root 4096 Apr  8 08:59 .
drwxr-xr-x 4 root root 4096 Apr  8 08:58 ..
-rw-r--r-- 1 root root  432 Apr  8 08:59 Dockerfile
-rw-r--r-- 1 root root   79 Apr  8 08:59 Dockerfile.wasm
-rw-r--r-- 1 root root 3456 Apr  8 08:59 main.go
-rw-r--r-- 1 root root  305 Apr  8 08:59 spin.toml
```

At this point, the environment was ready for:

* traditional Docker build and benchmark,
* WASM build and execution with `ctr`,
* optional local or cloud Spin testing for the bonus task.

