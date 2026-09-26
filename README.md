# Isaac Sim on MacBook — Cloud GPU Setup

> **Created by Tanay**  
> A practical, reproducible setup for learning and developing with NVIDIA Isaac Sim and Isaac Lab from a MacBook using a remote NVIDIA GPU.
![Personal Website](https://tanayp.vercel.app)
![LinkedIn](https://LinkedIn.com/in/tanaypadar)

![Platform](https://img.shields.io/badge/Platform-macOS-black)
![GPU](https://img.shields.io/badge/GPU-NVIDIA%20L40S-76B900)
![Simulator](https://img.shields.io/badge/Simulator-Isaac%20Sim-76B900)
![Framework](https://img.shields.io/badge/Framework-Isaac%20Lab-76B900)
![IDE](https://img.shields.io/badge/IDE-Cursor-7C3AED)

---

## 👋 About This Repository

I wanted to learn modern robotics simulation and robot learning using **NVIDIA Isaac Sim / Isaac Lab**, but my primary computer is a **MacBook**.

Since Isaac Sim is designed for NVIDIA GPU environments, instead of buying a local NVIDIA workstation, I built a remote development setup:

```text
MacBook
   │
   │ Cursor + SSH
   ▼
NVIDIA Brev
   │
   ▼
AWS
   │
   ▼
NVIDIA L40S — 48 GB VRAM
   │
   ▼
Docker
   ├── Isaac Sim
   └── Isaac Lab
```

This repository documents **exactly how I approached the setup, what I tried, what failed, what I avoided, and the final workflow I use.**

---

## 🎯 Goal

The goal is not simply to install Isaac Sim.

I want to use the environment to study:

- Robot simulation
- Isaac Sim
- Isaac Lab
- Reinforcement learning
- Imitation learning
- Robot policies
- Simulation-to-real workflows
- Modern Physical AI
- Robot learning research

The cloud GPU is therefore treated as **compute infrastructure**, while my MacBook remains my development machine.

---

## 🧑‍💻 My Development Stack

| Layer | Choice |
|---|---|
| Laptop | MacBook |
| IDE | **Cursor** |
| Remote GPU management | **NVIDIA Brev CLI** |
| Cloud | **AWS through Brev** |
| GPU | **NVIDIA L40S 48 GB** |
| OS | Linux |
| Containers | Docker / Docker Compose |
| Simulator | **NVIDIA Isaac Sim** |
| Robot learning | **NVIDIA Isaac Lab** |
| Remote connection | SSH |
| Version control | Git + GitHub |
| Project workspace | `/home/ubuntu/workspace` |

---

# 🏗️ Architecture

```text
                         INTERNET
                            │
                            ▼
                    ┌───────────────┐
                    │ NVIDIA Brev   │
                    └───────┬───────┘
                            │
                            ▼
                    ┌───────────────┐
                    │ AWS GPU VM    │
                    │               │
                    │ NVIDIA L40S   │
                    │ 48 GB VRAM    │
                    └───────┬───────┘
                            │
                          Docker
                            │
             ┌──────────────┴──────────────┐
             │                             │
      ┌──────▼───────┐              ┌──────▼───────┐
      │ Isaac Lab    │              │ Isaac Sim    │
      │ RL / Policy  │              │ Simulation   │
      │ Learning     │              │ Rendering    │
      └──────────────┘              └──────────────┘
                            ▲
                            │ SSH
                            │
                    ┌───────┴────────┐
                    │     Cursor     │
                    │    MacBook     │
                    └────────────────┘
```

---

# 🚀 Why Cloud Instead of Local?

Isaac Sim is GPU-intensive and its supported workstation platforms are Windows/Linux with NVIDIA GPU hardware.

A MacBook therefore works very well as the **development interface**, but the simulation should run on a remote NVIDIA GPU.

The basic idea is:

```text
MacBook = Development
Cloud GPU = Compute
GitHub = Source of Truth
```

This also means I can use the same MacBook for normal development while renting expensive GPU compute only when needed.

---

# 🟢 Why NVIDIA L40S?

For this workflow, I selected an **NVIDIA L40S with 48 GB VRAM**.

It gives a strong balance of:

- 48 GB VRAM
- RTX capabilities
- RT cores
- Tensor cores
- CUDA
- Isaac Sim compatibility
- enough memory for many Isaac Lab experiments

There is no universal "best GPU." GPU choice depends on the workload, VRAM requirements, provider availability, networking and cost.

For my current goal of learning Isaac Sim/Isaac Lab and experimenting with robot learning, **L40S is my chosen baseline.**

---

# ☁️ Why NVIDIA Brev?

I could manually configure an AWS EC2 GPU instance, but that adds a lot of infrastructure work:

```text
AWS
 ↓
EC2
 ↓
Drivers
 ↓
CUDA
 ↓
Docker
 ↓
NVIDIA Container Toolkit
 ↓
SSH
 ↓
Networking
 ↓
Isaac Sim
```

Brev simplifies this:

```text
Brev
 ↓
Choose GPU
 ↓
Create instance
 ↓
SSH
 ↓
Cursor
 ↓
Isaac Sim
```

Brev also manages SSH keys, configuration and instance IP changes.

Official docs:

- [Brev Documentation](https://docs.nvidia.com/brev/)
- [Brev Quickstart](https://docs.nvidia.com/brev/getting-started/quickstart)
- [Brev CLI](https://docs.nvidia.com/brev/cli/cli-overview)

---

# ☁️ Why AWS?

I evaluated multiple cloud GPU options.

One important discovery was that **the exact cloud provider matters for the deployment method**.

NVIDIA's official Isaac Launchable documentation currently identifies AWS as tested and notes incompatibility with Crusoe instances for that Launchable.

Therefore I selected:

```text
Brev
  ↓
AWS
  ↓
L40S
```

### Lesson

> Do not choose a GPU provider only because it has the cheapest hourly price. First verify that the exact Isaac Sim / Isaac Lab deployment you intend to use supports it.

---

# 🧪 What I Tried

## RunPod

I initially experimented with RunPod.

The GPU itself was capable, but remote Isaac Sim visualization introduced networking complexity around:

- WebRTC
- TCP/UDP ports
- public IPs
- host networking
- streaming clients

RunPod can work, but for this learning setup I preferred the more integrated Brev workflow.

---

## Browser VS Code

I also deployed NVIDIA's official Isaac Launchable.

The Docker environment successfully built and started.

The Launchable includes:

- Isaac Sim
- Isaac Lab
- VS Code
- nginx
- web viewer
- streaming infrastructure

However, getting the browser IDE exposed cleanly through Brev became unnecessarily awkward.

So I changed the development architecture to:

```text
Cursor on Mac
      ↓
SSH
      ↓
Brev GPU
```

while keeping Isaac Sim/Isaac Lab on the GPU.

This is the workflow documented in the setup guide:

**[Detailed Setup Guide → `ISAAC_SIM_MAC_SETUP.md`](./ISAAC_SIM_MAC_SETUP.md)**

---

# 📦 NVIDIA Isaac Launchable

Official repository:

[isaac-sim/isaac-launchable](https://github.com/isaac-sim/isaac-launchable)

Clone:

```bash
cd /home/ubuntu

git clone https://github.com/isaac-sim/isaac-launchable

cd isaac-launchable/isaac-lab
```

Start:

```bash
docker compose up -d
```

Verify:

```bash
docker ps
```

Expected containers include:

```text
isaac-lab-nginx
isaac-lab-vscode
isaac-lab-web-viewer
```

---

# 🍎 MacBook Setup

## Install Brev CLI

```bash
brew install brevdev/homebrew-brev/brev
```

Verify:

```bash
brev --version
```

Authenticate:

```bash
brev login
```

Sync web-created instances:

```bash
brev refresh
```

List instances:

```bash
brev ls
```

---

# 🖥️ Cursor Setup

Install the Cursor command-line launcher from Cursor:

```text
⌘ + Shift + P
→ Install 'cursor' command in PATH
```

Verify:

```bash
which cursor
```

Expected:

```text
/usr/local/bin/cursor
```

Then connect:

```bash
brev start shiny-violet-gayal
brev refresh

brev open shiny-violet-gayal cursor
```

Or directly open the project workspace:

```bash
brev open shiny-violet-gayal cursor   --dir /home/ubuntu/workspace/isaac-projects
```

---

# 📁 Project Organization

I keep my own projects separate from the Isaac Sim installation.

```text
/home/ubuntu/workspace/
│
├── isaac-projects/
│   ├── 01-isaac-basics/
│   ├── 02-rl/
│   ├── 03-robot-learning/
│   └── 04-policy-experiments/
│
└── datasets/
```

This separation is important.

```text
Isaac Sim installation
        ≠
My research/projects
```

My projects are version-controlled with Git and pushed to GitHub.

---

# 🤖 Isaac Sim vs Isaac Lab

### Isaac Sim

The simulation platform:

```text
Physics
Rendering
Robots
Sensors
Scenes
Simulation
```

### Isaac Lab

The robot-learning framework built on Isaac Sim:

```text
Reinforcement Learning
Imitation Learning
Robot Environments
Parallel Simulation
Policy Training
Evaluation
```

Conceptually:

```text
Isaac Sim
    ↓
Simulation
    ↓
Isaac Lab
    ↓
Robot Learning
    ↓
Policy
```

Official Isaac Sim:

https://github.com/isaac-sim/IsaacSim

Official Isaac Lab:

https://github.com/isaac-sim/IsaacLab

---

# 🧠 Headless Training vs Visualization

One of the biggest things I learned during setup:

**The graphical viewport does not need to run continuously.**

For training:

```text
Isaac Lab
   ↓
Isaac Sim
   ↓
Headless
   ↓
L40S
   ↓
Training
```

For evaluation:

```text
Saved Policy
   ↓
Evaluation
   ↓
Livestream
   ↓
Viewer
```

Example headless training:

```bash
./isaaclab.sh train     --rl_library skrl     --task Isaac-Ant-v0     --headless
```

Example visualization/evaluation:

```bash
./isaaclab.sh play     --rl_library skrl     --task Isaac-Ant-v0     --livestream 2
```

This matters because streaming the simulator when I only need training wastes resources and adds networking complexity.

---

# 💰 GPU Cost Strategy

The most important rule:

> **If the GPU is running, I assume I am paying for compute.**

When I finish:

```bash
brev stop shiny-violet-gayal
```

When I return:

```bash
brev start shiny-violet-gayal
brev refresh
brev open shiny-violet-gayal cursor
```

### STOP vs DELETE

```text
STOP
 ↓
GPU released
 ↓
Workspace preserved
```

Whereas:

```text
DELETE
 ↓
Instance removed
 ↓
Persistent data can be removed
```

Therefore:

**Stop when finished. Do not delete unless I intentionally want to destroy the environment.**

---

# 🛑 What I Would Avoid

### 1. Running Isaac Sim natively on the Mac

Use a remote NVIDIA GPU instead.

### 2. Choosing only by price

Cheap GPU ≠ best Isaac Sim setup.

Check:

- GPU
- VRAM
- RT cores
- CUDA/driver compatibility
- Isaac Sim compatibility
- provider compatibility
- networking
- streaming support

### 3. Keeping the GPU running 24/7

Stop it when finished.

### 4. Keeping all code only on the cloud VM

Use Git + GitHub.

### 5. Mixing personal projects with the simulator installation

Keep:

```text
/home/ubuntu/workspace/isaac-projects
```

separate.

### 6. Streaming during every training run

Prefer:

```text
Training → Headless
Evaluation → Livestream
```

---

# 🔧 Troubleshooting

## GPU

```bash
nvidia-smi
```

Expected:

```text
NVIDIA L40S
```

## Docker

```bash
docker ps
```

Expected:

```text
isaac-lab-nginx
isaac-lab-vscode
isaac-lab-web-viewer
```

## Restart Launchable

```bash
cd /home/ubuntu/isaac-launchable/isaac-lab

docker compose down
docker compose up -d
```

## Brev

```bash
brev refresh
brev ls
```

## SSH

```bash
brev shell shiny-violet-gayal
```

## Cursor

```bash
which cursor
```

Then:

```bash
brev open shiny-violet-gayal cursor
```

---

# 🔗 Official Resources

### NVIDIA

- [NVIDIA Isaac Sim](https://github.com/isaac-sim/IsaacSim)
- [NVIDIA Isaac Sim Organization](https://github.com/isaac-sim)
- [NVIDIA Isaac Lab](https://github.com/isaac-sim/IsaacLab)
- [Isaac Launchable](https://github.com/isaac-sim/isaac-launchable)
- [NVIDIA Brev](https://brev.nvidia.com/)
- [Brev Documentation](https://docs.nvidia.com/brev/)

### My Setup Documentation

- **[Full MacBook Setup Guide](./ISAAC_SIM_MAC_SETUP.md)**

---

# 📚 What I Plan To Learn With This

This environment is only the infrastructure.

The actual goal is to use it for:

```text
Isaac Sim
    ↓
Robot Simulation
    ↓
Isaac Lab
    ↓
RL / Imitation Learning
    ↓
Robot Policies
    ↓
VLA / Modern Robot Learning
    ↓
Simulation-to-Real
```

The long-term objective is to understand **modern robot intelligence**, not merely learn how to operate a simulator.

---

# 🗺️ Current Status

```text
MacBook
   ✅
Cursor
   ✅
Brev CLI
   ✅
Brev authentication
   ✅
AWS GPU
   ✅
NVIDIA L40S
   ✅
Docker
   ✅
Isaac Launchable
   ✅
Isaac Sim environment
   ✅
Isaac Lab environment
   ✅
SSH → Cursor
   ✅
GitHub documentation
   ✅
```

---

# ⭐ Final Workflow

```bash
# Start GPU
brev start shiny-violet-gayal

# Sync connection details
brev refresh

# Open Isaac project workspace in Cursor
brev open shiny-violet-gayal cursor   --dir /home/ubuntu/workspace/isaac-projects
```

Work → train → evaluate → commit.

Then:

```bash
git push
brev stop shiny-violet-gayal
```

---

## Author

**Tanay**

Building and learning at the intersection of:

**AI × Robotics × Physical AI × Robot Learning**

This repository documents my own journey setting up and learning the NVIDIA Isaac ecosystem from a MacBook.

---

## Disclaimer

Cloud GPU availability, pricing, NVIDIA software versions, Brev commands, provider compatibility, and Isaac Sim/Isaac Lab installation requirements can change over time.

This repository documents the setup that worked for me and should be checked against the current official NVIDIA documentation before reproducing it.

