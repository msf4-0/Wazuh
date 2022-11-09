# Wazuh
Wazuh is a free and open source security platform that unifies XDR (Extended Detection and Response) and SIEM (Security Information and Event Management) capabilities. It protects workloads across on-premises, virtualized, containerized, and cloud-based environments.

## Recommended Environment
Ubuntu is recommended to deploy Wazuh Manager. 

For lightweight deployment, use Ubuntu Server for less resource consumption on the OS.

# Wazuh Installation (Version 4.3)
## Single Node Deployment
Download the installer
```shell
curl -sO https://packages.wazuh.com/4.3/wazuh-install.sh
```

Execute the script
```shell
sudo bash ./wazuh-install.sh -a
```

After successfully running the script, an output containing the credentials to access Wazuh will be printed.
```
INFO: --- Summary ---
INFO: You can access the web interface https://<wazuh-dashboard-ip>
    User: admin
    Password: <ADMIN_PASSWORD>
INFO: Installation finished.
```

All the passwords can be printed with this command
```shell
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

## Cluster Deployment
Edit the configuration file in `/var/ossec/etc/ossec.conf` to configure the `<cluster>` block.
### Master Node
The master node can be configured as follow:
```
<cluster>
    <name>wazuh</name>
    <node_name>master-node</node_name>
    <key>GENERATED_KEY</key>
    <node_type>master</node_type>
    <port>1516</port>
    <bind_addr>0.0.0.0</bind_addr>
    <nodes>
        <node>MASTER_IP_ADDRESS</node>
    </nodes>
    <hidden>no</hidden>
    <disabled>no</disabled>
</cluster>
```
- name: Set the name of the cluster.
- node-name: Set the name of the master node
- key: The key used must be 32 characters and the same key needs to be used across all nodes in the cluster. A key can be generated with the command:
  ```shell
  openssl rand -hex 16  
  ```
- node-type: Set the node type. (Master/Worker)
- port: Set the port used for cluster communication
- bind_addr: Set the IP address of what this node is listening to (Default is 0.0.0.0, meaning any IP)
- nodes: Set the IP address (or DNS name if any) of the master node. This must be specified in every node including the master node.
- hidden: Toggles whether or not to show information about the cluster that generated an alert.
- disabled: Indicates whether the node will be enabled or not in the cluster. Must be set to no if the node needs to be included in a cluster.

More information can be found [here](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/cluster.html#cluster).

Restart the master node:
```shell
sudo systemctl restart wazuh-manager
```

### Worker Node
```
<cluster>
    <name>wazuh</name>
    <node_name>worker01-node</node_name>
    <key>GENERATED_KEY</key>
    <node_type>worker</node_type>
    <port>1516</port>
    <bind_addr>0.0.0.0</bind_addr>
    <nodes>
        <node>MASTER_IP_ADDRESS</node>
    </nodes>
    <hidden>no</hidden>
    <disabled>no</disabled>
</cluster>
```

Restart the worker node:
```shell
sudo systemctl restart wazuh-manager
```

Run the following command to check if the configuration is correct:
```shell
sudo /var/ossec/bin/cluster_control -l
```

The expected output example:
```
NAME           TYPE    VERSION  ADDRESS
master-node    master  4.3.9   wazuh-master
worker01-node  worker  4.3.9   172.22.0.3
```

# Wazuh Agent Installation
## Linux
1. Access the Wazuh web interface
2. Go to Agents
3. Deploy new agents
4. Select and fill in the data according to the endpoint that needs to install Wazuh agent
5. Run the command shown in step 5 and 6

## Windows
### Installer
The installer can be downloaded with the link at [Wazuh Agent Installer](https://packages.wazuh.com/4.x/windows/wazuh-agent-4.3.9-1.msi)

### Powershell
The same can be done at the [Linux]() section in the powershell. Fill in the data with the Windows OS selected and copy the command into powershell to execute.

# Features
## Active Reponse

## Agents In Cluster
