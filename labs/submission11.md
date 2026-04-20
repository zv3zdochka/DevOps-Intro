## Task 1 — Build Reproducible Artifacts from Scratch

### 1.1 Install Nix Package Manager

#### Command
```bash
curl --proto '=https' --tlsv1.2 -sSf -L https://install.determinate.systems/nix | sh -s -- install --no-confirm

if [ -f /nix/var/nix/profiles/default/etc/profile.d/nix-daemon.sh ]; then
  . /nix/var/nix/profiles/default/etc/profile.d/nix-daemon.sh
fi

export PATH="/nix/var/nix/profiles/default/bin:$PATH"
hash -r

nix --version
nix run nixpkgs#hello
````

#### Output

```text
info: downloading the Determinate Nix Installer
 INFO nix-installer v3.17.2
 INFO Step: Create directory `/nix`
 INFO Step: Install Determinate Nixd
 INFO Step: Provision Nix
 INFO Step: Create build users (UID 30001-30032) and group (GID 30000)
 INFO Step: Configure Nix
 INFO Step: Create directory `/etc/tmpfiles.d`
 INFO Step: Configure the Determinate Nix daemon
 INFO Step: Cleanup
 INFO Running self test for shell sh
 INFO Running self test for shell bash
Nix was installed successfully!
To get started using Nix, open a new shell or run `. /nix/var/nix/profiles/default/etc/profile.d/nix-daemon.sh`

nix (Determinate Nix 3.17.2) 2.33.3
Hello, world!
```

### 1.2 Create a Simple Application

#### `main.go`

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Printf("Built with Nix at compile time\n")
    fmt.Printf("Running at: %s\n", time.Now().Format(time.RFC3339))
}
```

#### `go.mod`

```go
module app

go 1.22
```

### 1.3 Write a Nix Derivation

#### `default.nix`

```nix
let
  pkgs = import (fetchTarball "https://github.com/NixOS/nixpkgs/archive/nixos-25.05.tar.gz") {};
in
pkgs.buildGoModule {
  pname = "app";
  version = "1.0.0";
  src = pkgs.lib.cleanSource ./.;
  vendorHash = null;
}
```

I used `pkgs.lib.cleanSource ./.` so that logs, temporary files, and build artifacts would not affect the source hash. I also set `vendorHash = null;` because this Go program has no external dependencies to vendor.

### 1.4 Prove Reproducibility

#### Command

```bash
. /nix/var/nix/profiles/default/etc/profile.d/nix-daemon.sh
export PATH="/nix/var/nix/profiles/default/bin:$PATH"
hash -r

mkdir -p /root/labs/lab11/repro-src
mkdir -p /root/labs/lab11/proof

cp /root/labs/lab11/app/main.go /root/labs/lab11/repro-src/main.go
cp /root/labs/lab11/app/go.mod /root/labs/lab11/repro-src/go.mod

cat > /root/labs/lab11/repro-src/default.nix <<'EOF'
let
  pkgs = import (fetchTarball "https://github.com/NixOS/nixpkgs/archive/nixos-25.05.tar.gz") {};
in
pkgs.buildGoModule {
  pname = "app";
  version = "1.0.0";
  src = pkgs.lib.cleanSource ./.;
  vendorHash = null;
}
EOF

cd /root/labs/lab11/repro-src

rm -f result
nix-build 2>&1 | tee /root/labs/lab11/proof/nix-build-repro-1.log
./result/bin/app | tee /root/labs/lab11/proof/app-run-repro-1.txt
readlink result | tee /root/labs/lab11/proof/store-path-repro-1.txt
sha256sum ./result/bin/app | tee /root/labs/lab11/proof/binary-sha256-repro-1.txt

rm -f result
nix-build 2>&1 | tee /root/labs/lab11/proof/nix-build-repro-2.log
./result/bin/app | tee /root/labs/lab11/proof/app-run-repro-2.txt
readlink result | tee /root/labs/lab11/proof/store-path-repro-2.txt
sha256sum ./result/bin/app | tee /root/labs/lab11/proof/binary-sha256-repro-2.txt

printf 'STORE_PATH_1=%s\nSTORE_PATH_2=%s\n' "$(cat /root/labs/lab11/proof/store-path-repro-1.txt)" "$(cat /root/labs/lab11/proof/store-path-repro-2.txt)" | tee /root/labs/lab11/proof/store-path-repro-compare.txt
cmp -s /root/labs/lab11/proof/store-path-repro-1.txt /root/labs/lab11/proof/store-path-repro-2.txt && echo IDENTICAL_STORE_PATHS | tee -a /root/labs/lab11/proof/store-path-repro-compare.txt || echo DIFFERENT_STORE_PATHS | tee -a /root/labs/lab11/proof/store-path-repro-compare.txt

printf 'SHA256_1=%s\nSHA256_2=%s\n' "$(awk '{print $1}' /root/labs/lab11/proof/binary-sha256-repro-1.txt)" "$(awk '{print $1}' /root/labs/lab11/proof/binary-sha256-repro-2.txt)" | tee /root/labs/lab11/proof/binary-sha256-repro-compare.txt
cmp -s /root/labs/lab11/proof/binary-sha256-repro-1.txt /root/labs/lab11/proof/binary-sha256-repro-2.txt && echo IDENTICAL_BINARY_HASHES | tee -a /root/labs/lab11/proof/binary-sha256-repro-compare.txt || echo DIFFERENT_BINARY_HASHES | tee -a /root/labs/lab11/proof/binary-sha256-repro-compare.txt
```

