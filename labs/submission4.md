# Lab 4 — Operating System & Networking

## Task 1 — Operating System Analysis


```bash
systemd-analyze
```

```
Startup finished in 3.072s (kernel) + 11.640s (userspace) = 14.713s
graphical.target reached after 9.486s in userspace.
```

```bash
systemd-analyze blame | head -n 25
```

```
35.313s apt-daily.service
31.193s apt-daily-upgrade.service
2.733s docker.service
2.128s systemd-journal-flush.service
2.123s cloud-init-local.service
2.029s logrotate.service
1.929s cloud-final.service
1.751s cloud-config.service
1.420s dev-vda1.device
1.247s systemd-networkd-wait-online.service
997ms certbot.service
996ms tuned.service
985ms cloud-init.service
921ms containerd.service
838ms gpu-manager.service
751ms accounts-daemon.service
635ms rsyslog.service
563ms ufw.service
405ms user@104.service
396ms man-db.service
396ms user@0.service
392ms systemd-udev-trigger.service
345ms mysql.service
344ms polkit.service
334ms fstrim.service
```

```bash
uptime
```

```
14:25:31 up 86 days, 8:06, 3 users, load average: 0.18, 0.11, 0.06
```

```bash
w
```

```
USER   TTY   FROM        LOGIN@   IDLE WHAT
root         172.17.0.XXX 14:24   86days sshd: root@pts/0
lightdm      -            28Nov25 86days lightdm --session-child
```

* Total boot time is **14.7 seconds**, indicating fast system initialization.
* The largest boot delays are caused by automatic package update services (`apt-daily`, `apt-daily-upgrade`).
* System load averages are very low (<0.2), showing minimal CPU utilization.
* The system has been running continuously for **86 days**, indicating stable uptime.


```bash
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head -n 6
```

```
PID     PPID CMD                         %MEM %CPU
2771591 ...  python3.12                  4.7  0.0
2771590 ...  python3.12                  4.7  0.0
2771588 ...  python3.12                  4.6  0.0
3605418 ...  python3.12                  4.5  0.0
241        1 systemd-journald            3.6  0.0
```

```bash
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head -n 6
```

```
PID     PPID CMD                %MEM %CPU
3655872 547  sshd: root [priv]   0.2 10.1
3655873 ... sshd: root [net]     0.1 0.9
```

### Observations

* Multiple Python processes consume the largest portion of memory.
* SSH daemon temporarily shows the highest CPU usage due to active remote session handling.
* Background system processes maintain low CPU utilization.

---

```bash
systemctl list-dependencies | head -n 80
```

output truncated

```
default.target
├─accounts-daemon.service
├─lightdm.service
└─multi-user.target
 ├─docker.service
 ├─nginx.service
 ├─openvpn.service
 ├─ssh.service
 ├─systemd-networkd.service
 └─ufw.service
```

```bash
systemctl list-dependencies multi-user.target | head -n 120
```

output truncated

```
multi-user.target
├─docker.service
├─nginx.service
├─openvpn.service
├─ssh.service
├─rsyslog.service
├─systemd-networkd.service
└─timers.target
```

### Observations

* The system follows a standard Linux service hierarchy managed by `systemd`.
* Core infrastructure services include Docker, SSH, networking, logging, and firewall services.
* One dependency (`mysql.service`) appears inactive or failed, which may indicate unused or misconfigured service.

---

```bash
who -a
```

```
system boot 2025-11-28
run-level 5 2025-11-28
root pts/0 172.17.0.XXX still logged in
```

```bash
last -n 5
```

```
root pts/0 172.17.0.XXX still logged in
root pts/0 188.130.155.XXX Mon Feb 16
root pts/0 172.17.0.XXX Mon Feb 9
root pts/0 188.130.155.XXX Fri Feb 6
root pts/1 172.17.0.XXX Sun Feb 1
```

### Observations

* Remote access is performed via SSH sessions.
* Login history confirms repeated administrative access.
* No abnormal login patterns were observed.

---

```bash
free -h
```

```
Mem:   3.8Gi total, 1.3Gi used, 199Mi free, 2.6Gi buff/cache
Swap:  4.0Gi total, 194Mi used
```

```bash
cat /proc/meminfo | grep -e MemTotal -e SwapTotal -e MemAvailable
```

```
MemTotal:     4009884 kB
MemAvailable: 2608444 kB
SwapTotal:    4194300 kB
```

### Observations

* Significant portion of RAM is used as filesystem cache.
* Available memory (~2.5 GB) indicates no memory pressure.
* Swap usage is minimal, confirming efficient memory management.

---

## Resource Utilization Patterns

* CPU usage remains consistently low.
* Memory consumption is dominated by application-level Python processes.
* System services operate efficiently with stable long-term uptime.
* Cached memory usage improves disk I/O performance.

---

