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

Before you find out the number of CPUs in your Linux computer, re-visit the CPU terminology (modern hybrid processors).

- Socket (physical CPU): The actual processor chip plugged into the motherboard
- Physical core: A real independent processing unit inside the CPU that can execute instructions
- Performance core (P-core): A high-speed core designed for heavy, compute-intensive tasks and supports hyper-threading
- Efficiency core (E-core): A lower-power core designed for background or highly parallel light tasks and does not support hyper-threading
- Logical core (logical CPU): What the operating system sees as a CPU; a schedulable execution unit exposed by physical cores and hyper-threading. These are the “CPUs” you see in: htop, nproc, Nextflow cpus, load average interpretation
- Thread (hardware thread): A single execution slot built on a core; logical CPUs correspond to hardware threads that software can run on

Now, find out the logical CPUs in your Linux computer that you can allocate in your workflow.

```
nproc
```
>20

`nproc` reports the number of logical CPUs available to Linux (i.e., CPU threads seen by the scheduler). This is the correct number to use when setting tool thread counts, configuring Nextflow cpus, and interpreting load average. It may be higher than physical cores in your Linux computer if hyper-threading / Simultaneous Multithreading (SMT) is enabled.

Performance cores provide high single-job speed, while efficiency cores add extra parallel capacity; Linux exposes both as logical CPUs, but they are not equally powerful.

**How much memory (RAM) does my Linux computer have**

```
free -h
```
>125 Gi

This is your absolute scheduling budget.

---
## **Step 2 — Open a monitor**

```
htop
```
This is your main real-time diagnostic panel.

## **Upper Section**

<br /> 
<p align="center">
  <img src="https://github.com/asadprodhan/Bioinformatics_System_Check_Linux/blob/main/htop_upper_ref_codeahoy.png" width="100%">
</p>

<p><strong> Figure 1.</strong> The upper panel of htop summarises real-time system-level resource usage, including per-CPU activity, memory and swap consumption, and overall load average. These metrics allow rapid assessment of whether a workstation is under-utilised, efficiently saturated, or oversubscribed. A healthy system shows load averages close to (but not far above) the number of CPUs, with running tasks roughly matching available cores; an oversubscribed system shows load averages far exceeding CPU count, indicating excessive thread contention rather than true parallelism. </p>
<br />


**The three most important numbers are**

**A) Load average (top right)** 

Load average answers one question:

> How many tasks, on average, wanted the CPU at the same time?

An example:

```
Load average: 3.26 2.64 2.38
```

- Now            → ~ 4 heavy tasks want CPU
- 5 minutes ago  → ~ 3 heavy tasks wanted CPU
- 15 minutes ago → ~ 3 heavy tasks wanted CPU

You interpret it relative to the number of CPUs.

If you Linux computer has 20 CPUs for example, then only 4 out of 20 CPUs are in demand.

> **Key takeaway:** Values much lower than total CPUs mean the system is lightly loaded / under-utilised and far from CPU oversubscription.


**B) Running tasks**

```
Tasks: 80, 180 thr: 4 running
```

Meaning:

The system currently has 
  - 80 processes
  - 180 total threads
  - 4 actively running tasks

The ‘thr’ value shows the total number of software threads that exist on the system, not how many are using the CPU. Only the ‘running’ value tells you how many threads are actively competing for CPU cores.
On a 20-CPU workstation, this indicates the system is lightly loaded and far from CPU oversubscription (most cores are idle).

> The danger sign is “running”, not “thr”.

Examples:

Light load (your case):

```
180 thr; 4 running; 20 CPUs → under-utilised
```

Oversubscribed case:

```
500 thr; 60 running; 20 CPUs → oversubscribed
```

**C) Per-CPU bars (real-time visual cue)**

**Healthy, well-scheduled system:**

  - Most CPU bars are active but not constantly maxed out
  - Smooth, steady movement rather than chaotic spikes
  - Load roughly matches CPU count
  - System remains responsive (terminal, mouse, UI)

**Oversubscribed system:**

  - All CPU bars pinned at or near 100% for long periods
  - Rapid flickering and uneven spikes across cores
  - Load far higher than total CPUs
  - Noticeable lag (slow terminal, delayed UI, noisy fans)
  - Jobs get slower as more are added, not faster

## **Lower Section**

<br /> <p align="center"> <img src="https://github.com/asadprodhan/Bioinformatics_System_Check_Linux/blob/main/htop_lower_ref_codeahoy.png" width="100%" > </p> <p> <strong>Figure 2. </strong> The lower panel of htop displays per-process CPU and memory usage, allowing verification of whether a workflow is driven by one multi-threaded bioinformatics job or by multiple competing processes. A healthy system shows many threads from one dominant program; an oversubscribed system shows many heavy programs running simultaneously. This distinction is a key indicator of correct workflow scheduling versus CPU oversubscription. </p> <br />

**Lower section of htop (process table) — what each field means**

  - PID – Unique process ID assigned by Linux.
  - USER – The user account that owns the process.
  - PRI – Kernel scheduling priority of the process.
  - NI – “Niceness” value (user-space priority hint; −20 highest, 19 lowest).
  - VIRT – Total virtual memory requested by the process.
  - RES – Resident memory actually held in RAM (real usage).
  - SHR – Shared memory used with other processes.
  - S – Process state (R = running, S = sleeping, etc.).
  - CPU% – Percentage of CPU currently being used.
  - MEM% – Percentage of total system RAM being used.
  - TIME+ – Total CPU time consumed since the process started.
  - Command – The command/program that launched the process.

> The lower panel tells you who is running, how much CPU they are using, and how much real memory they actually occupy.

