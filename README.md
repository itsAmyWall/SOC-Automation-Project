
# SOC Automation Project

This project demonstrates the deployment and configuration of a Security Operations Center (SOC) automation system using Wazuh and TheHive, hosted on virtual machines and droplets. The setup includes a Windows 10 client, a Wazuh server, and a Hive server, both running on Ubuntu 22.04 hosted via DigitalOcean.

## Architecture Overview

- **Windows 10 Client**: Hosted on VirtualBox to simulate a client machine in the network.
- **Wazuh Server**: Installed on a DigitalOcean droplet for threat detection and monitoring.
- **Hive Server**: Also hosted on a DigitalOcean droplet for incident response and investigation.
- **Firewall**: Configured on DigitalOcean to allow access only from trusted IP addresses for security.

## Step 1: Setting Up the Windows 10 Client

Created a Windows 10 Virtual Machine using VirtualBox to simulate an endpoint in the network.

## Step 2: Installing the Wazuh Server

### Create a Droplet/VM:
Deployed an Ubuntu 22.04 instance on DigitalOcean for the Wazuh server.

### Update and Upgrade the System:
Run the following commands to update the package list and upgrade the system:
```bash
sudo apt-get update && sudo apt-get upgrade -y
```

### Install Wazuh:
Used the official installation script provided by Wazuh:
```bash
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash wazuh-install.sh -a
```
The `-a` flag automates the installation process.

