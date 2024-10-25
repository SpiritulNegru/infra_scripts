# Ansible Project for Managing Windows Services

This Ansible project allows you to install, configure, and manage Windows services based on user input. The two services included in this project are:

1. **Microsoft SQL Server** (requires input parameters like instance name and password).
2. **Windows Time Service (w32time)** (does not require input parameters).

The playbook prompts the user to select a service and then runs the corresponding role to install or manage the service on remote Windows servers.

## Prerequisites

### Ansible Requirements
- Ansible installed on a controller machine.
- WinRM (Windows Remote Management) enabled on the Windows servers.
- Python `pywinrm` package installed on the Ansible control node.
  pip install pywinrm

### Windows Requirements
Remote servers should have WinRM enabled and configured to allow Ansible to communicate over the network.

1. Create an Inventory File
Create an inventory file (inventory.ini) listing the Windows servers you want to manage:

[windows_servers]
win_server1 
win_server2

[windows_servers:vars]
ansible_user=admin
ansible_password={{ ansible_pass }}
ansible_connection=winrm
ansible_winrm_transport=ntlm
ansible_winrm_server_cert_validation=ignore
This inventory file contains information about the remote Windows servers and authentication credentials.

2. Running the Playbook
Run the main playbook (playbook.yml) to select and manage services based on user input:

ansible-playbook playbook.yml --ask-vault-pass
You'll be prompted to enter the Ansible Vault password to decrypt the vault.yml file.

After that, the playbook will prompt you to choose which service to manage:

Which service do you want to manage? (1: SQL Server, 2: Windows Time Service):
Select 1 to manage SQL Server.
Select 2 to manage Windows Time Service.

3. Roles Overview
SQL Server Role (sql_server_role)
This role installs and configures Microsoft SQL Server with user-defined inputs (e.g., instance name and SA password). The main task file is located at:

roles/sql_server_role/tasks/main.yml
You can modify SQL Server settings (e.g., instance name, SA password) in the variables file:

roles/sql_server_role/vars/main.yml
Windows Time Service Role (w32time_role)
This role ensures that the Windows Time Service (w32time) is running on the remote servers. The main task file is located at:

roles/w32time_role/tasks/main.yml
4. Ignoring Sensitive Files
Ensure that sensitive files like vault.yml or any other secrets are not committed to version control by using a .gitignore file:

# .gitignore

# Ignore inventory files with secrets
inventory.ini

# Ignore unencrypted vault files
vault.yml

# Ignore other sensitive files
*.secrets
*.key

How winRM Works in the Playbook:
When you run the playbook, Ansible connects to the remote Windows servers via WinRM.
For services like SQL Server, Ansible uses WinRM to pass the required input parameters to the installer.
For services like w32time, which don’t require input parameters, Ansible simply uses WinRM to manage the service state.
In both cases, WinRM acts as the communication channel between the Ansible controller and the Windows target servers, executing the desired tasks.

### Ways can you utilize Ansible playbooks to automate routine service management
Automating Service Management: run these playbooks periodically to ensure critical services remain running or restart them if they fail.
Ensuring Service Health: By checking service status regularly and restarting services that aren’t running properly, to ensure the overall health of environment.
Scheduled Tasks: use cron jobs to schedule these playbooks to run at specific intervals.

### modules I would typically use:
Routine Service Management: win_service, win_service_info for checking, starting, stopping, or restarting Windows services.
Software and Feature Installation: win_package for installing applications, win_feature for enabling Windows features like IIS or .NET.
Configuration & Management: win_registry, win_file, win_task, win_hostname, win_shell and win_copy for managing configuration files and registry settings.
Security and User Management: win_user, win_group, and win_firewall_rule to manage users, groups, and firewall settings.
System Updates and Patching: win_updates for managing Windows updates.
System Reboots: win_reboot

