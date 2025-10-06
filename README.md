# Introduction

If you have several Juniper routers, you may want to back up their configurations regularly. This repository contains an Ansible playbook that automates the backup process for Juniper devices.

## Prerequisites

- Ansible installed on your control machine (Linux/MacOS/WSL)
- Access to the Juniper devices with credentials
- SSH key-based authentication set up for secure access
- Basic knowledge of Ansible and YAML syntax

## Installation and Setup

For installation, the following commands will update the repository and install Ansible on your Ansible server.

~~~bash
add-apt-repository --yes --update ppa:ansible/ansible
apt install ansible
~~~

We will create a folder to store the working files.

~~~bash
mkdir ansible
~~~

We will create the necessary config file for Ansible Playbooks.

~~~bash
nano ansible.cfg
~~~

Contents to be written inside the config file:

~~~yaml
[defaults]
inventory = inventory.yaml
private_key_file = ~/.ssh/id_ed25519
callback_whitelist = email_playbook_results
~~~

We will create the necessary Inventory files for Ansible Playbooks.

~~~bash
nano inventory.yaml
~~~

Example inventory.yaml file:

~~~yaml
---
juniper:
  hosts:
    ISP-RTR-2:
      datacenter: DC01
      ansible_host: 10.10.10.1
      user: "juniper-username"
      passwd: "juniper-password"
    ISP-RTR-1:
      datacenter: DC02
      ansible_host: 10.10.20.1
      user: "juniper-username"
      passwd: "juniper-password"
    BB-RTR-1:
      datacenter: DC03
      ansible_host: 10.10.30.1
      user: "juniper-username"
      passwd: "juniper-password"
~~~

## Inventory content explanation

- Juniper: // Used only for naming.
- ISP-RTR-2: // Hostname of the Juniper device.
- datacenter: DC01 // Custom variable to identify the data center location.
- ansible_host: IP address of the Juniper device.

You can add multiple Juniper devices by following the same structure in the inventory file. Make sure to replace the placeholder values with your actual device details and credentials.

## Running the Playbook

To run the playbook and back up the configurations of all Juniper devices listed in the inventory file, use the following command:

~~~bash
ansible-playbook -i inventory.yaml juniper-backup-playbook.yml
~~~

This command will execute the playbook and create backup files for each Juniper device in the specified directory.

### Playbook Variables

You can change below variables in the playbook as per your requirements.

~~~bash
  vars:
    dest_path: "/root/{{ datacenter }}"
    folder: "{{ dest_path }}/{{ inventory_hostname }}/{{ hostvars['localhost']['backup_date'] }}"
    filename: "{{ folder }}/backup_{{ hostvars['localhost']['backup_date'] }}_{{ hostvars['localhost']['backup_time'] }}.yaml"
    latest_file: "{{ dest_path }}/{{ inventory_hostname }}/latest/latest.yaml"
~~~

- dest_path: // Base directory where backups will be stored. You can customize it using the datacenter variable.
- folder: // Directory structure for each backup, organized by device hostname and date.
- filename: // Naming convention for the backup files, including date and time.
- latest_file: // Path to the latest backup file for comparison.

You can customize these variables to fit your directory structure and naming preferences.

## Playbook Explanation

In brief, the playbook first checks for the existence of the backup directories and creates them if they do not exist.

Then, it uses the Juniper credentials to take a backup and saves it as **latest**. It also compares the new backup with the previous one and stores the differences in a **compare** file. This way, you can easily see the changes between configurations.

It backs up all VDOMs on the Juniper. If desired, you can filter specific VDOMs or mask passwords in the backup. However, if masking is applied, the backup file cannot be directly uploaded in case of an issue.  

## Callback Plugin for Email Notifications

- The repository includes a custom callback plugin (`email_playbook_results.py`) that sends email notifications with the results of playbook executions.
- Update the email addresses and SMTP server details in the plugin as needed.
- Ensure that the callback plugin is placed in the `callback_plugins` directory and that Ansible is configured to use it.

### Example Email Output

~~~
Starting task: Backing up Junipers' committed config
Task succeeded on RACK-O1-ISP-RTR-1: Backing up Junipers' committed config
Task succeeded on RACK-O1-ISP-RTR-2: Backing up Junipers' committed config
Task succeeded on RTR-1: Backing up Junipers' committed config
Task succeeded on RTR-2: Backing up Junipers' committed config
~~~

## Security Considerations

- Ensure that sensitive information such as passwords and API keys are managed securely, using Ansible Vault or environment variables.
- Regularly update Ansible and related dependencies to mitigate security vulnerabilities.
- Use secure methods for storing and transmitting backup files, especially if they contain sensitive configuration data.

## Contributions

Contributions to enhance the playbook or add new features are welcome. Please fork the repository and submit a pull request with your changes.
