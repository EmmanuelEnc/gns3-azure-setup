# 🛠️ GNS3 Azure Setup – Stage 1

## 📌 Project Overview
This project is part of a multi-stage portfolio titled Cloud_netOps360, which simulates enterprise network operations and security monitoring using both cloud and local virtualization tools.
Stage 1 focuses on deploying a cloud-based GNS3 server in Microsoft Azure that will host and simulate the entire infrastructure of later stages. This includes multiple subnets, routers, switches, security tools, and client/server services.

The GNS3 server is hosted on an Ubuntu VM in Azure and configured to allow remote management using the GNS3 GUI from a local Windows system. This setup lays the groundwork for routing simulation, IDS testing, and realistic network segmentation in later stages.

## 🛠️ Technologies Used
- **Microsoft Azure** – Cloud platform for network simulation, VM provisioning, and subnetting
- **Ubuntu Server 22.04 LTS** – Operating system for the GNS3 VM
- **GNS3 Server** – Network simulation software used to build and manage virtual networks
- **Docker** – Provides lightweight, containerized devices for testing connectivity and functionality
- **uBridge** – Required GNS3 backend tool for bridging Docker, VPCS, and virtual network links
- **VNet & Subnetting** – Configured Azure networking with GNS3-Net for VM communication
- **NSG (Network Security Groups)** – Used to control traffic to the GNS3 VM
- **SSH (port 22)** – Used for remote terminal access from local Windows machine
- **GNS3 GUI (local)** – Installed on Windows PC to remotely control the Azure GNS3 server

## ⚙️ To Replicate This Project:

### 📍 Step 1: Plan Network Layout
- Created a new **Virtual Network (VNet)** named `CloudNetops360-VNet` with address space `10.0.0.0/16`.  

![vnet](https://i.imgur.com/bSfbWSZ.png)
- Created a subnet named `GNS3-Net` within the VNet using address range `10.0.100.0/24`.  

![subnet](https://i.imgur.com/rquGsUn.png)
- This subnet is dedicated to the GNS3 VM.

### 📍 Step 2: Deploy the GNS3 VM on Azure
- Launched a new virtual machine:
  - OS: **Ubuntu 22.04 LTS (Server)**
  - VM Size: **Standard E2s_v3** (2 vCPU, 16GB RAM – supports nested virtualization and Docker)
  - Assigned to subnet: **GNS3-Net**
  - Assigned a **static private IP**: `10.0.100.4`
  - Attached a **public IP** to enable remote access  

![gns3vm](https://i.imgur.com/ZQNJgF3.png)

### 📍 Step 3: Configure Network Security Group (NSG)
Created an NSG with the following inbound rules:

| Priority | Name             | Port(s)    | Protocol | Source        | Destination | Action | Purpose                                  |
|----------|------------------|------------|----------|----------------|-------------|--------|------------------------------------------|
| 100      | Allow-SSH        | 22         | TCP      | My Public IP   | Any         | Allow  | SSH access to VM                         |
| 110      | Allow-GNS3-API   | 3080       | TCP      | My Public IP   | Any         | Allow  | Remote GNS3 GUI to connect               |
| 120      | Allow-GNS3-Ports | 2000–5000  | TCP/UDP  | My Public IP   | Any         | Allow  | Device communication (Dynamips, QEMU)   |
| 130      | Allow-ICMP       | *          | ICMP     | My Public IP   | Any         | Allow  | Ping testing                             |
| 1000     | Deny-All         | *          | *        | Any            | Any         | Deny   | Block all other traffic (default deny)   |

### 📍 Step 4: SSH into VM & Install GNS3 + Dependencies
Connected to VM via SSH:
```bash
ssh gns3-admin@<AzurePublicIP>
```
Ran the following setup:
```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:gns3/ppa
sudo apt update
sudo apt install -y gns3-server docker.io ubridge
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
sudo usermod -aG ubridge $USER
newgrp docker
newgrp ubridge
```

### 📍 Step 5: Configure GNS3 Server for Remote Access
Created the GNS3 server config:
```bash
mkdir -p ~/.config/GNS3
nano ~/.config/GNS3/gns3_server.conf
```
Added this configuration:
```ini
[Server]
host = 0.0.0.0
port = 3080
local = True
```
Started the GNS3 server:
```bash
gns3server &
```

### 📍 Step 6: Pull Docker Images (for testing)
```bash
docker pull alpine
docker pull ubuntu
docker pull busybox
```

### 📍 Step 7: Setup GNS3 GUI on Local Windows PC
- Downloaded GNS3 from https://gns3.com/software/download
- Installed only the **GNS3 GUI**, skipping GNS3 VM and emulators
- Added remote server in:
  - **Edit > Preferences > Server > Remote Server**
  - Host: Azure VM Public IP
  - Port: 3080
  - Marked as **Main Server**

![gns3-guisettings](https://i.imgur.com/eVZNgDE.png)  

![servercli](https://i.imgur.com/r8uWhwd.png)

### 📍 Step 8: Validate Remote Connection
- Opened GNS3 GUI
- Created a new project
- Successfully added Docker containers (alpine, ubuntu) and VPCS nodes
- Started devices and confirmed functionality
- Verified logs on server and ensured `gns3server` responded without error  

![devicesconnected](https://i.imgur.com/4zK93MQ.png)

---

## 🚀 Future Improvements

As the foundation of the Cloud_netOps360 project, this GNS3 VM deployment in Azure sets the stage for future expansions and advanced simulations. The next steps include:

### 🔸 Stage 2: Simulate Internal Network Topology
In Stage 2, the GNS3 server will be used to build a simulated internal enterprise network. This will include:
- Virtual routers and switches (using IOS images or Docker-based devices)
- A basic LAN topology representing separate departments or zones
- Simulated traffic paths to validate GNS3 functionality and routing behavior

### 🔧 Additional Improvements:
- Automate GNS3 server startup via `systemd`  
- Add advanced router/firewall images (Cisco IOSv, pfSense, VyOS)  
- Enable encrypted GNS3 API communication  
- Integrate Azure monitoring or Sentinel for logging  
- Use Ansible or Terraform for infrastructure as code (IaC)

---

## ✅ Conclusion

Stage 1 successfully established the core of the Cloud_netOps360 lab environment by deploying and configuring a **remote GNS3 server in Microsoft Azure**. Key accomplishments include:

- Creation of a VNet and subnet for the GNS3 environment
- Deployment and configuration of an Ubuntu VM running GNS3, Docker, and uBridge
- Setup of remote access via GNS3 GUI from a local Windows PC
- Verified Docker-based containers and connectivity
- Documented every configuration step for reproducibility

This stage provides a secure and scalable platform to simulate advanced networking concepts, traffic flow, and cybersecurity scenarios in upcoming stages.

---
© 2025 Emmanuel Encarnacion • GitHub Portfolio
