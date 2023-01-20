# **Wazuh**

Wazuh is a free and open source security platform that unifies XDR (Extended Detection and Response) and SIEM (Security Information and Event Management) capabilities. It protects workloads across on-premises, virtualized, containerized, and cloud-based environments.

## **Recommended Environment**

Ubuntu is recommended to deploy Wazuh Manager.

For lightweight deployment, use Ubuntu Server for less resource consumption on the OS.

## **Wazuh Installation (Version 4.3)**

### **Single Node Deployment**

Download the installer

```console
# curl -sO https://packages.wazuh.com/4.3/wazuh-install.sh
```

Execute the script

```console
# bash ./wazuh-install.sh -a
```

After successfully running the script, an output containing the credentials to access Wazuh will be printed.

```console
INFO: --- Summary ---
INFO: You can access the web interface https://<wazuh-dashboard-ip>
    User: admin
    Password: <ADMIN_PASSWORD>
INFO: Installation finished.
```

All the passwords can be printed with this command

```console
# tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

### **Cluster Deployment**

Edit the configuration file in `/var/ossec/etc/ossec.conf` to configure the `<cluster>` block.

#### **Master Node**

The master node can be configured as follow:

```xml
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

  ```console
  # openssl rand -hex 16  
  ```

- node-type: Set the node type. (Master/Worker)
- port: Set the port used for cluster communication
- bind_addr: Set the IP address of what this node is listening to (Default is 0.0.0.0, meaning any IP)
- nodes: Set the IP address (or DNS name if any) of the master node. This must be specified in every node including the master node.
- hidden: Toggles whether or not to show information about the cluster that generated an alert.
- disabled: Indicates whether the node will be enabled or not in the cluster. Must be set to no if the node needs to be included in a cluster.

