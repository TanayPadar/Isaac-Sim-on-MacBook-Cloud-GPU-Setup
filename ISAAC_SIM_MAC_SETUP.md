# Isaac Sim on macOS — Cloud GPU Development Setup

A practical guide for running NVIDIA Isaac Sim + Isaac Lab from a
MacBook using a remote NVIDIA GPU, while using Cursor locally as the
development environment.

> **Architecture:** MacBook → Cursor → SSH/Brev → Cloud NVIDIA GPU →
> Isaac Lab/Isaac Sim

## Why this setup

Isaac Sim is a GPU-intensive robotics simulator. For a MacBook workflow,
use the Mac as the development interface and a remote Linux NVIDIA GPU
as the compute machine.

``` text
MacBook
   ↓
Cursor + SSH
   ↓
NVIDIA Brev
   ↓
AWS Linux GPU
   ↓
Docker
   ├── Isaac Sim
   └── Isaac Lab
```

## Recommended GPU

For this workflow, NVIDIA L40S is a strong starting point:

- 48 GB VRAM
- RTX capabilities
- RT cores
- Tensor cores
- CUDA
- Suitable memory for Isaac Sim and many Isaac Lab workloads

There is no universal “best GPU”; workload, VRAM, provider
compatibility, networking, and cost all matter.

## Cloud provider choice

We used **NVIDIA Brev + AWS**.

Brev provides GPU provisioning, SSH, instance management, persistent
workspace, Cursor integration, and port management.

We chose AWS because NVIDIA’s official `isaac-launchable` repository
documents AWS as tested and currently notes that the project is not
compatible with Crusoe instances.

**Lesson:** do not choose a GPU only by hourly price. Verify that the
provider and deployment method work with the exact Isaac Sim/Isaac Lab
stack you plan to use.

## What we tried

### RunPod

RunPod was tested first. The GPU was capable, but remote Isaac Sim
networking/streaming introduced additional complexity around WebRTC,
TCP/UDP ports, public IPs, and host networking.

For this learning setup, Brev + AWS provided a cleaner workflow.

### Browser VS Code

NVIDIA’s official Isaac Launchable includes a browser VS Code
environment. We successfully built and started the Launchable
containers, but exposing the browser IDE through Brev was unnecessarily
awkward.

We therefore use **Cursor over SSH** for normal development while
retaining the Isaac Sim/Isaac Lab container environment.

------------------------------------------------------------------------

# Final setup

``` text
MacBook
├── Cursor
└── Brev CLI
      │
      │ SSH
      ▼
NVIDIA Brev
└── AWS EC2
    └── NVIDIA L40S 48 GB
        └── Docker
            ├── Isaac Sim
            └── Isaac Lab

Persistent project workspace:
/home/ubuntu/workspace/

Version control:
Git + GitHub
```

------------------------------------------------------------------------

# Setup

## 1. Create the GPU instance

In NVIDIA Brev, create an environment with:

``` text
GPU: NVIDIA L40S
VRAM: 48 GB
Provider: AWS
Architecture: x86_64
```

Exact availability and pricing can change.

## 2. Clone NVIDIA Isaac Launchable

On the GPU instance:

``` bash
cd /home/ubuntu

git clone https://github.com/isaac-sim/isaac-launchable

cd isaac-launchable/isaac-lab
```

Start it:

``` bash
docker compose up -d
```

The first build can take a long time because large container images are
downloaded/built.

## 3. Verify Docker

``` bash
docker ps
```

Expected containers:

``` text
isaac-lab-nginx
isaac-lab-vscode
isaac-lab-web-viewer
```

If these are running, the Launchable environment is alive.

## 4. Configure Brev Secure Links

The Launchable uses an nginx gateway. Add an HTTP Secure Link with:

``` text
Destination port: 80
Protocol: HTTP
```

For Kit App Streaming, NVIDIA’s Launchable documentation uses ports
including:

``` text
1024
47998
49100
```

## 5. Install Brev CLI on macOS

``` bash
brew install brevdev/homebrew-brev/brev
```

Verify:

``` bash
brev --version
```

If Homebrew reports outdated Command Line Tools, update them through:

**System Settings → General → Software Update**

Then retry.

## 6. Log in and sync

``` bash
brev login
brev refresh
brev ls
```

Example:

``` text
NAME                 STATUS      GPU

shiny-violet-gayal   STOPPED     L40S
```

`brev refresh` synchronizes web-created instances with local SSH
configuration.

## 7. Install Cursor CLI

Open Cursor and press:

``` text
⌘ + Shift + P
```

Run:

``` text
Install 'cursor' command in PATH
```

Verify:

``` bash
which cursor
```

Expected:

``` text
/usr/local/bin/cursor
```

## 8. Connect Cursor to Brev

Start the GPU:

``` bash
brev start shiny-violet-gayal
```

Then:

``` bash
brev refresh
```

Open the remote environment in Cursor:

``` bash
brev open shiny-violet-gayal cursor
```

Or open the project directory directly:

``` bash
brev open shiny-violet-gayal cursor   --dir /home/ubuntu/workspace/isaac-projects
```

------------------------------------------------------------------------

# Project storage

Keep your projects separate from the Isaac Sim installation.

Recommended:

