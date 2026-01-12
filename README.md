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
>20

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

<br /> 
<p align="center">
  <img src="https://github.com/asadprodhan/Bioinformatics_System_Check_Linux/blob/main/htop_upper_ref_codeahoy.png" width="100%">
</p>

<p><strong>Figure 1. Upper section of the htop output screen.</strong> </p>
<br />


**The three most important numbers are**

Load average (top right). Load average answers one question: 

> How many tasks, on average, wanted the CPU at the same time?

An example:

```
Load average: 3.26 2.64 2.38
```

- Now            → ~4 heavy tasks want CPU
- 5 minutes ago  → ~3 heavy tasks wanted CPU
- 15 minutes ago → ~3 heavy tasks wanted CPU

You interpret it relative to the number of CPUs.

If you Linux computer has 20 CPUs for example, then only 4 out of 20 CPUs are in demand.

> **Key takeaway:** Values much lower than total CPUs mean the system is lightly loaded / under-utilised and far from CPU oversubscription.



<br /> <p align="center"> <img src="https://github.com/asadprodhan/Bioinformatics_System_Check_Linux/blob/main/htop_lower_ref_codeahoy.png" width="100%" > </p> <p> <strong>Figure 1. htop output screen- lower section. </p> <br />

<br />


The htop output screen is explained below.