#### Output

```text
/nix/store/7lv0d4i4ykknmkhnq326i64ndrn4c3y2-app-1.0.0
Built with Nix at compile time
Running at: 2026-04-06T12:25:29Z
/nix/store/7lv0d4i4ykknmkhnq326i64ndrn4c3y2-app-1.0.0
34171ef70af8cc848dee57e8dc284fc2d088626c24d3217c0b6101576c6b421a  ./result/bin/app
/nix/store/7lv0d4i4ykknmkhnq326i64ndrn4c3y2-app-1.0.0
Built with Nix at compile time
Running at: 2026-04-06T12:25:30Z
/nix/store/7lv0d4i4ykknmkhnq326i64ndrn4c3y2-app-1.0.0
34171ef70af8cc848dee57e8dc284fc2d088626c24d3217c0b6101576c6b421a  ./result/bin/app
STORE_PATH_1=/nix/store/7lv0d4i4ykknmkhnq326i64ndrn4c3y2-app-1.0.0
STORE_PATH_2=/nix/store/7lv0d4i4ykknmkhnq326i64ndrn4c3y2-app-1.0.0
IDENTICAL_STORE_PATHS
SHA256_1=34171ef70af8cc848dee57e8dc284fc2d088626c24d3217c0b6101576c6b421a
SHA256_2=34171ef70af8cc848dee57e8dc284fc2d088626c24d3217c0b6101576c6b421a
IDENTICAL_BINARY_HASHES
```

The result symlink pointed to the same Nix store path in both builds, and the SHA256 hash of the produced binary was also identical. This shows that the build is reproducible.

### Docker Comparison

#### `Dockerfile`

```dockerfile
FROM golang:1.22
WORKDIR /app
COPY main.go .
RUN go build -o app main.go
CMD ["/app/app"]
```

#### Command

```bash
. /nix/var/nix/profiles/default/etc/profile.d/nix-daemon.sh
export PATH="/nix/var/nix/profiles/default/bin:$PATH"
hash -r

cd /root/labs/lab11/repro-src
mkdir -p /root/labs/lab11/proof

cat > Dockerfile <<'EOF'
FROM golang:1.22
WORKDIR /app
COPY main.go .
RUN go build -o app main.go
CMD ["/app/app"]
EOF

docker build --pull --no-cache -t test-app:run1 . 2>&1 | tee /root/labs/lab11/proof/docker-build-run1.log
docker image inspect test-app:run1 --format 'ID={{.Id}} CREATED={{.Created}}' | tee /root/labs/lab11/proof/docker-image-run1.txt

sleep 2

docker build --pull --no-cache -t test-app:run2 . 2>&1 | tee /root/labs/lab11/proof/docker-build-run2.log
docker image inspect test-app:run2 --format 'ID={{.Id}} CREATED={{.Created}}' | tee /root/labs/lab11/proof/docker-image-run2.txt

printf 'RUN1=%s\nRUN2=%s\n' "$(docker image inspect test-app:run1 --format '{{.Id}}')" "$(docker image inspect test-app:run2 --format '{{.Id}}')" | tee /root/labs/lab11/proof/docker-image-ids-compare.txt

docker images --no-trunc | grep 'test-app' | tee /root/labs/lab11/proof/docker-images-list.txt
```

