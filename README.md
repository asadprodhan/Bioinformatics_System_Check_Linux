<h1 align="center">How to Tell if a Bioinformatics Workstation is CPU-Oversubscribed</h1>


<h3 align="center">Asad Prodhan<sup>*</sup></h3>


<div align="center"><b> DPIRD Diagnostics and Laboratory Services </b></div>

<div align="center"><b> Department of Primary Industries and Regional Development </b></div>


<div align="center"><b> 3 Baron-Hay Court, South Perth, WA 6151, Australia </b></div>


<div align="center"><b> <sup>*</sup>Correspondence: Asad.Prodhan@dpird.wa.gov.au </b></div>


<br />


<p align="center">
  <a href="https://github.com/asadprodhan/blastn/tree/main#GPL-3.0-1-ov-file"><img src="https://img.shields.io/badge/License-GPL%203.0-yellow.svg" alt="License GPL 3.0" style="display: inline-block;"></a>
  <a href="https://orcid.org/0000-0002-1320-3486"><img src="https://img.shields.io/badge/ORCID-green?style=flat-square&logo=ORCID&logoColor=white" alt="ORCID" style="display: inline-block;"></a>
</p>


<br />

## **Step 1 — Know your machine**

**How many CPUs does my Linux computer have?**

```
nproc
```

> 20

**How much memory (RAM) does my Linux computer have**

```
free -h
```

> 125 Gi

 
---

## Why this repo exists

Bioinformatics pipelines often fail or slow down due to:

- **Out-of-memory (OOM)** kills (especially HUMAnN, Kraken2/Bracken, DIAMOND, ML jobs)
- **Disk bottlenecks** (scratch vs home, network storage, small-file storms)
- **Runaway processes** (Nextflow / Java / container spawns)
- **Overloaded nodes** (especially on shared machines or HPC login nodes)

This repo provides **copy-paste-ready commands** and optional scripts to triage issues fast.

---

## Quick Start (copy/paste)

### 1) 10-second health snapshot
```bash
uptime
free -h
df -h
ps -eo pid,user,cmd,%cpu,%mem --sort=-%cpu | head



2) Best interactive monitor
htop

Install recommended tools (optional)
Ubuntu / Debian
sudo apt update
sudo apt install -y htop sysstat iotop lm-sensors pciutils psmisc

RHEL / CentOS / Rocky
sudo yum install -y htop sysstat iotop lm_sensors pciutils psmisc


sysstat provides iostat + mpstat
lm-sensors provides sensors (temperatures)

What to check (bioinformatics-focused checklist)
A) CPU & load (are you CPU bound?)
uptime
lscpu
mpstat -P ALL 1


Interpretation tips:

Load average much higher than total cores can indicate CPU contention or heavy I/O wait.

mpstat shows per-core utilization and %iowait.

B) Memory & swap (most common pipeline killer)
free -h
swapon --show
cat /proc/meminfo | head -n 30


Interpretation tips:

Low MemAvailable + rising swap often predicts crashes or severe slowdown.

If swap is heavily used, performance typically drops hard.

C) Top resource-hungry processes (find the culprit)

Top CPU:

ps -eo pid,user,ppid,cmd,%cpu,%mem --sort=-%cpu | head -n 20


Top memory:

ps -eo pid,user,ppid,cmd,%cpu,%mem --sort=-%mem | head -n 20


Process tree (great for Nextflow and container spawns):

pstree -ap | less

D) Disk space (silent killer)
df -h
lsblk


Largest folders (run inside your project/work dir):

du -sh * 2>/dev/null | sort -h


Interpretation tips:

If scratch/tmp fills up: tools may fail with cryptic errors.

Millions of small files can slow network filesystems badly.

E) Disk I/O (when CPU looks “idle” but jobs crawl)
iostat -xz 1


Interpretation tips:

High %util near 100% suggests a saturated disk.

High await indicates I/O latency (common on shared/network storage).

F) Live I/O per process (who is hammering the disk?)
sudo iotop -o


Interpretation tips:

Useful when Nextflow staging or container layers cause heavy read/write.

G) Network (if databases live on remote storage)
ip -br a
ss -tupn | head


Optional quick reachability test:

ping -c 2 google.com

H) GPU (if doing ML / deep learning)

NVIDIA:

nvidia-smi

I) Kernel messages (did the system kill your job?)
dmesg -T | tail -n 80


Look for:

Out of memory: Killed process ...

filesystem errors

hardware / driver issues

On some systems, dmesg access is restricted; use sudo.

One-liner: Bioinformatics “pre-run” sanity check ✅
echo "===== SYSTEM ====="; hostname; date; uptime; \
echo -e "\n===== CPU ====="; lscpu | egrep 'Model name|Socket|Core|Thread|CPU\(s\)'; \
echo -e "\n===== MEMORY ====="; free -h; swapon --show; \
echo -e "\n===== DISK ====="; df -h; \
echo -e "\n===== TOP CPU PROCESSES ====="; ps -eo pid,user,cmd,%cpu,%mem --sort=-%cpu | head -n 15; \
echo -e "\n===== TOP MEM PROCESSES ====="; ps -eo pid,user,cmd,%cpu,%mem --sort=-%mem | head -n 15

Suggested repo structure (optional)
