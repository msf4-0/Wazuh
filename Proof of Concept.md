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

   Hydra is a tool used for launching parallel and highly automated attacks on login credentials. It has the ability to perform dictionary attack. It is often used by security professionals to test the strength of passwords and authentication systems, but it can also be used by malicious actors to gain unauthorized access to systems and networks.

   Hydra can be used to launch attacks against a variety of protocols, including Telnet, FTP, HTTP, HTTPS, SMB, and many others. It is a highly flexible tool that can be used in a variety of ways, depending on the specific requirements of the target system and the objectives of the attacker.

   A dictionary attack is a type of cyber attack that uses a list of words as a password guessing mechanism. The attacker will take a list of common or likely passwords, known as a "dictionary," and attempt to use each word in the list as the password to gain access to a system. The goal of the attacker is to find a match between one of the passwords in the dictionary and the actual password used by the target system.

   Dictionary attacks are often automated, allowing the attacker to quickly test a large number of passwords in a short amount of time. The efficiency of a dictionary attack is increased by using dictionaries that are specifically tailored to the target system, such as dictionaries that contain words commonly used as passwords in the target's language or industry.

3. Get a common password wordlist:

   ```console
   # curl https://downloads.skullsecurity.org/passwords/500-worst-passwords.txt.bz2 -o password.bz2
   ```

   A wordlist is a list of words, phrases, or other strings that are used in a dictionary attack. In the context of a dictionary attack, the wordlist is used as a list of potential passwords that the attacker will try to use to gain access to a system.

   There are many wordlists available for dictionary attacks, some of the most popular ones are:
   1. RockYou: A wordlist containing over 14 million passwords that were obtained from the RockYou data breach in 2009.
   2. SecLists: A collection of multiple wordlists, including the famous "darkweb2017" and "10-million-password-list-top-1000000".
   3. Crackstation: A massive wordlist of over 1.4 billion hashes, which includes hashes for over 306 million unique passwords.
   4. Kali Linux Wordlists: A collection of wordlists that are included with the popular Kali Linux distribution, including the "common.txt" and "fasttrack.txt" wordlists.
   5. John the Ripper Wordlists: A collection of wordlists that are included with the popular John the Ripper password cracking tool.

   These wordlists can be used with various tools like Hydra, John the Ripper, and other password cracking utilities.

4. Extract the password from the compressed file:

   ```console
   # bzip2 -d password.bz2
   ```

   The extracted file should be called `password`.

5. Make sure ssh is enabled on the Wazuh Agent:

   ```console
   # apt install -y openssh-server
   # systemctl enable ssh
   # systemctl start ssh
   ```

   This attack will be attempting to attack the ssh login of the Wazuh Agent. Make sure the ssh is installed and working for that machine.

### **Generating Alert**

1. Use the attack machine to ping the device with Wazuh Agent installed to see if both machine can communicate with each other:

   ```console
   # ping <IP_ADDRESS>
   ```

   If the IP address of the Wazuh Agent is unkwown, use the command `ifconfig` to show the IP address.

2. Start the attack from the attack machine:

   ```console
   # hydra -l root -p password <AGENT_IP_ADDRESS> ssh
   ```

   `-l` is to specify the username used for the attack. It can either be specified by one usernamne, or a list of username by typing the name of the file.

   `-p` is to specify the password used for the attack. A wordlist is normally provided for the attack. In this case, the wordlist `password` will be used for the attack.

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

   **ufw** stands for "Uncomplicated Firewall" and is a tool used to manage the firewall rules in a Linux system. It provides an easy-to-use interface to configure and manage the firewall, making it a good choice for users who are not familiar with firewall concepts or who prefer a graphical interface.

   **ufw** is built on top of **iptables** and is used to define the policies for incoming and outgoing traffic on a Linux system. It can be used to block unwanted incoming traffic, allow specific incoming traffic, and even forward traffic from one network interface to another.

   Using **ufw** is straightforward and simple. You can enable or disable the firewall, allow or deny incoming traffic, and view the status of the firewall with a few commands. The default policies for incoming and outgoing traffic can be set to either `allow` or `deny`. If you want to change these policies, you can add new rules to the firewall configuration.

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

**curl** is a command-line tool used in Linux to transfer data from or to a server. It stands for "Client for URLs" and is used to transfer data using various protocols, including HTTP, HTTPS, FTP, and others.

**curl** can be used to retrieve the content of a web page, download files, send data to a server via POST requests, and perform other data transfer tasks. It is a flexible and powerful tool that can be used in a variety of ways, making it a common choice for developers and system administrators.

For example, you can use curl to retrieve the content of a web page:

```console
curl http://www.example.com
```

Or you can use curl to download a file:

```console
curl http://www.example.com/file.zip -o file.zip
```

**curl** supports many different options and features, including the ability to set custom headers, follow redirects, and control the rate of data transfer. It is also capable of sending and receiving data in a variety of formats, including plain text, JSON, and XML.

The `-X` or `--request` option in **curl** allows you to specify the type of HTTP request to be sent to the server. The `-X GET` option, which is often abbreviated as -`XGET`, is used to send an HTTP GET request to the server.

The GET request is the most common type of HTTP request and is used to retrieve a resource from the server. When you visit a website in your browser, the browser sends a GET request to the server to retrieve the content of the page.

For example, the following curl command sends a GET request to **<http://www.example.com>**:

```console
curl -XGET http://www.example.com
```