#### Output

```text
#10 writing image sha256:8adebb017053d90b6c34bc3712d24d7e7d65278347920cd4c9048d4464f02bf6 done
#10 naming to docker.io/library/test-app:run1 done
#10 DONE 0.4s
ID=sha256:8adebb017053d90b6c34bc3712d24d7e7d65278347920cd4c9048d4464f02bf6 CREATED=2026-04-06T12:30:45.41708609Z
#9 writing image sha256:ca37064d50f4641d5e6801105c251672b5749c7e67592e7d86109bc486e012de done
#9 naming to docker.io/library/test-app:run2 done
#9 DONE 0.4s
ID=sha256:ca37064d50f4641d5e6801105c251672b5749c7e67592e7d86109bc486e012de CREATED=2026-04-06T12:31:01.918746196Z
RUN1=sha256:8adebb017053d90b6c34bc3712d24d7e7d65278347920cd4c9048d4464f02bf6
RUN2=sha256:ca37064d50f4641d5e6801105c251672b5749c7e67592e7d86109bc486e012de
DIFFERENT_DOCKER_IMAGE_IDS
test-app                     run2      sha256:ca37064d50f4641d5e6801105c251672b5749c7e67592e7d86109bc486e012de   1 second ago     852MB
test-app                     run1      sha256:8adebb017053d90b6c34bc3712d24d7e7d65278347920cd4c9048d4464f02bf6   17 seconds ago   852MB
```

The two Docker builds produced different image IDs even though the source code was the same. The `CREATED` timestamps were also different. This shows that a normal Docker build is not reproducible by default.

### Analysis

Nix builds are reproducible because the build runs in an isolated environment with declared dependencies only. The derivation describes the exact inputs, and the output is placed in the Nix store under a content-addressed path. If the inputs do not change, the resulting store path and binary hash stay the same.

Docker is different in this comparison because the image metadata changes between builds. Even when the application code is identical, the final image still gets a different image ID.

### Nix Store Path Explanation

Example store path:

```text
/nix/store/7lv0d4i4ykknmkhnq326i64ndrn4c3y2-app-1.0.0
```

* `/nix/store` is the global Nix store.
* `7lv0d4i4ykknmkhnq326i64ndrn4c3y2` is the hash derived from the build inputs.
* `app-1.0.0` is the package name and version.

Because the path includes the hash of the inputs, changing the inputs changes the path, while identical inputs keep the same path.


## Task 2 — Reproducible Docker Images with Nix

### 2.1 Build Docker Image with Nix

#### `docker.nix`
```nix
let
  pkgs = import (fetchTarball "https://github.com/NixOS/nixpkgs/archive/nixos-25.05.tar.gz") {};
  app = pkgs.buildGoModule {
    pname = "app";
    version = "1.0.0";
    src = pkgs.lib.cleanSource ./.;
    vendorHash = null;
  };
in
pkgs.dockerTools.buildLayeredImage {
  name = "nix-app";
  tag = "latest";
  contents = [ app ];
  config = {
    Cmd = [ "/bin/app" ];
  };
}
```

I reused the same reproducible Go build from Task 1 and packaged it with `dockerTools.buildLayeredImage`. The source is cleaned with `pkgs.lib.cleanSource ./.`, and no current timestamp is injected into the image metadata.

#### Command

