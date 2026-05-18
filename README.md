# Kelvin2 HPC Cluster Detailed Specification

## 1. System & Controller Configuration
* **Cluster Name**: kelvin2.alces.network.
* **Control Node**: The central Slurm controller runs on `infra04`.
* **Network Ports**: Slurmctld communicates on port 6817, and Slurmd uses port 6819.
* **Authentication**: Managed via `auth/munge`.
* **Timeouts**: The Slurmctld and Slurmd timeout limit is 300 seconds. Message timeouts are 20 seconds, and the unkillable step timeout is set to 180 seconds.
* **Accounting & Resource Tracking**: Accounting is handled by `slurmdbd` (hosted on `infra04`), tracking general resources (GRES) like GPUs, associations, limits, and QOS. Job and task resources are tracked using Linux `cgroup` plugins. Energy gathering is performed via IPMI every 30 seconds.
* **Memory Limits**: The default memory per CPU core is set to 1024 MB.

## 2. Scheduler & Priority Multifactor Rules
The cluster uses the `sched/backfill` scheduler combined with a multi-factor job priority plugin that includes decay. The priority weights dictate how jobs are ranked in the queue:
* **Fairshare Weight**: 10,000 (Strongest factor, prioritising balanced cluster usage).
* **QOS Weight**: 4,000.
* **Age Weight**: 1,000 (Job priority increases as it waits in the queue, with a decay half-life of 7 days).
* **Job Size Weight**: 1,000.

## 3. Node Architecture & Topology
The cluster utilises a `topology/tree` structure. Nodes are assigned specific "Weights" to guide the scheduler on allocation preference (lower weight nodes are allocated first).

### Standard Compute Nodes
* **Standard (Weight 16)**: Nodes feature 128 CPUs, structured as 8 Sockets, 16 Cores per socket, and 1 Thread per core. 
* **Memory Tiers**: 
    * ~772 GB (e.g., node[101-141]).
    * ~966 GB (node[183-185]).
    * ~1.03 TB (e.g., node[161-182]).
    * ~2.06 TB (node[210-213]).

### High-Memory (SMP) Nodes
* **smp[106-107] (Weight 30)**: 256 CPUs (8 Sockets, 16 Cores, 2 Threads/core) with ~2.06 TB RAM.
* **smp[108-113] (Weight 30)**: 128 CPUs (8 Sockets, 16 Cores, 1 Thread/core) with ~2.06 TB RAM.

### GPU Accelerator Nodes
* **NVIDIA V100 (gpu[103-110])**: Weight 50. 48 CPUs (2 Sockets, 24 Cores, 1 Thread) with ~514 GB RAM and 4x V100 GPUs.
* **NVIDIA A100 (gpu[111-113])**: Weight 100. 128 CPUs (8 Sockets, 16 Cores, 1 Thread) with ~1.03 TB RAM and 4x A100 GPUs.
* **NVIDIA A100 MIG (gpu114)**: Weight 75. 128 CPUs (8 Sockets, 16 Cores, 1 Thread) with Multi-Instance GPUs (4x 3g.40gb, 8x 2g.20gb) and ~1.03 TB RAM.
* **NVIDIA H100 (gpu121)**: Weight 150. 96 CPUs (2 Sockets, 48 Cores, 1 Thread) with ~1.03 TB RAM and 4x H100 GPUs.
* **Intel i1100 (gpu122)**: Weight 200. 64 CPUs (2 Sockets, 32 Cores, 1 Thread) with ~514 GB RAM and 4x i1100 GPUs.
* **AMD MI300x (gpu123)**: Weight 250. 104 CPUs (2 Sockets, 52 Cores, 1 Thread) with ~2.06 TB RAM and 8x MI300x GPUs.

## 4. Partition Policies & Allowances

### Standard Job Queues
| Partition Name | Priority Tier | Default Time | Max Time | Allowed Groups | QOS |
|---|---|---|---|---|---|
| **k2-hipri** | 3000 | 3 hours | 3 hours | qub, ulster, siteadmins | nodeslimits |
| **k2-medpri** | 2000 | 1 day | 1 day | qub, ulster, siteadmins | nodeslimits |
| **k2-lowpri** | 1000 | 3 days | UNLIMITED | qub, ulster, siteadmins | nodeslimits |

### GPU & Interactive Queues
* **Dedicated GPU Queues (v100, a100, a100mig, h100, intel, amd)**: Priority Tier 1000, Default Time 12 hours, Max Time 3 days, restricted by `gpulimits` QOS.
* **k2-gpu-interactive**: Priority Tier 1000, Default Time 0, Max Time 3 hours. Accessible by `siteadmins, qub, ulster, epsrc` under `gpuintlimits`.

### EPSRC Dedicated Queues
* **k2-epsrc**: Priority Tier 5000, Default Time 7 days, Max Time UNLIMITED.
* **EPSRC GPU Queues**: Tier 5000 (except H100, which is Tier 1000), Default Time 12 hours, Max Time 3 days.
* **k2-epsrc-himem**: Tier 5000, Default Time 12 hours, Max Time 3 days.

### Specialized Projects & Testing
* **k2-sandbox**: Max Time 30 minutes, Tier 1000.
* **k2-hysafer, k2-math-physics, k2-living-labs, k2-fmipr, k2-bioinf, k2-gen17**: All share Priority Tier 5000 with `UNLIMITED` Max Times for their specific dedicated groups.
* **k2-math-physics-debug**: Max Time 1 hour.
* **k2-amd-xseries**: Max Time 3 days.
