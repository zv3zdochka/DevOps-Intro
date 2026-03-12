
## Task 1 — Container Lifecycle & Image Management

### 1.1 Basic container operations

First, I checked which containers already existed in the system.

```sh
docker ps -a
```

```text
CONTAINER ID   IMAGE                            COMMAND                  CREATED        STATUS                   PORTS                                                                                              NAMES
e684eb3f8d7e   amnezia-openvpn-cloak            "dumb-init /opt/amne…"   2 weeks ago    Up 6 days                0.0.0.0:88->443/tcp, [::]:88->443/tcp                                                              amnezia-openvpn-cloak
3e45b3b6293e   amnezia-xray                     "dumb-init /opt/amne…"   2 weeks ago    Up 6 days                0.0.0.0:210->210/tcp, [::]:210->210/tcp                                                            amnezia-xray
5f3d3718e962   amnezia-awg2                     "dumb-init /opt/amne…"   2 weeks ago    Up 6 days                0.0.0.0:431->431/udp, [::]:431->431/udp                                                            amnezia-awg2
f8c59cdb15d5   anonymous_telegram_bot-bot       "python -m bot.main"     4 weeks ago    Up 6 days (unhealthy)                                                                                                       anon_bot
c468c8ecaf13   ghcr.io/wg-easy/wg-easy:latest   "docker-entrypoint.s…"   4 months ago   Up 6 days (healthy)      0.0.0.0:51820->51820/udp, [::]:51820->51820/udp, 0.0.0.0:51821->51821/tcp, [::]:51821->51821/tcp   wg-easy
64d35f552fd0   my-dash-network                  "gunicorn app:server…"   4 months ago   Exited (0) 2 weeks ago                                                                                                      dash-container
```

Then I pulled the Ubuntu image and checked that it appeared locally.

```sh
docker pull ubuntu:latest
```

```text
latest: Pulling from library/ubuntu
01d7766a2e4a: Pull complete
Digest: sha256:d1e2e92c075e5ca139d51a140fff46f84315c0fdce203eab2807c7e495eff4f9
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

```sh
docker images ubuntu
```

```text
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       latest    bbdabce66f1b   4 weeks ago   78.1MB
```

After that I checked the image size in bytes and the number of layers.

```sh
docker image inspect ubuntu:latest --format='Image ID: {{.Id}}
Image size (bytes): {{.Size}}
Layer count: {{len .RootFS.Layers}}'
```

```text
Image ID: sha256:bbdabce66f1b7dde0c081a6b4536d837cd81dd322dd8c99edd68860baf3b2db3
Image size (bytes): 78129634
Layer count: 1
```

Then I ran the `ubuntu_container` container and checked the OS version and process list inside it.

```sh
docker run --name ubuntu_container ubuntu:latest bash -lc 'cat /etc/os-release; echo; ps aux || (apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y procps && ps aux)'
```

```text
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo

USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1 25.0  0.0   4324  3328 ?        Ss   07:07   0:00 bash -lc cat /etc/os-release; echo; ps aux || (apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y procps && ps aux)
root          11  0.0  0.1   7888  4096 ?        R    07:07   0:00 ps aux
```

### 1.2 Image export and dependency analysis

First, I exported the image into a tar file.

```sh
docker save -o ubuntu_image.tar ubuntu:latest
```

The `docker save` command does not print anything if it finishes successfully.

Then I checked the size of the created archive.

```sh
ls -lh ubuntu_image.tar
```

```text
-rw------- 1 root root 77M Mar 12 07:07 ubuntu_image.tar
```

After that I tried to remove the image while the container still existed.

```sh
docker rmi ubuntu:latest
```

```text
Error response from daemon: conflict: unable to remove repository reference "ubuntu:latest" (must force) - container 7685a1ca194e is using its referenced image bbdabce66f1b
```

Then I removed the container and repeated the image removal.

```sh
docker rm ubuntu_container
```

```text
ubuntu_container
```

```sh
docker rmi ubuntu:latest
```

```text
Untagged: ubuntu:latest
Untagged: ubuntu@sha256:d1e2e92c075e5ca139d51a140fff46f84315c0fdce203eab2807c7e495eff4f9
Deleted: sha256:bbdabce66f1b7dde0c081a6b4536d837cd81dd322dd8c99edd68860baf3b2db3
Deleted: sha256:efafae78d70c98626c521c246827389128e7d7ea442db31bc433934647f0c791
```

At the end I checked the container list again.

```sh
docker ps -a
```

```text
CONTAINER ID   IMAGE                            COMMAND                  CREATED        STATUS                   PORTS                                                                                              NAMES
e684eb3f8d7e   amnezia-openvpn-cloak            "dumb-init /opt/amne…"   2 weeks ago    Up 6 days                0.0.0.0:88->443/tcp, [::]:88->443/tcp                                                              amnezia-openvpn-cloak
3e45b3b6293e   amnezia-xray                     "dumb-init /opt/amne…"   2 weeks ago    Up 6 days                0.0.0.0:210->210/tcp, [::]:210->210/tcp                                                            amnezia-xray
5f3d3718e962   amnezia-awg2                     "dumb-init /opt/amne…"   2 weeks ago    Up 6 days                0.0.0.0:431->431/udp, [::]:431->431/udp                                                            amnezia-awg2
f8c59cdb15d5   anonymous_telegram_bot-bot       "python -m bot.main"     4 weeks ago    Up 6 days (unhealthy)                                                                                                       anon_bot
c468c8ecaf13   ghcr.io/wg-easy/wg-easy:latest   "docker-entrypoint.s…"   4 months ago   Up 6 days (healthy)      0.0.0.0:51820->51820/udp, [::]:51820->51820/udp, 0.0.0.0:51821->51821/tcp, [::]:51821->51821/tcp   wg-easy
64d35f552fd0   my-dash-network                  "gunicorn app:server…"   4 months ago   Exited (0) 2 weeks ago                                                                                                      dash-container
```

### Short analysis

The size of `ubuntu:latest` was `78.1 MB`, and the size of `ubuntu_image.tar` was `77 MB`.
The image had `1` layer.

The first attempt to remove the image failed because the `ubuntu_container` container was still linked to this image. Docker does not allow removing an image while there is still a container based on it. First the container must be removed, then the image can be removed.

So the relation here is simple:

* image is a template;
* container is an instance created from that template;
* while the container exists, Docker treats the image as still in use.

As for `ubuntu_image.tar`, it contains the saved image itself: its filesystem, layers, metadata, and tag information. In other words, it is a portable package of the image that can later be loaded back with `docker load`.

## Task 2 — Custom Image Creation & Analysis

### 2.1 Deploying Nginx and changing content

First, I started the `nginx_container` container on port 80.

```sh
docker run -d -p 80:80 --name nginx_container nginx
```

```text
fea72759c896e1df304c6e98b84995b63c839e040e3bb80d001e8b6458d6c32a
```

Then I used `curl` to check that the container was serving the default Nginx page.

```sh
curl http://localhost
```

```text
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy,
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

After that I created my own `index.html` file.

```sh
cat > index.html <<'EOF'
<html>
<head>
<title>The best</title>
</head>
<body>
<h1>website</h1>
</body>
</html>
EOF
```

The file content was:

```sh
cat index.html
```

```text
<html>
<head>
<title>The best</title>
</head>
<body>
<h1>website</h1>
</body>
</html>
```

Then I copied the new file into the container directory from which Nginx serves the website.

```sh
docker cp index.html nginx_container:/usr/share/nginx/html/index.html
```

```text
Successfully copied 2.05kB to nginx_container:/usr/share/nginx/html/index.html
```

Then I checked the page again with `curl`.

```sh
curl http://localhost
```

```text
<html>
<head>
<title>The best</title>
</head>
<body>
<h1>website</h1>
</body>
</html>
```

This confirmed that the content inside the container changed successfully.

### 2.2 Creating a custom image and testing it

Next, I saved the current state of the container into a new image called `my_website:latest`.

```sh
docker commit nginx_container my_website:latest
```

```text
sha256:6cd39b9c52385ce66ec9add46a851d88fbafd0a0eb2f53cb3e6ffda7c9c86a74
```

Checking the image list:

```sh
docker images my_website
```

```text
REPOSITORY   TAG       IMAGE ID       CREATED                  SIZE
my_website   latest    6cd39b9c5238   Less than a second ago   161MB
```

After that I removed the original container.

```sh
docker rm -f nginx_container
```

```text
nginx_container
```

Then I started a new container from my custom image.

```sh
docker run -d -p 80:80 --name my_website_container my_website:latest
```

