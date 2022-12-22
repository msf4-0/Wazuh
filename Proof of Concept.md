# **Proof of Concept**

## **Detecting Brute Force Attacks**

### **Preparing For The Attack**

> **Note:** If you wish to compare the results of with and without the active response, don't insert the `<active-response>` block into `ossec.conf` of the Wazuh Manager just yet since active response will block off your attack.
---
> Step 1 is done on Wazuh Manager.
>
> Step 2-4 is done on attacking machine.
>
> Step 5 is done on Wazuh Agent.

1. Follow the configuration at [Active Response](README.md#active-response-configuration) on *Wazuh Manager*.
2. On the *attacking device*, install a common brute forcing tool called Hydra:

   ```console
   # apt install -y hydra
   ```

3. Get a common password wordlist:

   ```console
   # curl https://downloads.skullsecurity.org/passwords/500-worst-passwords.txt.bz2 -o password.bz2
   ```

4. Extract the password from the compressed file:

   ```console
   # bzip2 -d password.bz2
   ```

5. Make sure ssh is enabled on the Wazuh Agent:

   ```console
   # apt install -y openssh-server
   # systemctl enable ssh
   # systemctl start ssh
   ```

### **Generating Alert**

1. Use the attack machine to ping the device with Wazuh Agent installed to see if both machine can communicate with each other:

   ```console
   # ping <IP_ADDRESS>
   ```

2. Start the attack from the attack machine:

   ```console
   # hydra -l root -p password <AGENT_IP_ADDRESS> ssh
   ```

3. The alert will be generated on Wazuh Dashboard at Security Events module.

## Detecting SQL Injection

### Install Apache Server

> Apache Server is used as a sample environment.

1. Install `apache2` on machine with Wazuh Agent installed.

   ```console
   # apt update
   # apt install apache2
   ```

2. Configure firewall

   ```console
   # ufw allow 'Apache'
   ```

   To verify the change:

   ```console
   # ufw status
   ```

   Expected output:

   ```console
   Status: active

   To                         Action      From
   --                         ------      ----
   OpenSSH                    ALLOW       Anywhere                  
   Apache                     ALLOW       Anywhere                
   OpenSSH (v6)               ALLOW       Anywhere (v6)             
   Apache (v6)                ALLOW       Anywhere (v6)
   ```

3. Check web server status

   ```console
   # systemctl status apache2
   ```

   Expected output:

   ```console
   ● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor prese>
     Active: active (running) since Tue 2022-04-26 15:33:21 UTC; 43s ago
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 5089 (apache2)
      Tasks: 55 (limit: 1119)
     Memory: 4.8M
        CPU: 33ms
     CGroup: /system.slice/apache2.service
             ├─5089 /usr/sbin/apache2 -k start
             ├─5091 /usr/sbin/apache2 -k start
             └─5092 /usr/sbin/apache2 -k start
   ```

4. Server Access

   The server can be accessed at browser with:

   ```console
   http://<YOUR_IP_ADDRESS>
   ```

### Wazuh Configuration

1. At the machine with Wazuh Agent, add the following block into `/var/ossec/etc/ossec.conf`:

   ```xml
   <localfile>
      <log_format>apache</log_format>
      <location>/var/log/apache2/access.log</location>
   </localfile>
   ```

2. Restart Wazuh Agent

   ```console
   # systemctl restart wazuh-agent
   ```

3. Modify the FilesMatch Directive at `/etc/apache2/apache2.conf`:

   ```xml
   <FilesMatch ".ht*">
      Require all denied
   </FilesMatch>
   ```

### Attempt SQL Injection to Generate Alert

Execute the following from a attacking machine to attempt SQL injection on the machine with Apache2 set up:

```console
curl -XGET "http://REPLACE_IP_ADDRESS>/?id=SELECT+*+FROM+users";
```