```bash
cat > /root/lab11_task2.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail

. /nix/var/nix/profiles/default/etc/profile.d/nix-daemon.sh
export PATH="/nix/var/nix/profiles/default/bin:$PATH"
hash -r

SRC_DIR="/root/labs/lab11/repro-src"
PROOF_DIR="/root/labs/lab11/task2-proof"
TRAD_DIR="/root/labs/lab11/traditional-app"

mkdir -p "$SRC_DIR" "$PROOF_DIR" "$TRAD_DIR"

cd "$SRC_DIR"

cat > docker.nix <<'EONIX'
let
  pkgs = import (fetchTarball "https://github.com/NixOS/nixpkgs/archive/nixos-25.05.tar.gz") {};
  app = pkgs.buildGoModule {
    pname = "app";
    version = "1.0.0";
    src = pkgs.lib.cleanSource ./.;
    vendorHash = null;
  };
in
pkgs.dockerTools.buildLayeredImage {
  name = "nix-app";
  tag = "latest";
  contents = [ app ];
  config = {
    Cmd = [ "/bin/app" ];
  };
}
EONIX

cp docker.nix "$PROOF_DIR/docker.nix"

rm -f result
nix-build docker.nix 2>&1 | tee "$PROOF_DIR/nix-docker-build-1.log"
ls -lhL result | tee "$PROOF_DIR/nix-image-result-size-1.txt"
readlink result | tee "$PROOF_DIR/nix-image-store-path-1.txt"
sha256sum result | tee "$PROOF_DIR/nix-image-tar-sha256-1.txt"
docker load -i result | tee "$PROOF_DIR/nix-docker-load-1.txt"
docker image inspect nix-app:latest --format 'ID={{.Id}} CREATED={{.Created}} SIZE={{.Size}}' | tee "$PROOF_DIR/nix-docker-image-1.txt"
docker run --rm nix-app:latest | tee "$PROOF_DIR/nix-docker-run.txt"
docker history nix-app:latest --no-trunc | tee "$PROOF_DIR/nix-docker-history-1.txt"

rm -f result
nix-build docker.nix 2>&1 | tee "$PROOF_DIR/nix-docker-build-2.log"
ls -lhL result | tee "$PROOF_DIR/nix-image-result-size-2.txt"
readlink result | tee "$PROOF_DIR/nix-image-store-path-2.txt"
sha256sum result | tee "$PROOF_DIR/nix-image-tar-sha256-2.txt"
docker load -i result | tee "$PROOF_DIR/nix-docker-load-2.txt"
docker image inspect nix-app:latest --format 'ID={{.Id}} CREATED={{.Created}} SIZE={{.Size}}' | tee "$PROOF_DIR/nix-docker-image-2.txt"

printf 'STORE_PATH_1=%s\nSTORE_PATH_2=%s\n' "$(cat "$PROOF_DIR/nix-image-store-path-1.txt")" "$(cat "$PROOF_DIR/nix-image-store-path-2.txt")" | tee "$PROOF_DIR/nix-image-store-path-compare.txt"
cmp -s "$PROOF_DIR/nix-image-store-path-1.txt" "$PROOF_DIR/nix-image-store-path-2.txt" && echo IDENTICAL_NIX_IMAGE_STORE_PATHS | tee -a "$PROOF_DIR/nix-image-store-path-compare.txt" || echo DIFFERENT_NIX_IMAGE_STORE_PATHS | tee -a "$PROOF_DIR/nix-image-store-path-compare.txt"

printf 'SHA256_1=%s\nSHA256_2=%s\n' "$(awk '{print $1}' "$PROOF_DIR/nix-image-tar-sha256-1.txt")" "$(awk '{print $1}' "$PROOF_DIR/nix-image-tar-sha256-2.txt")" | tee "$PROOF_DIR/nix-image-tar-sha256-compare.txt"
cmp -s "$PROOF_DIR/nix-image-tar-sha256-1.txt" "$PROOF_DIR/nix-image-tar-sha256-2.txt" && echo IDENTICAL_NIX_IMAGE_TAR_HASHES | tee -a "$PROOF_DIR/nix-image-tar-sha256-compare.txt" || echo DIFFERENT_NIX_IMAGE_TAR_HASHES | tee -a "$PROOF_DIR/nix-image-tar-sha256-compare.txt"

cp "$SRC_DIR/main.go" "$TRAD_DIR/main.go"

cat > "$TRAD_DIR/Dockerfile.traditional" <<'EODOCKER'
FROM golang:1.22 AS builder
WORKDIR /src
COPY main.go .
ENV CGO_ENABLED=0
RUN go build -o /out/app main.go

FROM scratch
COPY --from=builder /out/app /app
ENTRYPOINT ["/app"]
EODOCKER

cd "$TRAD_DIR"
docker build --pull --no-cache -f Dockerfile.traditional -t traditional-app:latest . 2>&1 | tee "$PROOF_DIR/traditional-build.log"
docker image inspect traditional-app:latest --format 'ID={{.Id}} CREATED={{.Created}} SIZE={{.Size}}' | tee "$PROOF_DIR/traditional-image.txt"
docker history traditional-app:latest --no-trunc | tee "$PROOF_DIR/traditional-history.txt"

docker images | grep -E 'nix-app|traditional-app' | tee "$PROOF_DIR/image-size-compare.txt"

echo
echo "DONE"
echo "PROOF_DIR=$PROOF_DIR"
EOF

chmod +x /root/lab11_task2.sh
bash /root/lab11_task2.sh 2>&1 | tee /root/lab11_task2.log
```