More information can be found [here](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/cluster.html#cluster).

Restart the master node:

```console
# systemctl restart wazuh-manager
```

#### **Worker Node**

```xml
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

```console
# systemctl restart wazuh-manager
```

Run the following command to check if the configuration is correct:

```console
# /var/ossec/bin/cluster_control -l
```

The expected output example:

```console
NAME           TYPE    VERSION  ADDRESS
master-node    master  4.3.9   wazuh-master
worker01-node  worker  4.3.9   172.22.0.3
```

### **Docker Deployment**

#### **Setup Docker**

##### **Set Container Memory**

Increase max_map_count on your host witht the following command:

```console
# sysctl -w vm.max_map_count=262144
```

Permenantly set the value by setting `vm.max_map_count=262144` in `/etc/sysctl.conf`.

> **Note:** If the max_map_count is not set, Wazuh Indexer will not function properly.

##### **Docker Engine**

Run Docker installation script

```console
# curl -sSL https://get.docker.com/ | sh
```

Start docker service

```console
# systemctl enable docker
# systemctl start docker
```

##### **Docker Compose**

Download Docker Compose binary

```console
# curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Grant execution permissions

```console
# chmod +x /usr/local/bin/docker-compose
```

> **Note:** If docker-compose fail to start after installation, a symlink can be created.

```console
# ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Test if the installation is complete

```console
# docker-compose --version
```

#### **Single-Node Deployment**

Clone the repository

```consolele
git clone https://github.com/wazuh/wazuh-docker.git -b v4.3.9 --depth=1
```

Change directory into `single-node`

```bash
cd wazuh-docker/single-node/
```

Generate certificates to secure traffic between nodes

```bash
docker-compose -f generate-indexer-certs.yml run --rm generator
```

Start Wazuh using docker-compose

```bash
docker-compose up -d
```

#### **Multi-Node Deployment**

Clone the repository

```bash
git clone https://github.com/wazuh/wazuh-docker.git -b v4.3.9 --depth=1
```

Change directory into `multi-node`

```bash
cd wazuh-docker/multi-node/
```

Generate certificates to secure traffic between nodes

```bash
docker-compose -f generate-indexer-certs.yml run --rm generator
```

Start Wazuh using docker-compose

```bash
docker-compose up -d
```

## **Wazuh Agent Installation**

### **Linux**

1. Access the Wazuh web interface
2. Go to Agents
3. Deploy new agents
4. Select and fill in the data according to the endpoint that needs to install Wazuh agent
5. Run the command shown in step 5 and 6

### **Windows**

#### **Installer**

The installer can be downloaded with the link at [Wazuh Agent Installer](https://packages.wazuh.com/4.x/windows/wazuh-agent-4.3.9-1.msi)

#### **Powershell**

The same can be done at the [Linux](#linux) section in the powershell. Fill in the data with the Windows OS selected and copy the command into powershell to execute.

### **Agents In Cluster**

Pointing the agent to the cluster node

```xml
<client>
    <server>
        <address>WORKER_NODE_IP</address>
    </server>
</client>
```

## **Wazuh Manager & Wazuh Agent Uninstall**

### **Wazuh Manager**

In the case where the Wazuh Manager is needed to be uninstalled, go to the directory where the `wazuh-install.sh` was stored. Then use the `-u` flag to uninstall the Wazuh Manager as follow:

```console
# bash wazuh-install.sh -u
```

### **Wazuh Agent**

If the Wazuh Agent on a certain machine is needed to be uninstalled, use the following command on the machine where the Wazuh Agent is installed:

```console
# sudo apt remove wazuh-agent
```

This will only uninstall the Wazuh Agent on the machine, but it will not update the Wazuh Manager that it was uninstalled so on the Wazuh Dashboard, it will only show the agent is disconnected. To fully remove it on the dashboard, run the following command:

```console
# cd /var/ossec/bin
# ./manage_agents
```

The expected output would look like:

```console
****************************************
* Wazuh v4.3.10 Agent manager.         *
* The following options are available: *
****************************************
   (A)dd an agent (A).
   (E)xtract key for an agent (E).
   (L)ist already added agents (L).
   (R)emove an agent (R).
   (Q)uit.
Choose your action: A,E,L,R or Q:
```

Type in `r` or `R` (not cap sensitive) where the output would look like:

```console
Available agents:
   ID: 001, Name: wazuh-agent, IP: any
   ID: 003, Name: donkodile, IP: any
Provide the ID of the agent to be removed (or '\q' to quit):
```

Provide the agent ID that is needed to be removed so that it will not show the agent is disconnected after uninstall on the Wazuh Dashboard.

## **Features**

### **Active Reponse**

When specific requirements are satisfied, active responses use different countermeasures to handle current threats, such as limiting access to an agent from the threat source.

Active responses run a script in response to certain alerts being triggered based on the alert level or rule group. In response to a trigger, any number of scripts can be launched; however, these replies must be carefully evaluated. Poor rule and response implementation may enhance the system's susceptibility.

#### **Active Response Configuration**

> The `command` already exists in `ossec.conf`. However, the `active-response` has to be added manuanlly into `ossec.conf`.

Command used:

```xml
<command>
  <name>firewall-drop</name>
  <executable>firewall-drop</executable>
</command>
```

Active Reponse:

```xml
<active-response>
  <command>firewall-drop</command>
  <location>all</location>
  <rules_group>authentication_failed|authentication_failures</rules_group>
  <timeout>700</timeout>
  <repeated_offenders>30,60,120</repeated_offenders>
</active-response>
```

> Other attributes that can be added in `active-response` can be found [here](https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/active-response.html#reference-ossec-active-response).

### **File Integrity Monitoring**

Wazuh's File Integrity Monitoring (FIM) system monitors certain files and sends notifications when they are updated. The component in charge of this task is known as syscheck. This component maintains the cryptographic checksum and other information of files or Windows registry keys and compares them on a regular basis with the system's current files, looking for changes.

#### **Real-time Monitoring Configuration**

Real-time monitoring is configured with the `realtime` attribute of the directories option. This attribute only works with the directories rather than with the individual files. Real-time change detection is paused during periodic syscheck scans and reactivates as soon as these scans are complete.

```xml
<syscheck>
  <directories check_all="yes" realtime="yes">/path/to/directory</directories>
</syscheck>
```

### **Vulnerabilities Detection**

Wazuh is able to detect vulnerabilities in the applications installed on agents using the Vulnerability Detector module. This software audit is performed through the integration of vulnerability feeds indexed by Canonical, Debian, Red Hat, Arch Linux, ALAS (Amazon Linux Advisories Security), Microsoft, and the National Vulnerability Database.

#### **Vulnerability Detection Configuration**

Enable `vulnerability-detector` in `ossec.conf` at Wazuh Manager.

```xml
<ossec_config>
<vulnerability-detector>
    <enabled>yes</enabled>
    <interval>5m</interval>
    <ignore_time>6h</ignore_time>
    <run_on_start>yes</run_on_start>

    <!-- Ubuntu OS vulnerabilities -->
    <provider name="canonical">
    <enabled>yes</enabled>
    <os>trusty</os>
    <os>xenial</os>
    <os>bionic</os>
    <os>focal</os>
    <os>jammy</os>
    <update_interval>1h</update_interval>
    </provider>

    <!-- Debian OS vulnerabilities -->
    <provider name="debian">
    <enabled>yes</enabled>
    <os>stretch</os>
    <os>buster</os>
    <os>bullseye</os>
    <update_interval>1h</update_interval>
    </provider>

    <!-- RedHat OS vulnerabilities -->
    <provider name="redhat">
    <enabled>yes</enabled>
    <os>5</os>
    <os>6</os>
    <os>7</os>
    <os>8</os>
    <os allow="Centos Linux-8">8</os>
    <os>9</os>
    <update_interval>1h</update_interval>
    </provider>

    <!-- Windows OS vulnerabilities -->
    <provider name="msu">
    <enabled>yes</enabled>
    <update_interval>1h</update_interval>
    </provider>

    <!-- Aggregate vulnerabilities -->
    <provider name="nvd">
    <enabled>yes</enabled>
    <update_from_year>2010</update_from_year>
    <update_interval>1h</update_interval>
    </provider>

</vulnerability-detector>
</ossec_config>
```

If the Wazuh Agent is installed in a Windows endpoint, configure the file at `C:\Program Files (x86)\ossec-agent\ossec.conf` to enable `hotfixes` and `packages` collection in the syscollector component:

```xml
<wodle name="syscollector">
    <disabled>no</disabled>
    <interval>1h</interval>
    <scan_on_start>yes</scan_on_start>
    <hardware>yes</hardware>
    <os>yes</os>
    <network>yes</network>
    <packages>yes</packages>
    <hotfixes>yes</hotfixes>
    <ports all="no">yes</ports>
    <processes>yes</processes>
</wodle>
```

If the Wazuh Agent is installed in a Linux endpoint, configure the file at `/var/ossec/etc/ossec.conf` to enable software `packages` collection in the syscollector component.

```xml
<wodle name="syscollector">
    <disabled>no</disabled>
    <interval>1h</interval>
    <scan_on_start>yes</scan_on_start>
    <hardware>yes</hardware>
    <os>yes</os>
    <network>yes</network>
    <packages>yes</packages>
    <ports all="no">yes</ports>
    <processes>yes</processes>
<   /wodle>
```

### **VirusTotal Integration**

> **Configuration added soon**

Wazuh can scan monitored files for malicious content in monitored files. This solution is possible through an integration with VirusTotal, which is a powerful platform that aggregates multiple antivirus products along with an online scanning engine. Combining this tool with our FIM engine provides a simple means of scanning the files that are monitored to inspect them for malicious content.

#### **VirusTotal Integration Configuration**

> **Note:**  
>
> - VirusTotal is an external API so there is a API request limit.
> - Only monitor important directories such as `/root`, or `/home/user/Downloads` to directly scan for malware as soon as the file is downloaded.
> - Requires Internet connection.

Enable File Integrity Monitoring in `/var/ossec/etc/ossec.conf` of the Wazuh Agent to monitor `/root` in real time.

```xml
<syscheck>
    <directories whodata="yes">/root</directories>
</syscheck>
```

At the Wazuh Manager, add the following rules to `/var/ossec/etc/rules/local_rules.xml` to generate an alert if a file is added or modified in the `/root` directory.

```xml
<group name="syscheck,pci_dss_11.5,nist_800_53_SI.7,">
    <!-- Rules for Linux systems -->
    <rule id="100200" level="7">
        <if_sid>550</if_sid>
        <field name="file">/root</field>
        <description>File modified in /root directory.</description>
    </rule>
    <rule id="100201" level="7">
        <if_sid>554</if_sid>
        <field name="file">/root</field>
        <description>File added to /root directory.</description>
    </rule>
</group>
```

Add the following configuration to the `/var/ossec/etc/ossec.conf` file at the Wazuh manager. Replace `YOUR_VIRUS_TOTAL_API_KEY` with your own API key. An API key can be obtained by signing up at [VirusTotal](https://www.virustotal.com/gui/join-us). Then head to this [page](https://www.virustotal.com/gui/my-apikey) to get an API key.

```xml
<ossec_config>
  <integration>
    <name>virustotal</name>
    <api_key>YOUR_VIRUS_TOTAL_API_KEY</api_key> <!-- Replace with your VirusTotal API key -->
    <rule_id>100200,100201</rule_id>
    <alert_format>json</alert_format>
  </integration>
</ossec_config>
```

Create a `/var/ossec/active-response/bin/remove-threat.sh` active response script at the endpoint for the removal of a file from the system. Copy the [remove-threat.sh](/remove-threat.sh) script into the created file.

Change owner and file permissions of `/var/ossec/active-response/bin/remove-threat.sh`.

```console
# chmod 750 /var/ossec/active-response/bin/remove-threat.sh
# chown root:wazuh /var/ossec/active-response/bin/remove-threat.sh
```

Install jq to allow the script to process JSON input:

```console
# apt-get install jq -y
```

Add the command and active response into Wazuh Manager `ossec.conf` to enable the manager to use `remove-threat.sh` script when VirusTotal query results for threat is a positive match.

```xml
<ossec_config>
    <command>
        <name>remove-threat</name>
        <executable>remove-threat.sh</executable>
        <timeout_allowed>no</timeout_allowed>
    </command>

    <active-response>
        <disabled>no</disabled>
        <command>remove-threat</command>
        <location>local</location>
        <rules_id>87105</rules_id>
    </active-response>
</ossec_config>
```

Edit `/var/ossec/etc/decoders/local_decoder.xml` at Wazuh Manager and add the active response decoder configuration.

```xml
<decoder name="ar_log_fields">
    <parent>ar_log</parent>
    <regex offset="after_parent">^(\S+) Removed threat located at (\S+)</regex>
    <order>script_name, path</order>
</decoder>
```

Add rules to `/var/ossec/etc/rules/local_rules.xml` at Wazuh Manager to get alert about the active response results.

```xml
<group name="virustotal,">
    <rule id="100092" level="12">
        <if_sid>657</if_sid>
        <match>Successfully removed threat</match>
        <description>$(parameters.program) removed threat located at $(parameters.alert.data.virustotal.source.file)</description>
    </rule>

    <rule id="100093" level="12">
        <if_sid>657</if_sid>
        <match>Error removing threat</match>
        <description>Error removing threat located at $(parameters.alert.data.virustotal.source.file)</description>
    </rule>
</group>
```

Restart Wazuh Agent to apply configuration.

```console
# systemctl restart wazuh-agent
```

Restart Wazuh Manager to apply configuration.

```console
# systemctl restart wazuh-manager
```

#### **Generating Alert for VirusTotal**

Download a malicious test file (it is harmless but gets recognized by various antivirus software) to `/root` in the endpoint. VirusTotal will detect it as maliciouis and the active response will remove the file that was downloaded.

```console
# cd /root
# curl -LO http://www.eicar.org/download/eicar.com && ls -lah eicar.com
```

An alert should be generated in the Wazuh Dashboard in `Security events` module.
