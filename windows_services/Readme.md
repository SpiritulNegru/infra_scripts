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

 
 

Routine Service Management: win_service, win_service_info for checking, starting, stopping, or restarting Windows services.
Software and Feature Installation: win_package for installing applications, win_feature for enabling Windows features like IIS or .NET.
Configuration & Management: win_registry, win_file, win_task, win_hostname, win_shell and win_copy for managing configuration files and registry settings.
Security and User Management: win_user, win_group, and win_firewall_rule to manage users, groups, and firewall settings.
System Updates and Patching: win_updates for managing Windows updates.
System Reboots: win_reboot