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