#### Output

```text
/nix/store/9kyklqd7i175k92a0n5rdb6nbpywq9bx-nix-app.tar.gz
-r--r--r-- 1 root root 1.2M Jan  1  1970 result
/nix/store/9kyklqd7i175k92a0n5rdb6nbpywq9bx-nix-app.tar.gz
68cf4ede86734c585ab2435eb2e420f7842ab209e2f2c162cb50d5e034dcc1e7  result
Loaded image: nix-app:latest
ID=sha256:c89600ad540c4dfeac12df99eee588aebe0ac3e17c4d1063ea6eb313b7494a85 CREATED=1970-01-01T00:00:01Z SIZE=3517253
Built with Nix at compile time
Running at: 2026-04-07T13:20:24Z
IMAGE                                                                     CREATED   CREATED BY   SIZE      COMMENT
sha256:c89600ad540c4dfeac12df99eee588aebe0ac3e17c4d1063ea6eb313b7494a85   N/A                    61B       store paths: ['/nix/store/m7jav4mvmhq9nvqylssf2yafmhmiy3rb-nix-app-customisation-layer']
<missing>                                                                 N/A                    1.62MB    store paths: ['/nix/store/nrmkcfgq3byz4qddh53fzjq5xm09rzf6-app-1.0.0']
<missing>                                                                 N/A                    1.9MB     store paths: ['/nix/store/2s4hpq73hn49jd84x976m3acp3rd3k1x-tzdata-2025b']
/nix/store/9kyklqd7i175k92a0n5rdb6nbpywq9bx-nix-app.tar.gz
-r--r--r-- 1 root root 1.2M Jan  1  1970 result
/nix/store/9kyklqd7i175k92a0n5rdb6nbpywq9bx-nix-app.tar.gz
68cf4ede86734c585ab2435eb2e420f7842ab209e2f2c162cb50d5e034dcc1e7  result
Loaded image: nix-app:latest
ID=sha256:c89600ad540c4dfeac12df99eee588aebe0ac3e17c4d1063ea6eb313b7494a85 CREATED=1970-01-01T00:00:01Z SIZE=3517253
STORE_PATH_1=/nix/store/9kyklqd7i175k92a0n5rdb6nbpywq9bx-nix-app.tar.gz
STORE_PATH_2=/nix/store/9kyklqd7i175k92a0n5rdb6nbpywq9bx-nix-app.tar.gz
IDENTICAL_NIX_IMAGE_STORE_PATHS
SHA256_1=68cf4ede86734c585ab2435eb2e420f7842ab209e2f2c162cb50d5e034dcc1e7
SHA256_2=68cf4ede86734c585ab2435eb2e420f7842ab209e2f2c162cb50d5e034dcc1e7
IDENTICAL_NIX_IMAGE_TAR_HASHES
#0 building with "default" instance using docker driver

#1 [internal] load build definition from Dockerfile.traditional
#1 DONE 0.0s

#1 [internal] load build definition from Dockerfile.traditional
#1 transferring dockerfile: 226B done
#1 DONE 0.0s

#2 [internal] load metadata for docker.io/library/golang:1.22
#2 ...

#3 [auth] library/golang:pull token for registry-1.docker.io
#3 DONE 0.0s

#2 [internal] load metadata for docker.io/library/golang:1.22
#2 DONE 0.9s

#4 [internal] load .dockerignore
#4 transferring context: 2B done
#4 DONE 0.0s

#5 [builder 1/4] FROM docker.io/library/golang:1.22@sha256:1cf6c45ba39db9fd6db16922041d074a63c935556a05c5ccb62d181034df7f02
#5 CACHED

#6 [internal] load build context
#6 transferring context: 218B done
#6 DONE 0.1s

#7 [builder 2/4] WORKDIR /src
#7 DONE 0.1s

#8 [builder 3/4] COPY main.go .
#8 DONE 0.1s

#9 [builder 4/4] RUN go build -o /out/app main.go
#9 DONE 13.4s

#10 [stage-1 1/1] COPY --from=builder /out/app /app
#10 DONE 0.0s

#11 exporting to image
#11 exporting layers 0.0s done
#11 writing image sha256:2f18c7d624f5351e5ce5bd907c2fa73ebbd12ae9d5063bbda428636d60c2fbd5 0.0s done
#11 naming to docker.io/library/traditional-app:latest done
#11 DONE 0.1s
ID=sha256:2f18c7d624f5351e5ce5bd907c2fa73ebbd12ae9d5063bbda428636d60c2fbd5 CREATED=2026-04-07T13:20:51.218625588Z SIZE=2011378
IMAGE                                                                     CREATED                  CREATED BY                      SIZE      COMMENT
sha256:2f18c7d624f5351e5ce5bd907c2fa73ebbd12ae9d5063bbda428636d60c2fbd5   Less than a second ago   ENTRYPOINT ["/app"]             0B        buildkit.dockerfile.v0
<missing>                                                                 Less than a second ago   COPY /out/app /app # buildkit   2.01MB    buildkit.dockerfile.v0
traditional-app              latest    2f18c7d624f5   Less than a second ago   2.01MB
nix-app                      latest    c89600ad540c   56 years ago             3.52MB

DONE
PROOF_DIR=/root/labs/lab11/task2-proof
```