### Access Wazuh Dashboard:
Retrieve the login credentials displayed after installation.
Use the public IP of the Wazuh server to access the web console:
```bash
https://<WAZUH_SERVER_PUBLIC_IP>
```
![unnamed](https://github.com/user-attachments/assets/a4230826-7bc9-4fff-9166-53dc0799287b)

## Step 3: Installing TheHive

### Create a Droplet/VM:
Deployed another Ubuntu 22.04 instance on DigitalOcean for TheHive server.

### Install Dependencies:
Installed required software (Java, Cassandra, Elasticsearch, etc.) 

Configured and started each service as required.

### Install TheHive:
Downloaded and installed TheHive following this documentation:
![unnamed](https://github.com/user-attachments/assets/2d94d10b-d14c-4a94-bf88-4e82d795b890)


Use the public IP of the Hive server to access the web console:
```bash
https://<THE_HIVE_SERVER_PUBLIC_IP>:9000
```
![unnamed](https://github.com/user-attachments/assets/b86b32d0-8777-408a-8cce-aa480f46ca1c)


### Configure Firewall:
Set up a firewall on DigitalOcean to allow access only from trusted local IPs.

## Configuration

Once all servers were up and the Windows 10 client was reporting into Wazuh, the following configurations were made:

### Wazuh Agent on Windows 10

Installed the Wazuh agent on the Windows 10 client, which generated an `ossec.conf` file.

![unnamed](https://github.com/user-attachments/assets/54dd7064-bae2-4376-a911-b3d87346325e)


Configured the agent to forward logs to the Wazuh server's public IP address.
Edited the `ossec.conf` file to ensure only system-level events were forwarded, excluding others (e.g., application security events).

Before:
![unnamed](https://github.com/user-attachments/assets/c32f91d7-48b1-4ea6-8fe6-0c6011809e07)

After:
![unnamed](https://github.com/user-attachments/assets/322b6764-468e-4060-9772-a45558764100)


Restarted Wazuh on the Windows 10 client to apply the changes.

### Cassandra Configuration
Edited the Cassandra configuration file (`cassandra.yaml`):
- Changed `cluster_name` to match TheHive's configuration.
- Updated `listen_address`, `rpc_address`, and `seed_address` from localhost to TheHive's public IP.

Cleared old Cassandra data:
```bash
sudo systemctl stop cassandra
sudo rm -rf /var/lib/cassandra/*
sudo systemctl start cassandra
```
Verified that Cassandra was running properly.

### Elasticsearch Configuration
Edited the Elasticsearch configuration file (`/etc/elasticsearch/elasticsearch.yml`):
- Uncommented `cluster.name` and set it to the desired cluster name.
- Updated `network.host` with TheHive's public IP.
- Uncommented `http.port: 9200` and `cluster.initial_master_nodes`.
- Removed unnecessary default nodes.

Started and enabled Elasticsearch:
```bash
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
```
Checked the Elasticsearch service status to ensure it was running.

### TheHive Configuration
Changed ownership of necessary directories to allow TheHive to write to the storage directory.

![unnamed](https://github.com/user-attachments/assets/dad12277-61de-448f-bb46-67a0ae474e3a)


Edited TheHive configuration file (`application.conf`):
- Set database and index hostnames to TheHive's public IP.
- Updated the `cluster_name` to match Cassandra's configuration.

Started TheHive:
```bash
sudo systemctl start thehive
```
Accessed TheHive dashboard at:
```bash
http://<HIVE_SERVER_PUBLIC_IP>:9000
```

## Telemetry Testing with Mimikatz

- Excluded the download path for Mimikatz in Windows Security > Virus & Threat Protection > Exclusions.
- Downloaded and executed Mimikatz on the Windows 10 client to generate telemetry.
![unnamed](https://github.com/user-attachments/assets/65f797ee-d30b-4082-beff-d1303ed52289)
  
- Verified logs were being ingested by Wazuh and forwarded to TheHive.
- Edited `ossec.conf` on the Windows client to ensure comprehensive logging of activity for SOC analysis.
- Restarted the Wazuh agent to apply changes.

## Alert Configuration

To enhance our SOC monitoring, we configured alerts for suspicious activities, including Mimikatz execution. Here’s the process:

### Backup and Configuration Updates

#### Backup and Edit ossec.conf:
- Created a backup of the `ossec.conf` file on the Wazuh console.
- Changed `logall` and `logall_json` from no to yes in the `ossec.conf` file.
- Restarted the Wazuh agent after making these changes.

#### Update Filebeat Configuration:
- Edited the `filebeat.yml` configuration file to enable archiving:
```bash
sudo nano /etc/filebeat/filebeat.yml
```
- Changed `archives.enabled: false` to `archives.enabled: true`.
- Restarted the Filebeat service to apply the changes:
```bash
sudo systemctl restart filebeat
```

#### Create an Index for Archived Logs:
- Created an index for the archived logs on Wazuh to store and manage historical data.

#### Verify Event Viewer Logs:
- Confirmed the event logs from the Wazuh agent by searching for event number 1 and verifying that they were captured correctly.
![unnamed](https://github.com/user-attachments/assets/a665397b-91bf-4540-8b49-a062e8352655)

### Custom Alert Rule

To ensure the system can detect attempts to hide activities (such as renaming files to avoid detection), we created a custom alert rule:

#### Rule Configuration:
- Created a custom rule to look for Mimikatz execution even if the file is renamed (by using the original file name in the alert rule). This ensures detection remains functional even if attackers try to modify file names to evade detection.
Example rule: `original_file_name` in Wazuh.
![unnamed](https://github.com/user-attachments/assets/07204c7b-55ac-49c8-81fd-c571d1167cfc)


#### Testing the Custom Alert:
- Ran Mimikatz again to trigger the custom rule, and verified the alert was generated in Wazuh.

## Integration with Shuffle for Workflow Automation

To automate the response to critical alerts, we integrated Shuffle with the SOC workflow. Here’s how the integration works:

#### Configure Webhook in ossec.conf:
- Added the Shuffle webhook to the `ossec.conf` file to trigger alerts for certain conditions (e.g., Mimikatz execution).

#### Alert Workflow:
- When a Mimikatz alert is triggered in Wazuh, the alert is sent to Shuffle via the webhook.
- Shuffle processes the alert, extracts the SHA256 hash of the file, and checks the file's reputation score using VirusTotal.
- If the file is suspicious, Shuffle sends the alert details to TheHive to create an incident.
- Additionally, Shuffle sends an email to the SOC analyst to notify them of the suspicious activity.

## Key Features
- **Wazuh**: Real-time threat detection and monitoring.
- **TheHive**: Efficient incident response and case management.
- **Secure Access**: Hardened environment with restricted firewall rules.
- **Alert Workflow**: Automated alerting and incident creation via Shuffle and TheHive.