```text
1837e9071f576a71e2eccc0ed0ed1e676bdccf3bed9396586c9426559dda4f3f
```

A check with `curl` showed that the custom page was still there after recreating the container from the new image.

```sh
curl http://localhost
```

```text
<html>
<head>
<title>The best</title>
</head>
<body>
<h1>website</h1>
</body>
</html>
```

### Filesystem change analysis

Then I checked the changes with `docker diff`.

```sh
docker diff my_website_container
```

```text
C /etc
C /etc/nginx
C /etc/nginx/conf.d
C /etc/nginx/conf.d/default.conf
C /run
C /run/nginx.pid
```

The meanings are:

* `A` — file or directory was added;
* `C` — file or directory was changed;
* `D` — file or directory was deleted.

In my case the output contains only `C`, so Docker detected changed files and directories. These are mostly service files related to Nginx startup inside the container, such as config and runtime files in `/run`.

It is also important that `index.html` is not shown here. This is because when `my_website_container` was started, this file was already part of the `my_website:latest` image created with `docker commit`. So for the new container it was not a new change anymore, but part of the base image.

### Short note on `docker commit` and Dockerfile

`docker commit` is convenient because it lets you quickly create a new image from an already configured container. It is useful for quick tests, debugging, and demos.

But it also has disadvantages:

* the process is not very reproducible;
* you cannot clearly see all steps;
* it is harder to maintain later;
* it is not very convenient for team work and CI/CD.

Dockerfile is better because:

* all steps are written explicitly;
* the build is reproducible;
* the history is easier to understand;
* it is more suitable for real development and deployment.

## Task 3 — Container Networking & Service Discovery

### 3.1 Creating a custom network

First, I removed old containers and the old network if they existed, and then created a new bridge network called `lab_network`.

```sh
docker network create lab_network
```

```text
6cfe0f2b151f48e8b17fe5fb8c87599cc19868c1fdb896d2c1e617a0278f7907
```

After that I checked the Docker network list.

```sh
docker network ls
```

```text
NETWORK ID     NAME                                 DRIVER    SCOPE
46fc0060dcda   amnezia-dns-net                      bridge    local
b2a1bf8af6dc   anonymous_telegram_bot_bot_network   bridge    local
18a683303a8e   bridge                               bridge    local
2aff535c1be6   host                                 host      local
6cfe0f2b151f   lab_network                          bridge    local
0b7b9f2406f0   none                                 null      local
```

Then I started two containers, `container1` and `container2`, inside `lab_network`.

```sh
docker run -dit --network lab_network --name container1 alpine ash
```

```text
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
589002ba0eae: Already exists
Digest: sha256:25109184c71bdad752c8312a8623239686a9a2071e8825f20acb8f2198c3f659
Status: Downloaded newer image for alpine:latest
36b4d2cf3c609fba45b2a17ed6e1325864602ccee0c7eac03dbd1feead3c5e51
```

```sh
docker run -dit --network lab_network --name container2 alpine ash
```

```text
923ee0aae307f9785db53dce697eae7de934ba0615c7fb2ff2e2f861b6cd24f6
```

### 3.2 Testing connectivity and DNS

First, I checked that `container1` could ping `container2` by name.

```sh
docker exec container1 ping -c 3 container2
```

```text
PING container2 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.138 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.239 ms
64 bytes from 172.19.0.3: seq=2 ttl=64 time=0.136 ms

--- container2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.136/0.171/0.239 ms
```

The ping was successful, so the containers inside the network could see each other.

Then I checked the detailed network information.

```sh
docker network inspect lab_network
```

```text
[
    {
        "Name": "lab_network",
        "Id": "6cfe0f2b151f48e8b17fe5fb8c87599cc19868c1fdb896d2c1e617a0278f7907",
        "Created": "2026-03-12T07:18:03.532090877Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "36b4d2cf3c609fba45b2a17ed6e1325864602ccee0c7eac03dbd1feead3c5e51": {
                "Name": "container1",
                "EndpointID": "60faaa89cfb5e634d6f94deefbc6994595deec6caab4221e3e23e0c80711a619",
                "MacAddress": "3a:94:62:8d:ad:46",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            },
            "923ee0aae307f9785db53dce697eae7de934ba0615c7fb2ff2e2f861b6cd24f6": {
                "Name": "container2",
                "EndpointID": "6a04e92cfec6772c7fdf1feda2c69ac89c012468f45aff5c7376e4f6ffc489cf",
                "MacAddress": "de:90:ff:70:3d:a0",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

From this output we can see:

* `container1` got IP `172.19.0.2`
* `container2` got IP `172.19.0.3`

After that I checked DNS name resolution for the container.

```sh
docker exec container1 nslookup container2
```

```text
Server:         127.0.0.11
Address:        127.0.0.11:53