### Deploy sample application and register with service using ansible
Environment-Specific Variables: Different settings (like ports, app paths, database URLs) are defined for each environment.
Modular Playbooks: Use Ansible roles to modularize tasks (e.g., app deployment, service configuration) and reuse them across environments.
Inventory Management: Separate inventory files help ensure different machines are targeted in each environment (development, testing, production).
Service Registration: The role configures and starts the application as a service, ensuring it starts on boot and opens the appropriate firewall port.

# Windows Server Monitoring
## the key performance metrics that should be monitor on Windows servers

CPU Usage 
Processor Queue Length
Available Memory
Page File Usage
Disk Queue Length
Disk Latency
Disk Space Usage
Network Interface Utilization
Packet Loss/Error Rate
Connections (TCP/UDP)
Service Status
Windows Event Logs
Application-Specific Metrics like IIS servers, monitor: requests per second, active connections Failed requests
For SQL servers, monitor: query execution times, blocked processes

## Automated Monitoring & Alarming
Set Up Data Collection: Use WMI collector scripts to regularly collect performance data.
Set Thresholds: Define acceptable limits for CPU, memory, disk, and network usage.
Configure Alarming: with tools like Graphana , Use PowerShell scripts to compare collected metrics against thresholds.
Trigger alarms (e.g., email alerts, logs, or script execution) when limits are exceeded.
Automate Actions: Use Task Scheduler to automate corrective actions such as restarting services or scaling resources when performance issues are detected.
A combination of native Windows tools and automation techniques in order to provides a robust monitoring and alerting solution for Windows servers.

## What key performance metrics should be monitor on network devices,
Bandwidth Utilization
Throughput (bps/pps)
Input/Output Traffic
CPU and Memory Usage
Packet Error Rate
CRC Errors
Dropped Packets
Latency and Jitter
Packet Loss
Interface Status
Bandwidth Errors and Congestion
Temperature and Environmental Metrics
Firewall-Specific Metrics
VLAN and Spanning Tree Protocol (STP) Status

## how I may automate monitoring, and alarming.
1. Simple Network Management Protocol (SNMP)
Enable SNMP on network devices (configure community strings, access controls, etc.).
Set up an SNMP monitoring tool (Nagios).
Define thresholds for key metrics (e.g., CPU utilization > 85%).
When thresholds are breached, SNMP traps can be sent to a network management system (NMS) to trigger alarms.

snmp-server host <monitoring-server-ip> traps version 2c <community-string>
snmp-server enable traps cpu

2. Syslog Monitoring
What it does: Network devices can send system and event logs to a central syslog server.
Automation: Configure syslog alerts based on specific log messages (interface down, high CPU usage).
Configure network devices to forward logs to a syslog server.
Set up alerting rules in the syslog server (e.g., if an interface goes down, send an email alert).
Syslog Configuration (Cisco):
logging host <syslog-server-ip>
logging trap informational

3. NetFlow/IPFIX Monitoring
What it does: NetFlow or IPFIX provides detailed information on IP traffic flows.
Automation: Configure NetFlow on network devices and use a tool to monitor traffic patterns, detect abnormal usage, and set alarms for thresholds like bandwidth utilization.
Example NetFlow Setup (Cisco):
ip flow-export destination <monitoring-server-ip> 9996
ip flow-export version 9
4. sFlow
sFlow samples network traffic and provides real-time traffic statistics.
Automation: sFlow data can be used to monitor traffic trends and detect unusual patterns. Monitoring software (e.g., PRTG, sFlowTrend) can trigger alerts if thresholds are breached.
sFlow Example (on a switch):
sflow enable
sflow destination <monitoring-server-ip>
sflow sample 1000

