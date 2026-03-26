## Task 1 — Key Metrics for SRE and System Analysis

### Environment

User: `root`  
OS: Ubuntu 24.04.3 LTS  
Kernel: Linux 6.8.0-87-generic

---

### 1.1 Monitor System Resources

#### Install monitoring tools

```bash
sudo apt install htop sysstat -y
htop --version
iostat -V
````

Output:

```text
WARNING: apt does not have a stable CLI interface. Use with caution in scripts.


WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

htop version:
htop 3.3.0
iostat version:
sysstat version 12.6.1
(C) Sebastien Godard (sysstat <at> orange.fr)
```

#### OS information

```bash
cat /etc/os-release
```

Output:

```text
PRETTY_NAME="Ubuntu 24.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.3 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
```

#### Basic system state

```bash
uptime
free -h
```

Output:

```text
 20:15:52 up 21 days,  9:20,  3 users,  load average: 0.10, 0.09, 0.06
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       1.0Gi       140Mi        28Mi       2.9Gi       2.8Gi
Swap:          4.0Gi        91Mi       3.9Gi
```

#### htop snapshot

```bash
htop -b -n 1
```

Output:

```text
htop: invalid option -- 'b'
```

#### CPU and I/O usage

```bash
iostat -x 1 5
```

Output:

```text
Linux 6.8.0-87-generic (gnow) 	03/26/26 	_x86_64_	(2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.79    0.00    0.77    0.02    0.76   96.66

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
vda              0.23      4.80     0.01   3.96    0.47    21.25    3.57     22.44     1.20  25.12    0.66     6.28    0.01      9.78     0.00   0.00    0.13  1019.77    0.28    0.30    0.00   0.06


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.00    0.00    0.50    0.00    0.50   99.00

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
vda              0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.50    0.00    0.00    0.00    0.50   99.00

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
vda              0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.50    0.00    0.00    0.00    0.00   99.50

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
vda              0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00


avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.00    0.00    0.50    0.00    0.50   99.00

Device            r/s     rkB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wkB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dkB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
vda              0.00      0.00     0.00   0.00    0.00     0.00    2.00    104.00    24.00  92.31    0.50    52.00    0.00      0.00     0.00   0.00    0.00     0.00    2.00    0.00    0.00   0.10
```

#### Top 3 CPU consumers

```bash
ps -eo pid,ppid,comm,cmd,%cpu,%mem --sort=-%cpu | head -n 4
```

Output:

```text
    PID    PPID COMMAND         CMD                         %CPU %MEM
2950343 2950098 ps              ps -eo pid,ppid,comm,cmd,%c 3600  0.1
   1632    1242 amneziawg-go    amneziawg-go awg0            0.4  0.6
    641       1 containerd      /usr/bin/containerd          0.2  1.4
```

#### Top 3 memory consumers

```bash
ps -eo pid,ppid,comm,cmd,%mem,%cpu --sort=-%mem | head -n 4
```

Output:

```text
    PID    PPID COMMAND         CMD                         %MEM %CPU
    918     916 lightdm-gtk-gre /usr/sbin/lightdm-gtk-greet  2.3  0.0
    710       1 dockerd         /usr/bin/dockerd -H fd:// -  2.3  0.2
    709     697 Xorg            /usr/lib/xorg/Xorg -core :0  2.3  0.0
```

#### Raw I/O process statistics

```bash
pidstat -d -p ALL 1 5
```

Output:

```text
Linux 6.8.0-87-generic (gnow) 	03/26/26 	_x86_64_	(2 CPU)

20:15:56      UID       PID   kB_rd/s   kB_wr/s kB_ccwr/s iodelay  Command
20:15:58        0         1      0.00      0.00      0.00       0  systemd
20:15:58        0         2      0.00      0.00      0.00       0  kthreadd
20:15:58        0         3      0.00      0.00      0.00       0  pool_workqueue_release
20:15:58        0         4      0.00      0.00      0.00       0  kworker/R-rcu_g
20:15:58        0         5      0.00      0.00      0.00       0  kworker/R-rcu_p
20:15:58        0         6      0.00      0.00      0.00       0  kworker/R-slub_
20:15:58        0         7      0.00      0.00      0.00       0  kworker/R-netns
20:15:58        0        12      0.00      0.00      0.00       0  kworker/R-mm_pe
20:15:58        0        13      0.00      0.00      0.00       0  rcu_tasks_kthread
20:15:58        0        14      0.00      0.00      0.00       0  rcu_tasks_rude_kthread
20:15:58        0        15      0.00      0.00      0.00       0  rcu_tasks_trace_kthread
20:15:58        0        16      0.00      0.00      0.00       0  ksoftirqd/0
20:15:58        0        17      0.00      0.00      0.00       0  rcu_preempt
20:15:58        0        18      0.00      0.00      0.00       0  migration/0
20:15:58        0        19      0.00      0.00      0.00       0  idle_inject/0
....................
Average:        0   2950342      0.00      0.00      0.00       0  kworker/1:3-events
Average:        0   2950347      0.00      0.00      0.00       0  pidstat
Average:        0   2950348      0.00    229.52      0.00       0  tee
```

#### Top 3 I/O consumers

```bash
awk '$1=="Average:" && $3 ~ /^[0-9]+$/ {sum=$4+$5+$6; printf "%.2f\tPID=%s\tRD=%s\tWR=%s\tCCWR=%s\tCMD=%s\n", sum,$3,$4,$5,$6,$8}' /tmp/lab8_pidstat.txt | sort -nr | head -n 3
```

Output:

```text
229.52	PID=2950348	RD=0.00	WR=229.52	CCWR=0.00	CMD=tee
229.52	PID=2950099	RD=0.00	WR=229.52	CCWR=0.00	CMD=tee
0.66	PID=918	RD=0.00	WR=0.66	CCWR=0.00	CMD=lightdm-gtk-gre
```

### Top 3 resource consumers summary

#### CPU

1. `ps`
2. `amneziawg-go`
3. `containerd`

#### Memory

1. `lightdm-gtk-gre`
2. `dockerd`
3. `Xorg`

#### I/O

1. `tee`
2. `tee`
3. `lightdm-gtk-gre`

---

### 1.2 Disk Space Management

#### Check disk usage

```bash
df -h
```

Output:

```text
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           392M   19M  373M   5% /run
/dev/vda1        38G   29G  7.1G  81% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           392M   24K  392M   1% /run/user/104
overlay          38G   29G  7.1G  81% /var/lib/docker/overlay2/5460a0c440cbec71852f3bb7c669d5e4a557a7f0e092ae7e7e3f6d4e50253a3c/merged
overlay          38G   29G  7.1G  81% /var/lib/docker/overlay2/05f4cd58dbf0dd670f3594c710074d4bbc6034bb859be53a706f30081f93737f/merged
overlay          38G   29G  7.1G  81% /var/lib/docker/overlay2/bd7ddc78aa18e2c64eef05df9b02572e40467fcbd9ab9fb87aac3d73198eb395/merged
overlay          38G   29G  7.1G  81% /var/lib/docker/overlay2/aca57a44bf217375e5d2a7ba7bb8d285c59fd3206ff524661b87b90d0ead0781/merged
overlay          38G   29G  7.1G  81% /var/lib/docker/overlay2/7f8eb131becd42371b25552fe818478ed52db904802283117eca6e29cc1b4283/merged
overlay          38G   29G  7.1G  81% /var/lib/docker/overlay2/650c9da6180e4b9c5fc8302ee4c93e8a3a6d5af9458b409533a461c46bbd75e6/merged
overlay          38G   29G  7.1G  81% /var/lib/docker/overlay2/69dbea64ffd7158c28a88369fb8886fcbe3525fa0c8b51bad2c43647c0568f85/merged
overlay          38G   29G  7.1G  81% /var/lib/docker/overlay2/482bc04350a360b79f871ee7a1b487873b184cafe3b4741d3c9b662c74a8402c/merged
tmpfs           392M   24K  392M   1% /run/user/0
overlay          38G   29G  7.1G  81% /var/lib/docker/overlay2/41d4b67d1ee1a5970d1661c0cc5097f24e7176e1c452ca22e7a4422ba33180d3/merged
```

#### Top 10 largest paths in `/var`

```bash
du -h /var | sort -rh | head -n 10
```

Output:

```text
20G	/var
16G	/var/lib
11G	/var/lib/mysql
4.6G	/var/lib/docker
4.5G	/var/lib/docker/overlay2
3.7G	/var/log
3.6G	/var/lib/mysql/qzaem
3.4G	/var/log/journal/9b37eb03297c4e679754dbc30bbad89a
3.4G	/var/log/journal
624M	/var/lib/docker/overlay2/9gz0fy0szqnctb2rjjn83f1bi/diff
```

#### Top 3 largest files in `/var`

```bash
sudo find /var -type f -exec du -h {} + | sort -rh | head -n 3
```

Output:

```text
3.6G	/var/lib/mysql/qzaem/s_users.ibd
135M	/var/log/btmp.1
132M	/var/lib/docker/overlay2/uhefpypbo4ldtre2ox0t1razw/diff/general_model.keras
```

### Top 3 largest files in `/var`

1. `/var/lib/mysql/qzaem/s_users.ibd` — 3.6G
2. `/var/log/btmp.1` — 135M
3. `/var/lib/docker/overlay2/uhefpypbo4ldtre2ox0t1razw/diff/general_model.keras` — 132M

---

### Analysis

The machine was mostly idle during the observation window. CPU usage stayed very low, and `iostat` showed almost no sustained read/write activity. Most samples had CPU idle around 99%, while disk `%util` also remained near zero. This means there was no CPU bottleneck and no visible I/O pressure during the measurement. The only noticeable disk write spike was very small and short-lived. These patterns indicate that the system was stable and lightly loaded at the time of inspection. 

Memory usage was also stable. The server had 3.8 GiB total RAM, only 1.0 GiB used, and 2.8 GiB available. Swap usage was low at 91 MiB. The top memory consumers were `lightdm-gtk-gre`, `dockerd`, and `Xorg`, but each of them used only 2.3% of memory, so no memory saturation was observed. 

The main issue on this server is disk consumption, not CPU or RAM. The root filesystem was already at 81% usage. Inside `/var`, the biggest consumers were MySQL data, Docker storage, and logs. The single largest file was `/var/lib/mysql/qzaem/s_users.ibd` at 3.6G, which is much larger than any other individual file found during the scan. 

Some top process results should be interpreted carefully. For example, `ps` appeared as the top CPU consumer and `tee` appeared as the top I/O consumer, but both were part of the monitoring pipeline itself. So these are measurement artifacts rather than real production hotspots. The real background services visible in the report were `amneziawg-go`, `containerd`, `dockerd`, `Xorg`, and `lightdm-gtk-gre`. 

### Reflection

Based on these results, I would optimize storage first. The first step would be to inspect the MySQL database and identify which tables are responsible for the 3.6G `.ibd` file. If some data is old or not needed for active workloads, I would archive or delete it. The second step would be Docker cleanup: remove unused images, stopped containers, and dangling layers. The third step would be log retention control, especially under `/var/log/journal`, because logs already occupy several gigabytes. 

I would also add monitoring thresholds for disk usage, because 81% on the root filesystem is not critical yet, but it is already high enough to become a future operational risk. A reasonable approach would be to set a warning alert near 80% and a critical alert near 90%. In contrast, no immediate CPU or memory tuning is required, because both resources were clearly underutilized in this measurement window. 