### Image Size Comparison

In this experiment, the Nix-built image was reproducible, but it was not smaller than the traditional `scratch` image:

* `traditional-app`: `2.01MB`
* `nix-app`: `3.52MB`

This happened because the Nix image included additional store content such as `tzdata`, while the traditional image only copied the final static binary into `scratch`.

### Reproducibility Proof

The Nix image tarball was built twice and produced:

* the same Nix store path:
  `/nix/store/9kyklqd7i175k92a0n5rdb6nbpywq9bx-nix-app.tar.gz`
* the same SHA256 hash:
  `68cf4ede86734c585ab2435eb2e420f7842ab209e2f2c162cb50d5e034dcc1e7`

This proves that the Nix-built Docker image artifact was reproducible.

### Layer Structure Comparison

For the Nix-built image, the history shows layers derived from Nix store paths:

* customisation layer
* application layer
* `tzdata` layer

The image metadata also had a fixed creation time:

* `CREATED=1970-01-01T00:00:01Z`

For the traditional Docker image, the history shows normal Dockerfile instructions:

* `COPY /out/app /app`
* `ENTRYPOINT ["/app"]`

The traditional image had a regular current build timestamp:

* `CREATED=2026-04-07T13:20:51.218625588Z`

### Analysis

The Nix-built Docker image is reproducible because the build inputs are fixed, the source is cleaned, and the resulting tarball is generated with stable metadata. This is why both the tarball hash and the store path stayed identical across repeated builds.

The traditional Docker image is simpler and smaller in this specific case, but it is not designed around content-addressed reproducibility in the same way. Its metadata depends on the build time, and standard Docker workflows do not guarantee identical image artifacts by default.

### Practical Advantages of Content-Addressable Docker Images

Nix-based Docker images provide several practical benefits:

* repeated builds can produce identical artifacts;
* the result is tied to exact declared inputs;
* layer contents are traceable through Nix store paths;
* builds are easier to audit and compare;
* this reduces "works on my machine" problems in CI/CD and collaboration.