5. PowerShell & SSH for Device Monitoring
PowerShell scripts (or Python) to SSH into devices and collect metrics periodically.
Automation:
Schedule scripts via Task Scheduler or Cron to run at regular intervals.
The script can query SNMP, run CLI commands via SSH, and analyze the output for anomalies.
$session = New-SSHSession -ComputerName "switch1" -Credential (Get-Credential)
$output = Invoke-SSHCommand -SSHSession $session -Command "show cpu"
if ($output -like "*85%*") {
    # Send an alert email or trigger an action
}
Remove-SSHSession -SSHSession $session
6. Network Monitoring Tools
Nagios: An open-source network monitoring tool that can monitor devices via SNMP, with custom alerting rules.

## how do I analyse these metrics to identify potential bottlenecks or failures KPI 
 Compare Against Thresholds 
 Look for Spikes and Anomalies
 Correlate Metrics Across Devices
 Analyze Traffic Patterns
 Analyze Error Rates and Discards
 Investigate Resource, logs Interface Utilization and Congestion

# VMWare
## Steps to Create and Deploy a New Virtual Machine in a VMware Environment
Log in to VMware vSphere Web Client
Access the VMware vSphere environment using the web client or vSphere client.
Navigate to Hosts and Clusters
In the vSphere Web Client, go to the "Hosts and Clusters" section to view available hosts.
Select a Datacenter and Host
Choose the appropriate datacenter and ESXi host where you want to create the virtual machine (VM).
Create New Virtual Machine
Right-click on the host or cluster and select New Virtual Machine.
Virtual Machine Configuration
Select Creation Type: Choose between typical or custom creation.
Name and Location: Give the VM a name and select the location in the inventory.
Select a Compute Resource: Choose the ESXi host or cluster for the VM.
Select Storage: Pick the datastore where the VM’s files will be stored.
Compatibility: Choose compatibility with specific vSphere versions.
Select a Guest OS: Specify the guest OS type
Configure Hardware Resources
Set CPU, memory, and disk size.
Configure network adapter settings
Add or modify other hardware settings like SCSI controllers, CD/DVD drives.
Finalize VM Creation
Review the summary of the VM configuration and click Finish to create the virtual machine.
Install Guest OS
Power on the VM and connect to the ISO file of the operating system.
Follow the installation steps for the guest OS.
Install VMware Tools
After the OS is installed, install VMware Tools to improve VM performance and manageability.
Configure VM Settings Post-Deployment
Configure additional settings like static IP address, DNS, and other system configurations according to the requirements of the deployed application or service.

## Ensuring Appropriate Resource Allocation for Optimal Performance
CPU Allocation:
Assign the VM the appropriate number of virtual CPUs (vCPUs) based on workload requirements.
Enable CPU hot add if required for flexibility.

Memory Allocation:
Allocate sufficient memory based on the guest OS and expected application usage.
Ensure memory reservation is set if required for consistent performance.

Storage Considerations:
Use high-performance datastores for high I/O workloads.
Use thin provisioning to optimize storage usage but keep an eye on datastore space.

Networking:
Allocate network adapters to use a high-speed network.
Use dedicated VLANs or distributed switches for better network performance.

Resource Pools:
Create resource pools and allocate CPU/memory shares to ensure critical VMs receive necessary resources.

Monitoring:
Continuously monitor VM resource usage using vSphere Performance charts.
Use VMware’s Distributed Resource Scheduler (DRS) to automatically balance resource allocation across hosts.

## Automating VMware Virtual Machine Deployment with Ansible
ansible-playbook deploy_vm.yml --ask-vault-pass


# Antivirus
## How I would automate the management of windows updates
I would use an installation package and deploy it via Group Policy (GPO), or scripts in Ansible/PowerShell.
## capture the necessary information to determine a sucessfull update
Use Windows exporter to query the status of antivirus tools.
Use Ansible/PowerShell to monitor the status of Windows Defender or third-party antivirus tools by querying the system’s health status.
## capture the necessary information to determine a unsuccessful update and the steps to retry the update
Use Windows exporter to query the status of antivirus tools.
Use Ansible/PowerShell to monitor the status of Windows Defender or third-party antivirus tools by querying the system’s health status.

## check sample code automation
