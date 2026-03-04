# Lab 5 — Virtualization & System Analysis


# Task 1 — VirtualBox Installation

## Host Operating System

- OS: Windows 11 Home  
- Version: 23H2  
- Build: 22631.6199  

### System Information Screenshot

![Windows and VirtualBox version](screenshots\lab_5\lab_5_1.png)

This screenshot shows the Windows system information window together with the installed VirtualBox version.


## VirtualBox Version

Command used:

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" -v
````

Output:

```
7.2.4r170995
```

### Installation Notes

VirtualBox was installed using the official installer from the VirtualBox website with default settings.
No installation issues occurred during the process.

---

# Task 2 — Ubuntu VM and System Analysis

## 2.1 Virtual Machine Configuration

The Ubuntu virtual machine was created in Oracle VirtualBox with the following configuration:

* Operating System: Ubuntu 24.04.4 LTS
* RAM: 4 GB
* CPU Cores: 4
* Storage: 25 GB (VirtualBox VDI)
* Network Mode: NAT

### Ubuntu VM Screenshot

![Ubuntu Virtual Machine running](screenshots\lab_5\lab_5_2.png)
![Ubuntu Virtual Machine running](screenshots\lab_5\lab_5_3.png)

This screenshot shows the running Ubuntu virtual machine inside VirtualBox.


# 2.2 System Information Discovery

Various Linux command line tools were used to analyze system information inside the Ubuntu virtual machine.

# CPU Details

Tools used:

* `lscpu`
* `/proc/cpuinfo`
* `lshw`

### Command

```bash
lscpu
```

Output:

```
Architecture:                x86_64
  CPU op-mode(s):            32-bit, 64-bit
  Address sizes:             48 bits physical, 48 bits virtual
  Byte Order:                Little Endian
CPU(s):                      4
  On-line CPU(s) list:       0-3
Vendor ID:                   AuthenticAMD
  Model name:                AMD Ryzen 5 4600H with Radeon Graphics
    CPU family:              23
    Model:                   96
    Thread(s) per core:      1
    Core(s) per socket:      4
    Socket(s):               1
    Stepping:                1
    BogoMIPS:                5988.74
    Flags:                   fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pg
                             e mca cmov pat pse36 clflush mmx fxsr sse sse2 ht s
                             yscall nx mmxext fxsr_opt rdtscp lm constant_tsc re
                             p_good nopl xtopology nonstop_tsc cpuid extd_apicid
                              tsc_known_freq pni pclmulqdq ssse3 fma cx16 sse4_1
                              sse4_2 x2apic movbe popcnt aes xsave avx f16c rdra
                             nd hypervisor lahf_lm cmp_legacy cr8_legacy abm sse
                             4a misalignsse 3dnowprefetch ssbd vmmcall fsgsbase 
                             bmi1 avx2 bmi2 rdseed adx clflushopt sha_ni arat
Virtualization features:     
  Hypervisor vendor:         KVM
  Virtualization type:       full
NUMA:                        
  NUMA node(s):              1
  NUMA node0 CPU(s):         0-3
```

---

### Command

```bash
cat /proc/cpuinfo | head -n 20
```

Output:

```
processor	: 0
vendor_id	: AuthenticAMD
cpu family	: 23
model		: 96
model name	: AMD Ryzen 5 4600H with Radeon Graphics
stepping	: 1
microcode	: 0x8600104
cpu MHz		: 2994.374
cache size	: 512 KB
physical id	: 0
siblings	: 4
core id		: 0
cpu cores	: 4
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 16
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid extd_apicid tsc_known_freq pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt aes xsave avx f16c rdrand hypervisor lahf_lm cmp_legacy cr8_legacy abm sse4a misalignsse 3dnowprefetch ssbd vmmcall fsgsbase bmi1 avx2 bmi2 rdseed adx clflushopt sha_ni arat
```

---

### Command

```bash
sudo lshw -class processor
```

Output:

