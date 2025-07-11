# :racing_car: F1tenth-Installation-Guide

This step-by-step guide helps you install **Ubuntu 20.04**, **ROS 2 Foxy**, and launch the **F1TENTH gym simulation** using either **native** or **Docker-based** installation. Designed for students and first-time users with no prior setup.

---

## :computer: Minimum System Requirments

| Component         | Minimum                              | Recommended                          |
|------------------|---------------------------------------|--------------------------------------|
| **CPU**          | Dual-core (Intel i5/Ryzen 5+)         | Quad-core or better                  |
| **RAM**          | 8 GB                                  | 16 GB or more                        |
| **Storage**      | 40 GB free                            | 100+ GB SSD preferred                |
| **Graphics**     | Integrated GPU                        | NVIDIA GPU (for acceleration)        |
| **OS**           | Ubuntu 20.04 LTS                      | Ubuntu 20.04 LTS                     |
> :buib: Windows/macOS users can dual-boot Ubuntu or run it in a VM (VirtualBox/VMware). WSL2 with GUI is also possible (advanced users).

---

## :one: Install Ubuntu 20.04
## Option A: Dual-boot
1. Download Ubuntu 20.04 ISO from [ubuntu.com](https://releases.ubuntu.com/20.04/).
2. Create a bootable USB (using [Rufus](https://rufus.ie/) or [balenaEtcher](https://www.balena.io/etcher/)).
3. Install Ubuntu by booting from USB.
4. Update your system:

```bash
sudo apt update && sudo apt upgrade -y
```
---
## Option B: WSL2 on Windows

Open PowerShell as Administrator.

Enable WSL and Virtual Machine Platform:

```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
Restart your PC.

Set WSL2 as the default:
```bash
wsl --set-default-version 2
```
Install Ubuntu 20.04 under WSL:
```bash
wsl --install -d Ubuntu-20.04
```
Launch Ubuntu 20.04 from the Start menu, create your UNIX username/password, then update:

```bash
sudo apt update && sudo apt upgrade -y
```
## :two: Install Docker On Ununtu 20.04

### Remove Old Versions
```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

### Install Docker Engine
```bash
sudo apt update
sudo apt install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Test Docker
```bash
sudo docker run hello-world
```

### Optional: Run Docker without ```sudo```
```bash
sudo usermod -aG docker $USER
newgrp docker
```

---
## :three: Install ROS 2 Foxy

### Add ROS 2 Repository
```bash
sudo apt update && sudo apt install curl gnupg2 lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
  http://packages.ros.org/ros2/ubuntu \
  $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

sudo apt update
sudo apt install ros-foxy-desktop python3-argcomplete
```

### Source ROS in Every New Terminal:
```bash
echo "source /opt/ros/foxy/setup.bash" >> ~/.bashrc
source ~/.bashrc
```


---
## :four: Clone F1TENTH Gym ROS Repository
```bash
mkdir -p ~/sim_ws/src
cd ~/sim_ws/src
git clone https://github.com/f1tenth/f1tenth_gym_ros.git
```

---
## :five: Choose your Setup Method 

You can either build the simulation natively on Ubuntu with ROS 2, or run it inside a Docker container.

### Option A: Native ROS Build (On Ubuntu)
a. Update map path in config
Edit ```f1tenth_gym_ros/config/sim.yaml```

```yaml
map_path: "/home/YOUR_USERNAME/sim_ws/src/f1tenth_gym_ros/maps/levine"
```
Replace ```YOUR_USERNAME``` with your actual Ubuntu username.

b. Install dependencies & build
```bash
cd ~/sim_ws
rosdep install -i --from-path src --rosdistro foxy -y
colcon build
```

c. Source the workspace
```bash
source install/setup.bash
```

---
### Option B: Docker-Based Setup
a. Launch noVNC GUI simulation (no NVIDIA GPU needed):
```bash
cd f1tenth_gym_ros
docker-compose up
```
Then open your browser to: http://localhost:8080/vnc.html

b. With NVIDIA GPU (hardware-accelerated):
Install ```nvidia-docker2``` (optional, if GPU available):

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt update
sudo apt install -y nvidia-docker2
sudo systemctl restart docker
```
Then run:
```bash
rocker --nvidia --x11 \
  --volume $(pwd):/sim_ws/src/f1tenth_gym_ros \
  -- f1tenth_gym_ros
```

---

## :six: Launch the Simulation
From your native build or Docker shell:

```bash
source /opt/ros/foxy/setup.bash
source install/setup.bash   # Only if native build

ros2 launch f1tenth_gym_ros gym_bridge_launch.py
```

---
## :seven: Keyboard Teleoperation 
Enable in ```config/sim.yaml```:

```yaml
kb_teleop: true
```

Then run in a separate terminal:
```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

Use the following keys:
```bash
i: forward      ,: reverse
u/o: turn left/right while moving forward
m/.: turn left/right while reversing
k: stop
```
## :toolbox: Tips & Troubleshooting
- Always source ROS and your workspace before launching.

- If simulation doesn't launch, double-check your map path in sim.yaml.

- Use colcon build --packages-select f1tenth_gym_ros to build selectively.

- For graphical issues in VM, enable 3D acceleration or use Docker + noVNC.