Non-authoritative answer:

Non-authoritative answer:
Name:   container2
Address: 172.19.0.3
```

### Short analysis

Inside a user-defined bridge network, Docker runs an internal DNS service. Because of this, containers can communicate with each other by name, not only by IP address. In my case, `container1` successfully found `container2` by name, and `nslookup` showed that `container2` resolves to `172.19.0.3`.

This is convenient because container IP addresses may change after recreation, while the container name stays a simple and stable way to communicate inside the network.

Compared to the default bridge network, a user-defined bridge has several advantages:

* containers can be connected by name more easily;
* the network is better isolated;
* it is easier to manage which containers belong to it;
* there is no need to manually look up IP addresses or use old linking methods.

So, a separate bridge network makes container communication much easier and clearer.

## Task 4 — Data Persistence with Volumes

### 4.1 Creating a volume and writing data

First, I created a named volume called `app_data`.

```sh
docker volume create app_data
```

```text
app_data
```

After that I checked the list of volumes in the system.

```sh
docker volume ls
```

```text
DRIVER    VOLUME NAME
local     app_data
```

Then I started the `web` container and mounted the `app_data` volume into `/usr/share/nginx/html`.

```sh
docker run -d -p 80:80 -v app_data:/usr/share/nginx/html --name web nginx
```

```text
05a82857faff80d9a0f873732c522d4412ce0b44d4637935ff6449e1cf11bc48
```

After that I created my own `index.html` file with the following content:

```sh
cat > index.html <<'EOF'
<html><body><h1>Persistent Data</h1></body></html>
EOF
```

Checking the file content:

```sh
cat index.html
```

```text
<html><body><h1>Persistent Data</h1></body></html>
```

Then I copied this file into the container, and therefore into the mounted volume.

```sh
docker cp index.html web:/usr/share/nginx/html/index.html
```

```text
Successfully copied 2.05kB to web:/usr/share/nginx/html/index.html
```

After that I checked the page with `curl`.

```sh
curl http://localhost
```

```text
<html><body><h1>Persistent Data</h1></body></html>
```

This confirmed that the new data was written successfully.

### 4.2 Checking persistence after container removal

Then I stopped and removed the `web` container.

```sh
docker stop web
docker rm web
```

```text
web
web
```

After that I started a new container called `web_new`, but with the same `app_data` volume.

```sh
docker run -d -p 80:80 -v app_data:/usr/share/nginx/html --name web_new nginx
```

```text
21817686c5f2f92a799be2e4d4767999383f22ebd992b2020602ea5e200ade7b
```

Then I checked the page again with `curl`.

```sh
curl http://localhost
```

```text
<html><body><h1>Persistent Data</h1></body></html>
```

The content stayed the same even though the original container had already been removed. This shows that the data was stored not inside the container, but in the external volume.

At the end I checked the volume information.

```sh
docker volume inspect app_data
```

```text
[
    {
        "CreatedAt": "2026-03-12T07:22:39Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/app_data/_data",
        "Name": "app_data",
        "Options": null,
        "Scope": "local"
    }
]
```

### Short analysis

The data stayed after container removal because it was stored in the named volume `app_data`, not in the writable layer of the container itself. The container can be recreated, but the volume still exists separately and can be mounted again into a new container.

Data persistence is important in containerized applications because containers themselves are temporary. They can be removed, recreated, or updated, and without a separate storage mechanism the data would be lost.

If we compare the main storage options, the difference is:

* `volume` — separate storage managed by Docker; good for persistent app data, databases, websites, and other services;
* `bind mount` — a direct mount of a specific host folder into a container; useful when you want to work with project files directly from the host machine;
* `container storage` — the writable layer of the container itself; it exists only as long as the container exists and is not suitable for important persistent data.

So, for persistent data in Docker, using a volume is the right approach because it does not disappear when the container is removed.

