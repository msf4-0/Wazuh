# Wazuh
Wazuh is a free and open source security platform that unifies XDR (Extended Detection and Response) and SIEM (Security Information and Event Management) capabilities. It protects workloads across on-premises, virtualized, containerized, and cloud-based environments.

## Recommended Environment
Ubuntu is recommended to deploy Wazuh Manager. 

For lightweight deployment, use Ubuntu Server for less resource consumption on the OS.

# Wazuh Installation (Version 4.3)
Download the installer
```
curl -sO https://packages.wazuh.com/4.3/wazuh-install.sh
```

Execute the script
```
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
```
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt
```

Installing Wazuh agent to endpoints
1. Access the Wazuh web interface
2. Go to Agents
3. Deploy new agents
4. Select and fill in the data according to the endpoint that needs to install Wazuh agent
5. Run the command shown in step 5 and 6
# Features
