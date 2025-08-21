# Vagrant Learning Environment

This directory contains various Vagrant configurations demonstrating different virtualization concepts, multi-VM setups, provisioning strategies, and Infrastructure as Code (IaC) practices. Each subdirectory represents a different learning scenario or use case.

## Table of Contents

1. [What is Vagrant?](#what-is-vagrant)
2. [Prerequisites](#prerequisites)
3. [Directory Structure](#directory-structure)
4. [Core Concepts](#core-concepts)
5. [Project Descriptions](#project-descriptions)
6. [Common Commands](#common-commands)
7. [Troubleshooting](#troubleshooting)
8. [Best Practices](#best-practices)

## What is Vagrant?

Vagrant is a tool for building and managing virtual machine environments in a single workflow. It provides easy-to-configure, reproducible, and portable work environments that can be shared across teams and platforms.

### Key Benefits:
- **Consistency**: Same environment across development, testing, and production
- **Portability**: Works on Windows, macOS, and Linux
- **Automation**: Automated VM provisioning and configuration
- **Version Control**: Vagrantfiles can be committed to version control
- **Isolation**: Clean, separate environments for different projects

## Prerequisites

Before using these Vagrant configurations, ensure you have:

1. **Vagrant** installed ([Download here](https://www.vagrantup.com/downloads))
2. A **hypervisor** installed:
   - **VirtualBox** (free, cross-platform)
   - **VMware Desktop** (paid, better performance)
   - **Hyper-V** (Windows only)
3. **Git** for version control (optional but recommended)

## Directory Structure

```
vagrant/
├── README.md                 # This file
├── centos/                   # Single CentOS Stream 9 VM with Apache
├── ubuntu/                   # Basic Ubuntu 22.04 VM template
├── wordpress/               # WordPress development environment
├── finance/                 # CentOS VM with static finance website
├── multivms/               # Multi-VM setup (2 Ubuntu web + 1 CentOS DB)
├── my-vms/                 # VMware-specific configuration
├── financeIAC/             # IaC version of finance project
└── wordpressIAC/           # IaC version of WordPress project
```

## Core Concepts

### 1. Vagrantfile

The `Vagrantfile` is the configuration file that defines:
- Base operating system (box)
- Network configuration
- Resource allocation (CPU, memory)
- Provisioning scripts
- Shared folders
- Multi-machine setups

### 2. Boxes

Vagrant boxes are pre-built base images:
- **ubuntu/focal64**: Ubuntu 20.04 LTS
- **ubuntu/jammy64**: Ubuntu 22.04 LTS
- **centos/7**: CentOS 7 (End of Life)
- **eurolinux-vagrant/centos-stream-9**: CentOS Stream 9

### 3. Providers

Providers are the virtualization platforms:
- **VirtualBox**: Default, free, cross-platform
- **VMware**: Better performance, requires license
- **Hyper-V**: Windows built-in hypervisor

### 4. Networking

Three main networking modes:

#### Private Network (Host-Only)
```ruby
config.vm.network "private_network", ip: "192.168.56.10"
```
- VM accessible only from host machine
- Multiple VMs can communicate with each other
- Good for development environments

#### Public Network (Bridged)
```ruby
config.vm.network "public_network", bridge: "en0: Wi-Fi (AirPort)"
```
- VM gets IP from local network DHCP
- VM appears as separate device on network
- Accessible from other devices on network

#### Port Forwarding
```ruby
config.vm.network "forwarded_port", guest: 80, host: 8080
```
- Maps guest port to host port
- Access guest service via localhost:host_port

### 5. Provisioning

Automated configuration of VMs after creation:

#### Shell Provisioning
```ruby
config.vm.provision "shell", inline: <<-SHELL
  apt-get update
  apt-get install -y nginx
SHELL
```

#### External Script
```ruby
config.vm.provision "shell", path: "setup.sh"
```

### 6. Synced Folders

Share files between host and guest:
```ruby
config.vm.synced_folder "./data", "/vagrant_data"
```

### 7. Multi-Machine Environments

Define multiple VMs in single Vagrantfile:
```ruby
Vagrant.configure("2") do |config|
  config.vm.define "web" do |web|
    web.vm.box = "ubuntu/focal64"
  end
  
  config.vm.define "db" do |db|
    db.vm.box = "centos/7"
  end
end
```

## Project Descriptions

### 📁 ubuntu/
**Purpose**: Basic Ubuntu 22.04 template
- **Box**: ubuntu/jammy64
- **Use Case**: Clean Ubuntu environment for testing
- **Features**: Minimal configuration, commented examples

### 📁 centos/
**Purpose**: CentOS Stream 9 with web server
- **Box**: eurolinux-vagrant/centos-stream-9
- **Networking**: Public network (bridged)
- **Provisioning**: Apache, basic tools, system info collection
- **Features**: 
  - Installs httpd, wget, unzip, git
  - Disables firewall for development
  - Creates `/opt/devopsdir` with system information

### 📁 wordpress/
**Purpose**: WordPress development environment
- **Box**: ubuntu/focal64
- **Memory**: 1600 MB
- **Networking**: Private (192.168.56.28) + Public
- **Status**: Template only (provisioning commented out)

### 📁 finance/
**Purpose**: Basic CentOS with private networking
- **Box**: eurolinux-vagrant/centos-stream-9
- **Memory**: 1024 MB
- **Networking**: Private network (192.168.56.22) + Public
- **Status**: Template only (no active provisioning)

### 📁 multivms/ ⭐
**Purpose**: Multi-tier application architecture
- **Architecture**: 
  - **web01**: Ubuntu 20.04 (192.168.56.41)
  - **web02**: Ubuntu 20.04 (192.168.56.42)
  - **db01**: CentOS 7 (192.168.56.43) with MariaDB
- **Key Features**:
  - Demonstrates multi-VM coordination
  - Database server with MariaDB
  - Fixed CentOS 7 EOL repository issues
  - Private network communication between VMs

### 📁 my-vms/
**Purpose**: VMware Desktop provider example
- **Box**: ubuntu/jammy64
- **Provider**: VMware Desktop (not VirtualBox)
- **Resources**: 2 CPUs, 1GB RAM
- **Networking**: Private + Public (bridged to Wi-Fi)
- **Synced Folders**: `./scripts` → `/opt/scripts` (rsync)

### 📁 financeIAC/ 🚀
**Purpose**: Infrastructure as Code - Finance Website
- **Box**: eurolinux-vagrant/centos-stream-9
- **Application**: Mini Finance website template
- **Provisioning**: Fully automated web application deployment
- **Features**:
  - Downloads and deploys Tooplate finance theme
  - Configures Apache web server
  - Demonstrates complete IaC workflow
- **Access**: http://192.168.56.28 (after provisioning)

### 📁 wordpressIAC/ 🚀
**Purpose**: Infrastructure as Code - WordPress Installation
- **Box**: ubuntu/focal64
- **Memory**: 1600 MB
- **Application**: Complete WordPress LAMP stack
- **Provisioning**: Fully automated WordPress deployment
- **Features**:
  - Installs Apache, MySQL, PHP
  - Downloads and configures WordPress
  - Creates database and user
  - Sets up virtual host configuration
- **Access**: http://192.168.56.29 (after provisioning)
- **Database**: wordpress/wordpress/admin123

## Common Commands

### Basic Operations
```bash
# Start VM(s) in current directory
vagrant up

# SSH into VM (single VM environment)
vagrant ssh

# SSH into specific VM (multi-VM environment)
vagrant ssh web01

# Check status of all VMs
vagrant status

# Stop VM(s)
vagrant halt

# Restart VM(s)
vagrant reload

# Delete VM(s) completely
vagrant destroy
```

### Provisioning Commands
```bash
# Run provisioning on running VM
vagrant provision

# Provision specific VM
vagrant provision db01

# Reload VM with provisioning
vagrant reload --provision
```

### Box Management
```bash
# List installed boxes
vagrant box list

# Add new box
vagrant box add ubuntu/jammy64

# Update box
vagrant box update

# Remove box
vagrant box remove ubuntu/focal64
```

### Multi-VM Operations
```bash
# Start specific VM
vagrant up web01

# Start all VMs
vagrant up

# SSH to specific VM
vagrant ssh db01

# Provision all VMs
vagrant provision

# Check global status
vagrant global-status
```

## Troubleshooting

### Common Issues

#### 1. CentOS 7 Repository Errors
**Problem**: `Could not resolve host: mirrorlist.centos.org`
**Solution**: CentOS 7 reached EOL. Use vault repositories (see `multivms/Vagrantfile`)

#### 2. Network Interface Not Found
**Problem**: Bridge interface not found
**Solution**: Update bridge name in Vagrantfile:
```ruby
# Check available interfaces
VBoxManage list bridgedifs

# Update Vagrantfile
config.vm.network "public_network", bridge: "Your Interface Name"
```

#### 3. VM Won't Start
**Problem**: VirtualBox/VMware errors
**Solutions**:
- Check if virtualization is enabled in BIOS
- Ensure no conflicts with other hypervisors
- Try different provider

#### 4. Provisioning Fails
**Problem**: Scripts fail during provisioning
**Solutions**:
- Check internet connectivity in VM
- Verify script syntax
- Run provisioning separately: `vagrant provision`

#### 5. Port Conflicts
**Problem**: SSH port already in use
**Solution**: Vagrant automatically reassigns ports:
```
web01: 22 => 2222
web02: 22 => 2201
db01:  22 => 2202
```

### Best Practices

1. **Version Control**: Always commit Vagrantfiles
2. **Documentation**: Comment your configurations
3. **Resource Management**: Don't over-allocate memory
4. **Networking**: Use consistent IP ranges
5. **Provisioning**: Keep scripts idempotent
6. **Security**: Don't commit sensitive data
7. **Cleanup**: Regularly destroy unused VMs

## Learning Path

### Beginner
1. Start with `ubuntu/` - basic single VM
2. Try `centos/` - different OS with provisioning
3. Experiment with networking in `wordpress/`

### Intermediate  
1. Explore `multivms/` - multi-VM architecture
2. Try different providers with `my-vms/`
3. Understand provisioning with basic scripts

### Advanced
1. Study `financeIAC/` - complete web application deployment
2. Analyze `wordpressIAC/` - complex multi-service provisioning
3. Create your own multi-tier applications

## Additional Resources

- [Vagrant Documentation](https://www.vagrantup.com/docs)
- [Vagrant Cloud](https://app.vagrantup.com/boxes/search) - Find more boxes
- [VirtualBox Documentation](https://www.virtualbox.org/wiki/Documentation)
- [VMware Vagrant Provider](https://www.vagrantup.com/docs/providers/vmware)

## Support

For issues or questions:
1. Check the [Vagrant GitHub Issues](https://github.com/hashicorp/vagrant/issues)
2. Review provider-specific documentation
3. Consult the official Vagrant community forums

---

**Note**: This environment is designed for learning and development. Always review and understand configurations before running in production environments.
