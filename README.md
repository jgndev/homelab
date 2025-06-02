# Jeremy's Homelab

My homelab setup featuring a Kubernetes cluster deployed and managed through Ansible automation on energy-efficient N100 mini PCs.

## 🏗️ Infrastructure Overview

### Hardware Specifications
- **Platform**: N100 16GB RAM Mini PCs
- **Operating System**: Ubuntu 24.04 LTS
- **Network**: 192.168.1.x subnet

### Node Architecture

| Hostname     | IP Address    | Role          | Function                             |
|--------------|---------------|---------------|--------------------------------------|
| `docbar`     | 192.168.1.100 | Admin System  | Ansible control node, kubectl access |
| `steeldust`  | 192.168.1.101 | Control Plane | Kubernetes control plane             |
| `reride`     | 192.168.1.102 | Worker Node   | Kubernetes worker node               |
| `roughstock` | 192.168.1.103 | Worker Node   | Kubernetes worker node               |


## 🎯 Project Goals

This homelab serves as my deep dive into:
- **Infrastructure as Code** using Ansible
- **Container Orchestration** with Kubernetes
- **Automation Best Practices** for cluster management
- **Production-like Environment** simulation at home scale

## 🚀 Features

### Ansible Automation
- **Kubernetes Installation**: Complete k8s 1.32 cluster deployment
- **User Management**: Automated ansible user setup with sudo privileges
- **System Updates**: Targeted update playbooks for different node groups
- **Security Configuration**: sudo privilege management and system hardening

### Kubernetes Cluster
- **Container Runtime**: containerd with systemd cgroup driver
- **Network Configuration**: Bridge networking with proper kernel modules
- **Package Management**: Version-locked Kubernetes components (1.32.x)
- **High Availability**: Multi-worker setup for workload distribution

## 📁 Project Structure

```
homelab/
├── ansible/                    # Ansible automation
│   ├── inventory.ini          # Infrastructure inventory
│   ├── ansible.cfg           # Ansible configuration
│   ├── kubernetes/           # K8s deployment playbooks
│   │   └── install_dependencies.yml
│   ├── users/                # User management
│   │   └── ansible.yml
│   ├── update/               # System update playbooks
│   │   ├── all.yml
│   │   ├── control_plane.yml
│   │   ├── worker_nodes.yml
│   │   └── kubernetes.yml
│   ├── reboot/               # Reboot management
│   └── visudo/               # Sudo configuration
├── notebook/                 # Documentation & notes
│   └── kubectl-completion.md
└── 100-network.yaml         # Netplan network configuration
```

## ⚙️ Setup & Usage

### Prerequisites
- Ansible control machine (docbar admin system)
- SSH key-based authentication configured
- All nodes accessible via SSH
- Python 3.12 installed on target nodes

### Initial Deployment

1. **Configure SSH Access**
   ```bash
   # Ensure SSH key is available
   ssh-keygen -t rsa -f ~/.ssh/ansible_rsa
   # Distribute to all nodes
   ```

2. **Deploy Ansible User**
   ```bash
   cd ansible
   ansible-playbook users/ansible.yml
   ```

3. **Install Kubernetes Dependencies**
   ```bash
   ansible-playbook kubernetes/install_dependencies.yml
   ```

4. **Initialize Kubernetes Cluster**
   ```bash
   # Control plane initialization (manual step)
   # Worker node joining (manual step)
   ```

### Maintenance Operations

#### System Updates
```bash
# Update all nodes
ansible-playbook update/all.yml

# Update only control plane
ansible-playbook update/control_plane.yml

# Update only worker nodes
ansible-playbook update/worker_nodes.yml

# Update kubernetes nodes only
ansible-playbook update/kubernetes.yml
```

#### Cluster Management
```bash
# Access cluster from admin node
kubectl get nodes
kubectl get pods --all-namespaces
```

## 🔧 Technical Details

### Kubernetes Configuration
- **Version**: 1.32.0 (version-locked)
- **Container Runtime**: containerd
- **CNI**: Bridge networking with iptables
- **CGroup Driver**: systemd
- **Swap**: Disabled across all nodes

### Network Configuration
- **Subnet**: 192.168.1.0/24
- **Gateway**: 192.168.1.1
- **DNS**: Primary (192.168.1.1), Secondary (8.8.8.8, 8.8.4.4)
- **Static IPs**: Configured via netplan

### Security Features
- SSH key-based authentication only
- Dedicated ansible user with sudo privileges
- Package version locking for stability
- Host key verification disabled for lab environment

## 📚 Learning Outcomes

### Ansible Skills
- Inventory management and host grouping
- Playbook structure and task organization
- Module usage (apt, systemd, user, file, etc.)
- Conditional execution and loops
- System configuration management

### Kubernetes Skills
- Cluster architecture understanding
- Container runtime configuration
- Network plugin setup
- Node management and maintenance
- kubectl command-line proficiency

### Infrastructure Skills
- Linux system administration
- Network configuration with netplan
- Service management with systemd
- Package management and version control
- Security hardening basics

## 🔄 Current Status

- ✅ Hardware setup complete
- ✅ Ubuntu 24.04 deployment
- ✅ Ansible automation framework
- ✅ Kubernetes dependencies installation
- ✅ User management automation
- ✅ System update automation
- 🚧 Cluster initialization (in progress)
- 🚧 Application deployment pipelines
- 🚧 Monitoring and observability

## 🎯 Next Steps

1. **Complete K8s Cluster Setup**
   - Automate cluster initialization
   - Automate worker node joining
   - Deploy CNI plugin

2. **Expand Automation**
   - Application deployment playbooks
   - Backup and recovery procedures
   - Monitoring stack deployment

3. **Advanced Features**
   - GitOps integration
   - CI/CD pipelines
   - Service mesh evaluation

## 📖 Documentation

Additional documentation can be found in the `docs/` directory:
- `kubectl-completion.md`: Kubectl shell completion setup

## 🤝 Contributing

This is a personal learning project, but suggestions and improvements are welcome!

## 📄 License

See [LICENSE.md](LICENSE.md) for details.

---

*Built with 🤓 for learning and exploration*
