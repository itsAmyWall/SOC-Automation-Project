# SOC Automation Project with Wazuh, Hive, and Shuffle

## Overview
This project aimed to simulate a Security Operations Center (SOC) environment using the following tools:
- **Wazuh** for log collection and monitoring.
- **Hive** for case management.
- **Shuffle** for SOAR (Security Orchestration, Automation, and Response).

The setup includes:
- A **Windows 10 client** for generating logs.
- **Two Ubuntu servers** hosted on DigitalOcean vitual machines/droplets:
  - One server for **Wazuh**.
  - Another for **Hive**, with **Cassandra** and **Elasticsearch**.
  
The primary goals were to:
1. Simulate detecting credential theft with **Mimikatz**.
2. Automate threat analysis and response workflows with **Shuffle**.

---

## Infrastructure
### Components
- **Client**: Windows 10 machine.
- **Servers**: Two Ubuntu 20.04 VMs hosted on DigitalOcean.
- **Firewall**: Configured to allow access only from specific IPs (local IPs and management systems).

---

## Installation Steps
### Firewall
Configure the firewall to allow traffic only from specific IPs.

### Wazuh Installation
1. Set up the Ubuntu server for Wazuh.
2. Run the following commands:
   ```bash
   sudo apt-get update && sudo apt-get upgrade -y
   curl -s https://packages.wazuh.com/4.x/wazuh-install.sh | sudo bash

### Access the Wazuh Dashboard
Access the Wazuh dashboard via the browser using:https://<WAZUH_PUBLIC_IP>
![unnamed](https://github.com/user-attachments/assets/f3c401c5-326a-4814-9185-0d3958f5e6fc)


---

## Hive Installation

### Set Up a Second Ubuntu Server for Hive
1. **Install Dependencies:**  
   Install Java, Cassandra, and Elasticsearch.

2. **Configure Cassandra:**  
   - Edit the file `/etc/cassandra/cassandra.yaml`:
     ```yaml
     cluster_name: "HiveCluster"
     listen_address: <HIVE_PUBLIC_IP>
     ```
   - Clear existing data and restart Cassandra:
     ```bash
     sudo rm -rf /var/lib/cassandra/*
     sudo systemctl restart cassandra
     ```

3. **Configure Elasticsearch:**  
   - Edit the file `/etc/elasticsearch/elasticsearch.yml`:
     ```yaml
     network.host: <HIVE_PUBLIC_IP>
     http.port: 9200
     cluster.initial_master_nodes: ["<HIVE_PUBLIC_IP>"]
     ```
   - Restart Elasticsearch:
     ```bash
     sudo systemctl restart elasticsearch
     ```

4. **Configure Hive:**  
   Update the database and index host settings to the public IP of the Hive server.

---

## Configuration