```
*-cpu
     product: AMD Ryzen 5 4600H with Radeon Graphics
     vendor: Advanced Micro Devices [AMD]
     physical id: 2
     bus info: cpu@0
     version: 23.96.1
     width: 64 bits
     configuration: microcode=140509444
```

---

# Memory Information

Tools used:

* `free`
* `/proc/meminfo`

### Command

```bash
free -h
```

Output:

```
               total        used        free      shared  buff/cache   available
Mem:           4.4Gi       1.7Gi       119Mi       482Mi       3.2Gi       2.7Gi
Swap:             0B          0B          0B
```

---

### Command

```bash
cat /proc/meminfo | head -n 15
```

Output:

```
MemTotal:        4611192 kB
MemFree:          122128 kB
MemAvailable:    2780836 kB
Buffers:            5540 kB
Cached:          3308868 kB
SwapCached:            0 kB
Active:           870452 kB
Inactive:        3228624 kB
Active(anon):     614456 kB
Inactive(anon):   665792 kB
Active(file):     255996 kB
Inactive(file):  2562832 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:             0 kB
```

---

# Network Configuration

Tools used:

* `ip`
* `resolvectl`

### Command

```bash
ip a
```

Output:

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536
inet 127.0.0.1/8 scope host lo

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP>
inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic
```

---

### Command

```bash
ip r
```

Output:

```
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
```

---

### Command

```bash
resolvectl status
```

Output:

```
Current DNS Server: 1.1.1.1
DNS Servers: 1.1.1.1 1.0.0.1 192.168.88.1
```

---

# Storage Information

Tools used:

* `lsblk`
* `df`
* `fdisk`

### Command

```bash
lsblk -f
```

Output shows the virtual disk `VBOX HARDDISK` and the Linux filesystem created during installation.

---

### Command

```bash
df -hT
```

Output:

```
Filesystem     Type     Size  Used Avail Use% Mounted on
/dev/sr0       iso9660  6.2G  6.2G     0 100% /cdrom
/cow           overlay  2.2G  223M  2.0G  10% /
```

---

### Command

```bash
sudo fdisk -l
```

Output shows the VirtualBox disk:

```
Disk /dev/sda: 25 GiB
Disk model: VBOX HARDDISK
```

---

# Operating System Information

Tools used:

* `lsb_release`
* `/etc/os-release`
* `uname`
* `hostnamectl`

### Commands

```bash
lsb_release -a
```

Output:

```
Distributor ID: Ubuntu
Description: Ubuntu 24.04.4 LTS
Release: 24.04
Codename: noble
```

---

```bash
cat /etc/os-release
```

Output:

```
PRETTY_NAME="Ubuntu 24.04.4 LTS"
VERSION_ID="24.04"
```

---

```bash
uname -a
```

Output:

```
Linux ubuntu 6.17.0-14-generic x86_64 GNU/Linux
```

---

```bash
hostnamectl
```

Output:

```
Operating System: Ubuntu 24.04.4 LTS
Kernel: Linux 6.17.0-14-generic
Architecture: x86-64
Virtualization: oracle
Hardware Model: VirtualBox
```

---

# Virtualization Detection

Tools used:

* `systemd-detect-virt`
* `dmidecode`

### Commands

```bash
systemd-detect-virt
```

Output:

```
oracle
```

---

```bash
sudo dmidecode -s system-product-name
```

Output:

```
VirtualBox
```

---

```bash
sudo dmidecode -s system-manufacturer
```

Output:

```
innotek GmbH
```

---

# Reflection

Several command line tools were used to analyze the system configuration inside the virtual machine.

The most useful tools were:

* **lscpu** — provides a detailed overview of CPU architecture and virtualization features.
* **free -h** — quickly shows total and available memory.
* **ip a** — useful for identifying network interfaces and IP addresses.
* **df -h** — helps to analyze disk usage in a readable format.
* **systemd-detect-virt** — directly confirms that the system is running inside a virtual machine.

These tools are commonly used in Linux system administration and DevOps environments to quickly gather hardware and system information.


