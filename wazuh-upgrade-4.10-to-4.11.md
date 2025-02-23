# Upgrading Wazuh from 4.10.1 to 4.11.0

**Table of Contents**

1.  [Introduction](#introduction)
2.  [Prerequisites](#prerequisites)
3.  [Snapshot (Backup)](#snapshot-backup)
4.  [Upgrade Process](#upgrade-process)
    * [Stopping Services](#stopping-services)
    * [Upgrading Wazuh Indexer](#upgrading-wazuh-indexer)
    * [Starting Wazuh Manager](#starting-wazuh-manager)
    * [Upgrading Wazuh Manager](#upgrading-wazuh-manager)
    * [Configuring Filebeat](#configuring-filebeat)
    * [Upgrading Wazuh Dashboard](#upgrading-wazuh-dashboard)
    * [Verifying Installed Versions](#verifying-installed-versions)
5.  [Upgrading Wazuh Agents](#upgrading-wazuh-agents)
6.  [Verification](#verification)
7.  [Conclusion](#conclusion)

## Introduction <a name="introduction"></a>

This guide provides a step-by-step process for upgrading Wazuh from version 4.10.1 to 4.11.0. Wazuh 4.11.0 introduces various enhancements and bug fixes, as detailed in the official release notes: [Wazuh 4.11.0 Release Notes](https://documentation.wazuh.com/current/release-notes/release-4-11-0.html). 

This guide assumes you are running Wazuh on a Proxmox virtual environment.

## Prerequisites <a name="prerequisites"></a>

* Access to the Wazuh server via SSH.
* Root or sudo privileges.
* A running Wazuh 4.10.1 installation.
* Basic understanding of Linux command-line operations.
* Proxmox access for snapshot creation (if applicable).

## Snapshot (Backup) <a name="snapshot-backup"></a>

Before proceeding with the upgrade, it is crucial to create a snapshot or backup of your Wazuh server. This allows for a quick rollback in case of any issues.

1.  In the Proxmox interface, navigate to your Wazuh server.
2.  Select "Snapshots."
3.  Click "Take snapshot."
![image](https://github.com/user-attachments/assets/da5639f7-66c8-40df-a207-5105cbfcaf72)
   
5.  Provide a descriptive name for the snapshot (e.g., "Wazuh-4.10.1-pre-upgrade").
6.  Click "Take Snapshot."

![image](https://github.com/user-attachments/assets/a64d2f6b-7ca4-49c2-9094-7fe693d228ed)


## Upgrade Process <a name="upgrade-process"></a>

### Stopping Services <a name="stopping-services"></a>

1.  Connect to your Wazuh server via SSH.
2.  Update the package information:

    ```bash
    sudo apt-get update
    ```
![image](https://github.com/user-attachments/assets/f3f1e022-6c1f-4009-9f04-04f2fc1c7593)

3.  Stop the necessary Wazuh services:

    ```bash
    sudo systemctl stop filebeat
    sudo systemctl stop wazuh-dashboard
    sudo systemctl stop wazuh-manager
    sudo systemctl stop wazuh-indexer
    ```

### Upgrading Wazuh Indexer <a name="upgrading-wazuh-indexer"></a>

1.  Upgrade the Wazuh Indexer:

    ```bash
    sudo apt-get install wazuh-indexer
    ```
![image](https://github.com/user-attachments/assets/3d33e6f7-fa7b-4607-86f5-b752fa7f2ebf)

2.  Run the following commands to start the indexer service and enable it to auto start . You can check the status of the service with the last command

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable wazuh-indexer
    sudo systemctl start wazuh-indexer
    sudo systemctl status wazuh-indexer
    ```
![image](https://github.com/user-attachments/assets/b779bf30-f8ab-4407-a11a-c07a540da18a)

### Starting Wazuh Manager <a name="starting-wazuh-manager"></a>

1. Start the wazuh-manager

    ```bash
    sudo systemctl start wazuh-manager
    ```
    ![image](https://github.com/user-attachments/assets/bfb7c54c-c191-4ab8-9e4f-c092a6e3f61a)


### Upgrading Wazuh Manager <a name="upgrading-wazuh-manager"></a>

1.  Upgrade the Wazuh Manager:

    ```bash
    sudo apt-get install wazuh-manager
    ```
![image](https://github.com/user-attachments/assets/65be04f9-55da-45fc-afdc-cc731b12dcc7)

### Configuring Filebeat <a name="configuring-filebeat"></a>

1.  Download the Filebeat module:

    ```bash
    curl -s [https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.4.tar.gz](https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.4.tar.gz) | sudo tar -xvz -C /usr/share/filebeat/module
    ```
![image](https://github.com/user-attachments/assets/630a7b92-4fbb-4dde-a12f-62165502ce44)

2.  Download the alerts template:

    ```bash
    curl -so /etc/filebeat/wazuh-template.json [https://raw.githubusercontent.com/wazuh/wazuh/v4.11.0/extensions/elasticsearch/7.x/wazuh-template.json](https://raw.githubusercontent.com/wazuh/wazuh/v4.11.0/extensions/elasticsearch/7.x/wazuh-template.json)
    sudo chmod go+r /etc/filebeat/wazuh-template.json
    ```
![image](https://github.com/user-attachments/assets/1dfbf25b-0188-4a85-ba3f-97d101306e3c)

3.  Restart Filebeat:

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable filebeat
    sudo systemctl start filebeat
    ```

4.  Upload the new template and pipelines:

    ```bash
    sudo filebeat setup --pipelines
    sudo filebeat setup --index-management -E output.logstash.enabled=false
    ```

   ![image](https://github.com/user-attachments/assets/f94c27d1-473d-48e0-b1d2-374d1f465190)


### Upgrading Wazuh Dashboard <a name="upgrading-wazuh-dashboard"></a>

1.  Backup the existing dashboard configuration:

    ```bash
    sudo cp /etc/wazuh-dashboard/opensearch_dashboards.yml /etc/wazuh-dashboard/opensearch_dashboards.yml.old
    ```

2.  Upgrade the Wazuh Dashboard:

    ```bash
    sudo apt-get install wazuh-dashboard
    ```
![image](https://github.com/user-attachments/assets/1de4cd09-b3ff-446e-892b-46df795e6904)

3.  Manually reapply any custom configurations to `/etc/wazuh-dashboard/opensearch_dashboards.yml`.

    * Ensure `server.ssl.key` and `server.ssl.certificate` match the files in `/etc/wazuh-dashboard/certs/`.
    * Verify that `uiSettings.overrides.defaultRoute` is set to `/app/wz-home`:

        ```yaml
        uiSettings.overrides.defaultRoute: /app/wz-home
        ```
![image](https://github.com/user-attachments/assets/e4c15754-1324-424c-9939-d3d60154cea5)

4.  Restart and check the status of the Wazuh Dashboard:

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable wazuh-dashboard
    sudo systemctl start wazuh-dashboard
    sudo systemctl status wazuh-dashboard
    ```
![image](https://github.com/user-attachments/assets/d6dfbcad-764b-4857-874c-b4600eee623b)

5.  Access the Wazuh Dashboard at `https://<DASHBOARD_IP_ADDRESS>/app/wz-home` (replace `<DASHBOARD_IP_ADDRESS>` with your server's IP).
![image](https://github.com/user-attachments/assets/816ef064-ad94-4388-b085-6a15676f7029)
### Verifying Installed Versions <a name="verifying-installed-versions"></a>

1.  Verify the installed versions:

    ```bash
    apt list --installed wazuh-indexer
    apt list --installed wazuh-manager
    apt list --installed wazuh-dashboard
    ```
![image](https://github.com/user-attachments/assets/5a590bb5-399e-4afc-a32e-23e22abab313)


## Upgrading Wazuh Agents <a name="upgrading-wazuh-agents"></a>

1.  List outdated agents:

    ```bash
    /var/ossec/bin/agent_upgrade -l
    ```
![image](https://github.com/user-attachments/assets/b1c9cd79-204e-4422-b89b-2d5b496d63ef)


2.  Upgrade a single agent (replace `ID` with the agent ID):

    ```bash
    /var/ossec/bin/agent_upgrade -a ID
    ```

3.  For multiple agents, use the provided script:

    ```bash
    touch upgrade_wazuh_agents.sh && chmod +x upgrade_wazuh_agents.sh
    vi upgrade_wazuh_agents.sh
    ```

    Paste the following script content:

    ```bash
    #!/bin/bash

    outdated_agents=$(/var/ossec/bin/agent_upgrade -l | awk 'NR>1 && $1 != "Total" && NF > 0 {print $1}')

    if [[ -n "$outdated_agents" ]]; then
      echo "Outdated agents found. Upgrading..."

      while IFS= read -r agent_id; do
        echo "Upgrading agent ID: $agent_id"
        /var/ossec/bin/agent_upgrade -a "$agent_id"
        if [ $? -eq 0 ]; then
           echo "Agent ID: $agent_id upgraded successfully."
        else
           echo "Agent ID: $agent_id upgrade failed."
        fi

      done <<< "$outdated_agents"

      echo "Agent upgrade process completed."
    else
      echo "No outdated agents found."
    fi

    exit 0
    ```

4.  Run the script:

    ```bash
    sudo ./upgrade_wazuh_agents.sh
    ```
![image](https://github.com/user-attachments/assets/c6c97141-b408-4bb6-8a8d-c71bc28547a7)

## Verification <a name="verification"></a>

* Verify the upgraded versions in the Wazuh Dashboard.
* Check the agent status in the Wazuh Dashboard to confirm successful upgrades.
* Check the Wazuh indexer, manager and dashboard logs for any errors.
![image](https://github.com/user-attachments/assets/5c98e27a-2c02-4686-9377-df21688af328)

## Conclusion <a name="conclusion"></a>

You have successfully upgraded your Wazuh installation from 4.10
