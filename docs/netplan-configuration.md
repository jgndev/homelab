# Netplan Network Configuration Guide

This document explains how to use the netplan configuration file located in `netplan/100-network.yaml` to set up static IP networking for your homelab nodes.

## Overview

The netplan configuration provides a standardized template for configuring static IP addresses on Ubuntu 24.04 systems using Wi-Fi connectivity. This is essential for maintaining consistent network addresses across your homelab infrastructure.

## Configuration File Structure

```yaml
network:
  ethernets: {}
  version: 2
  wifis:
    wlo1:
      access-points:
        YourAccessPointName:
          password: YourWifiPassword
      dhcp4: false
      addresses: [192.168.1.101/24]
      nameservers:
        addresses: [192.168.1.1, 8.8.8.8, 8.8.4.4]
      routes:
        - to: default
          via: 192.168.1.1
```

## Configuration Parameters

### Network Interface
- **`wlo1`**: The default Wi-Fi interface name on most systems
- **Note**: Interface names may vary. Use `ip link show` to identify your Wi-Fi interface

### Access Point Configuration
- **`YourAccessPointName`**: Replace with your actual Wi-Fi network SSID
- **`YourWifiPassword`**: Replace with your Wi-Fi network password

### IP Address Configuration
- **`dhcp4: false`**: Disables DHCP and enables static IP configuration
- **`addresses: [192.168.1.101/24]`**: Sets the static IP address and subnet mask
  - Format: `[IP_ADDRESS/CIDR_NOTATION]`
  - `/24` represents a 255.255.255.0 subnet mask

### DNS Configuration
- **Primary DNS**: `192.168.1.1` (typically your router)
- **Secondary DNS**: `8.8.8.8` (Google DNS)
- **Tertiary DNS**: `8.8.4.4` (Google DNS backup)

### Routing Configuration
- **Default Route**: Directs all traffic to the gateway at `192.168.1.1`

## Node-Specific IP Assignments

Based on the homelab infrastructure, customize the IP address for each node. 
In my homelab the static IP configuration on the nodes looks like the following:

| Node          | Hostname   | IP Address    | Configuration File                |
|---------------|------------|---------------|-----------------------------------|
| Admin System  | docbar     | 192.168.1.100 | Use addresses: [192.168.1.100/24] |
| Control Plane | steeldust  | 192.168.1.101 | Use addresses: [192.168.1.101/24] |
| Worker Node 1 | reride     | 192.168.1.102 | Use addresses: [192.168.1.102/24] |
| Worker Node 2 | roughstock | 192.168.1.103 | Use addresses: [192.168.1.103/24] |

## Installation Steps

### 1. Prepare Configuration File

```bash
# Copy the template configuration
sudo cp netplan/100-network.yaml /etc/netplan/100-network.yaml
```

### 2. Customize for Each Node

Edit the configuration file with node-specific values:

```bash
sudo vim /etc/netplan/100-network.yaml
```

**Required Changes:**
1. Replace `YourAccessPointName` with your Wi-Fi SSID
2. Replace `YourWifiPassword` with your Wi-Fi password
3. Update the IP address in `addresses:` for the specific node
4. Verify the gateway address matches your network setup

### 3. Validate Configuration

```bash
# Test the configuration syntax
sudo netplan --debug try

# If successful, apply permanently
sudo netplan apply
```

### 4. Verify Network Configuration

```bash
# Check IP address assignment
ip addr show wlo1

# Test connectivity
ping -c 4 192.168.1.1  # Test gateway
ping -c 4 8.8.8.8      # Test external DNS
```

## Troubleshooting

### Common Issues

#### 1. Interface Name Mismatch
**Problem**: Network interface is not `wlo1`

**Solution**: Identify correct interface name
```bash
ip link show
# Look for wireless interface (often wlan0, wlp2s0, etc.)
```

#### 2. Wi-Fi Connection Fails
**Problem**: Cannot connect to Wi-Fi network

**Diagnostic Steps:**
```bash
# Check available networks
sudo iwlist scan | grep ESSID

# Check interface status
ip link show wlo1
```

#### 3. IP Address Conflict
**Problem**: Another device is using the same IP

**Solution**: 
- Check network with `nmap -sn 192.168.1.0/24`
- Choose an unused IP address
- Update the configuration accordingly

#### 4. DNS Resolution Issues
**Problem**: Cannot resolve domain names

**Solution**: Test DNS servers individually
```bash
nslookup google.com 192.168.1.1
nslookup google.com 8.8.8.8
```

### Configuration Rollback

If configuration causes network issues:

```bash
# Remove the netplan configuration
sudo rm /etc/netplan/100-network.yaml

# Apply default configuration
sudo netplan apply

# Or revert to DHCP temporarily
echo "network:
  version: 2
  wifis:
    wlo1:
      access-points:
        YourAccessPointName:
          password: YourWifiPassword
      dhcp4: true" | sudo tee /etc/netplan/100-network.yaml
```

## Security Considerations

### 1. File Permissions
Protect the configuration file containing Wi-Fi credentials:

```bash
sudo chmod 600 /etc/netplan/100-network.yaml
sudo chown root:root /etc/netplan/100-network.yaml
```

### 2. Credential Management
- Consider using WPA Enterprise authentication for production environments
- Store configuration files securely
- Avoid including actual credentials in version control

## Automation Integration

### Ansible Deployment

This configuration can be deployed via Ansible:

```yaml
- name: Deploy netplan configuration
  ansible.builtin.template:
    src: netplan/100-network.yaml.j2
    dest: /etc/netplan/100-network.yaml
    mode: '0600'
    owner: root
    group: root
  vars:
    wifi_ssid: "{{ homelab_wifi_ssid }}"
    wifi_password: "{{ homelab_wifi_password }}"
    node_ip: "{{ ansible_host }}"
  notify: Apply netplan configuration
```

### Template Variables

Create a Jinja2 template with variables:

```yaml
network:
  ethernets: {}
  version: 2
  wifis:
    wlo1:
      access-points:
        {{ wifi_ssid }}:
          password: {{ wifi_password }}
      dhcp4: false
      addresses: [{{ node_ip }}/24]
      nameservers:
        addresses: [192.168.1.1, 8.8.8.8, 8.8.4.4]
      routes:
        - to: default
          via: 192.168.1.1
```

## Best Practices

1. **Test Before Applying**: Always use `netplan try` before permanent application
2. **Backup Original**: Keep a copy of working configurations
3. **Document Changes**: Maintain a log of network configuration changes
4. **Consistent Naming**: Use systematic IP addressing across all nodes
5. **Monitor Connectivity**: Verify network stability after changes

## Related Documentation

- [Ubuntu Netplan Reference](https://netplan.io/reference/)

---

*Jeremy's Homelab Documentation* 