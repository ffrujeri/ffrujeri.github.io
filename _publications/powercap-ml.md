---
title: "Prediction-Based Power Oversubscription in Cloud Platforms"
collection: publications
permalink: /publication/powercap-ml
excerpt: "<img src='/images/powercap.png' width='480' style='display:block; margin:auto;'><br/>Datacenter designers rely on conservative estimates of IT equipment power draw to provision resources. This leaves resources underutilized and requires more datacenters to be built. Prior work has used power capping to shave the rare power peaks and add more servers to the datacenter, thereby oversubscribing its resources and lowering capital costs. This works well when the workloads and their server placements are known. Unfortunately, these factors are unknown in public clouds, forcing providers to limit the oversubscription so that performance is never impacted.

In this paper, we argue that providers can use predictions of workload performance criticality and virtual machine (VM) resource utilization to increase oversubscription. This poses many challenges, such as identifying the performance-critical workloads from black-box VMs, creating support for criticality-aware power management, and increasing oversubscription while limiting the impact of capping. We address these challenges for the hardware and software infrastructures of Microsoft Azure. The results show that we enable a 2x increase in oversubscription with minimum impact to critical workloads."
date: 2020-10-01
venue: 'Proceedings of the USENIX Annual Technical Conference (ATC)'
citation: Alok Kumbhare, Reza Azimi, Ioannis Manousakis, Anand Bonde, Felipe Frujeri, Nithish Mahalingam,
Pulkit A. Misra, Seyyed Ahmad Javadi, Bianca Schroeder, Marcus Fontoura, and Ricardo Bianchini
---
Datacenter designers rely on conservative estimates of IT equipment power draw to provision resources. This leaves resources underutilized and requires more datacenters to be built. Prior work has used power capping to shave the rare power peaks and add more servers to the datacenter, thereby oversubscribing its resources and lowering capital costs. This works well when the workloads and their server placements are known. Unfortunately, these factors are unknown in public clouds, forcing providers to limit the oversubscription so that performance is never impacted.

In this paper, we argue that providers can use predictions of workload performance criticality and virtual machine (VM) resource utilization to increase oversubscription. This poses many challenges, such as identifying the performance-critical workloads from black-box VMs, creating support for criticality-aware power management, and increasing oversubscription while limiting the impact of capping. We address these challenges for the hardware and software infrastructures of Microsoft Azure. The results show that we enable a 2x increase in oversubscription with minimum impact to critical workloads.

[Paper](https://www.microsoft.com/en-us/research/uploads/prod/2020/10/Per-VM-Capping-ATC21.pdf)