``` text
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

Use Git/GitHub as the source of truth.

Recommended project structure:

``` text
robot-learning-experiment/
├── README.md
├── configs/
├── scripts/
├── source/
├── environments/
├── policies/
├── checkpoints/
├── logs/
└── .gitignore
```

Do not commit large datasets or checkpoints blindly. Use Git LFS or
suitable external storage where appropriate.

------------------------------------------------------------------------

# Isaac Sim vs Isaac Lab

### Isaac Sim

The simulation platform:

``` text
Physics
Rendering
Sensors
Robots
Scenes
Simulation
```

### Isaac Lab

The robot-learning framework built on Isaac Sim:

``` text
Reinforcement Learning
Imitation Learning
Robot Environments
Parallel Simulation
Policy Training
Evaluation
```

Conceptually:

``` text
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

------------------------------------------------------------------------

# Headless training vs visualization

You do not need the graphical viewport running during every experiment.

### Training

``` text
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

Example:

``` bash
./isaaclab.sh train     --rl_library skrl     --task Isaac-Ant-v0     --headless
```

### Visualization / evaluation

``` text
Saved policy
   ↓
Evaluation
   ↓
Livestream
   ↓
Browser viewer
```

Example:

``` bash
./isaaclab.sh play     --rl_library skrl     --task Isaac-Ant-v0     --livestream 2
```

Use visualization when you actually need to observe the simulation.

------------------------------------------------------------------------

# What to avoid

## Do not force native Isaac Sim onto macOS

Use a remote Linux NVIDIA GPU.

## Do not use CPU-only cloud machines

Isaac Sim is GPU-heavy.

## Do not select only by hourly price

Check:

``` text
GPU
VRAM
RT cores
driver compatibility
Isaac Sim support
provider compatibility
streaming support
networking
```

## Do not leave the GPU running

When finished:

``` bash
brev stop shiny-violet-gayal
```

## Do not delete the instance to save money

Stopping and deleting are different.

``` text
STOP
 ↓
GPU released
 ↓
workspace preserved
```

Deleting removes the instance and can remove its persistent data.

## Do not manually manage IP addresses

After restarting:

``` bash
brev refresh
```

## Do not rely only on the cloud workspace

Push important code to GitHub.

------------------------------------------------------------------------

# Troubleshooting

## Check GPU

``` bash
nvidia-smi
```

Expected GPU:

``` text
NVIDIA L40S
```

## Check Docker

``` bash
docker ps
```

Expected:

``` text
isaac-lab-nginx
isaac-lab-vscode
isaac-lab-web-viewer
```

## Restart Launchable

``` bash
cd /home/ubuntu/isaac-launchable/isaac-lab

docker compose down
docker compose up -d
```

Then:

``` bash
docker ps
```

## Cursor cannot connect

First:

``` bash
brev refresh
```

Then test SSH:

``` bash
brev shell shiny-violet-gayal
```

Check Cursor:

``` bash
which cursor
```

Then:

``` bash
brev open shiny-violet-gayal cursor
```

------------------------------------------------------------------------

# Cost management

The most important rule:

> **GPU running = paying.**

When finished:

``` bash
brev stop shiny-violet-gayal
```

When continuing:

``` bash
brev start shiny-violet-gayal
brev refresh
brev open shiny-violet-gayal cursor
```

A stopped instance avoids GPU compute charges, although persistent
storage may still have a smaller cost depending on the
configuration/provider.

------------------------------------------------------------------------

# Recommended daily workflow

## Start

``` bash
brev start shiny-violet-gayal
brev refresh
brev open shiny-violet-gayal cursor
```

## Develop

Work inside:

``` text
/home/ubuntu/workspace/isaac-projects/
```

## Train

Run Isaac Lab headlessly.

``` text
Cursor terminal
      ↓
Isaac Lab
      ↓
Isaac Sim
      ↓
L40S
```

## Visualize

Use livestreaming only when needed.

## Commit

``` bash
git add .
git commit -m "experiment: ..."
git push
```

## Finish

``` bash
brev stop shiny-violet-gayal
```

------------------------------------------------------------------------

# Final recommended stack

| Layer             | Choice                      |
|-------------------|-----------------------------|
| Laptop            | MacBook                     |
| IDE               | **Cursor**                  |
| Remote management | **NVIDIA Brev CLI**         |
| Cloud provider    | **AWS**                     |
| GPU               | **NVIDIA L40S 48 GB**       |
| OS                | Linux                       |
| Container         | Docker                      |
| Simulator         | **Isaac Sim**               |
| Robot learning    | **Isaac Lab**               |
| Remote access     | SSH                         |
| Visualization     | Isaac Sim livestream/WebRTC |
| Project storage   | `/home/ubuntu/workspace`    |
| Version control   | Git                         |
| Repository        | GitHub                      |

------------------------------------------------------------------------

# Key lessons

1.  **The Mac does not need to be the compute machine.** It is the
    development interface.
2.  **L40S is a strong starting GPU** for this workflow; there is no
    universal best GPU.
3.  **Provider compatibility matters.** Check the exact Isaac deployment
    method before choosing the cheapest GPU.
4.  **Cursor over SSH is cleaner for development** than relying on a
    browser IDE.
5.  **Separate your projects from the simulator installation.**
6.  **Use headless mode for training and livestreaming for
    visualization.**
7.  **Stop the GPU when finished.**
8.  **Use GitHub as the source of truth.**

------------------------------------------------------------------------

# Final command sequence

Once everything is configured:

``` bash
# Start GPU
brev start shiny-violet-gayal

# Sync SSH information
brev refresh

# Open remote project directory in Cursor
brev open shiny-violet-gayal cursor   --dir /home/ubuntu/workspace/isaac-projects
```

When finished:

``` bash
git push
brev stop shiny-violet-gayal
```

**Mac → Brev → AWS L40S → Cursor → Isaac Sim → Isaac Lab → Robot
Learning**
