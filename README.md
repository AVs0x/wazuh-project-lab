# wazuh-project-lab
![Wazuh](https://wroberts.me/wp-content/uploads/2022/11/wazuh-666x628.webp "Wazuh Dashboard")
This guide aims to help you set up and create Wazuh dashboards for monitoring and managing security operations. The guide includes detailed instructions on installation, configuration, and use cases for Wazuh.

## Table of Contents
1. [Introduction](#introduction)
2. [Installing Wazuh](#installing-wazuh)
   - [Wazuh Manager](#wazuh-manager)
   - [Wazuh Agent](#wazuh-agent)
3. [Agent Enrollment](#agent-enrollment)
   - [Windows Agent Enrollment](#windows-agent-enrollment)
   - [Linux Agent Enrollment](#linux-agent-enrollment)
4. [Use Cases](#use-cases)
   - [File Integrity Monitoring (FIM)](#file-integrity-monitoring-fim)
   - [Network Intrusion Detection with Suricata](#network-intrusion-detection-with-suricata)
   - [Vulnerability Detection](#vulnerability-detection)
   - [Malicious Command Execution Detection](#malicious-command-execution-detection)
   - [SSH Brute Force Attack Detection and Blocking](#ssh-brute-force-attack-detection-and-blocking)
   - [Malicious File Detection with VirusTotal](#malicious-file-detection-with-virustotal)
5. [Accessing the Dashboard](#accessing-the-dashboard)
6. [Resources](#resources)

---

## Introduction
Wazuh is an open-source security platform designed for real-time monitoring, threat detection, and compliance management. This guide covers the steps to install, configure, and visualize data using Wazuh.

---

## Installing Wazuh

### Wazuh Manager
1. Add the Wazuh repository and GPG key:
   ```bash
   curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
   echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
   ```
2. Update and install the Wazuh Manager:
   ```bash
   sudo apt update
   sudo apt install wazuh-manager -y
   ```
3. Start and enable the Wazuh Manager service:
   ```bash
   sudo systemctl start wazuh-manager
   sudo systemctl enable wazuh-manager
   ```

### Wazuh Agent
1. Install the Wazuh Agent:
   ```bash
   sudo apt install wazuh-agent -y
   ```
2. Start and enable the Wazuh Agent service:
   ```bash
   sudo systemctl start wazuh-agent
   sudo systemctl enable wazuh-agent
   ```

---

## Agent Enrollment

### Windows Agent Enrollment
1. Download the Windows Agent installer from the [Wazuh Downloads page](https://documentation.wazuh.com/current/installation-guide/installing-wazuh-agent/wazuh_agent_windows.html).
2. Install the agent and configure it to connect to the Wazuh Manager.
3. Restart the agent service to apply the changes.

### Linux Agent Enrollment
1. Edit the agent configuration file:
   ```bash
   sudo nano /var/ossec/etc/ossec.conf
   ```
2. Add the manager’s IP address under the `<client>` tag:
   ```xml
   <client>
     <server>
       <address>[manager_ip]</address>
       <port>1514</port>
     </server>
   </client>
   ```
3. Restart the agent:
   ```bash
   sudo systemctl restart wazuh-agent
   ```

---

## Use Cases

### File Integrity Monitoring (FIM)
#### Objective:
Detect unauthorized file changes in specific directories.

#### Steps:
1. Edit the configuration file:
   ```bash
   sudo nano /var/ossec/etc/ossec.conf
   ```
2. Add the following under the `<syscheck>` tag:
   ```xml
   <directories check_all="yes" realtime="yes">/root</directories>
   ```
3. Restart Wazuh services:
   - For Docker:
     ```bash
     service wazuh-manager restart
     ```
   - For direct server installation:
     ```bash
     systemctl restart wazuh-manager
     ```
4. Test the configuration:
   - Add a new file in the monitored directory:
     ```bash
     touch /root/sample-file.txt
     ```
   - Check the Wazuh dashboard for alerts under the "Security Events" tab.

### Network Intrusion Detection with Suricata
#### Objective:
Detect network-based attacks using Suricata IDS.

#### Steps:
1. Install Suricata:
   ```bash
   sudo apt update
   sudo apt install suricata -y
   ```
2. Download and extract Emerging Threats rules:
   ```bash
   wget https://rules.emergingthreats.net/open/suricata-5.0/emerging.rules.tar.gz
   tar -xvzf emerging.rules.tar.gz -C /etc/suricata/rules/
   ```
3. Edit the Suricata configuration file:
   ```bash
   sudo nano /etc/suricata/suricata.yaml
   ```
   Update the following:
   ```yaml
   default-rule-path: /etc/suricata/rules
   home-net: "[your_network_IP_range]"
   ```
4. Restart Suricata:
   ```bash
   systemctl restart suricata
   ```
5. Integrate Suricata logs with Wazuh:
   ```xml
   <localfile>
     <log_format>json</log_format>
     <location>/var/log/suricata/eve.json</location>
   </localfile>
   ```
6. Restart the Wazuh Agent:
   ```bash
   systemctl restart wazuh-agent
   ```
7. Test the configuration:
   - Launch a network scan using nmap:
     ```bash
     nmap -sS [target_IP]
     ```
   - Check the Wazuh dashboard for alerts.

### Vulnerability Detection
#### Objective:
Detect vulnerabilities on monitored endpoints.

#### Steps:
1. Enable vulnerability detection:
   ```xml
   <vulnerability-detector>
     <enabled>yes</enabled>
     <provider>ubuntu</provider>
   </vulnerability-detector>
   ```
2. Restart the Wazuh Manager:
   ```bash
   systemctl restart wazuh-manager
   ```
3. Check vulnerabilities:
   - Go to the Wazuh dashboard and navigate to "Vulnerabilities" to view detected issues.

### Malicious Command Execution Detection
#### Objective:
Detect malicious commands executed by privileged users.

#### Steps:
1. Install AuditD:
   ```bash
   sudo apt install auditd -y
   ```
2. Edit AuditD rules:
   ```bash
   sudo nano /etc/audit/audit.rules
   ```
   Add the following rule:
   ```bash
   -a always,exit -F arch=b64 -S execve -F euid=0 -k root_commands
   ```
3. Restart AuditD:
   ```bash
   systemctl restart auditd
   ```
4. Integrate AuditD logs with Wazuh:
   ```xml
   <localfile>
     <log_format>audit</log_format>
     <location>/var/log/audit/audit.log</location>
   </localfile>
   ```
5. Restart the Wazuh Agent:
   ```bash
   systemctl restart wazuh-agent
   ```
6. Test the configuration:
   - Execute a command as root:
     ```bash
     sudo netstat -an
     ```
   - Check the Wazuh dashboard for alerts.

### SSH Brute Force Attack Detection and Blocking
#### Objective:
Detect and block SSH brute force attacks using active response.

#### Steps:
1. Enable active response:
   ```xml
   <active-response>
     <command>firewalldrop</command>
     <location>local</location>
     <rules_id>5710</rules_id>
   </active-response>
   ```
2. Restart the Wazuh Manager:
   ```bash
   systemctl restart wazuh-manager
   ```
3. Test the configuration:
   - Launch a brute force attack using Hydra:
     ```bash
     hydra -l root -P passwords.txt ssh://[target_IP]
     ```
   - Check the Wazuh dashboard for alerts and verify that the attacker’s IP is blocked.

### Malicious File Detection with VirusTotal
#### Objective:
Detect malicious files using VirusTotal integration.

#### Steps:
1. **Enable File Integrity Monitoring:**
   - Edit the configuration file located at `/var/ossec/etc/ossec.conf`:
     ```xml
     <syscheck>
       <directories check_all="yes" realtime="yes">/root</directories>
     </syscheck>
     ```

2. **Obtain VirusTotal API Key:**
   - Sign up at [VirusTotal](https://www.virustotal.com/) to get your API key.

3. **Configure VirusTotal Integration:**
   - Edit `/var/ossec/etc/ossec.conf` to include the following:
     ```xml
     <virustotal>
       <enabled>yes</enabled>
       <api_key>[your_api_key]</api_key>
     </virustotal>
     ```

4. **Restart Wazuh Manager:**
   ```bash
   systemctl restart wazuh-manager
   ```

5. **Test:**
   - Add a suspicious file to the monitored directory:
     ```bash
     echo "malware" > /root/suspicious-file.txt
     ```
   - Check the Wazuh dashboard for VirusTotal analysis results.

---

## Accessing the Dashboard

To visualize and manage the data collected by Wazuh:

1. **Access the Default URL:**
   - Open a browser and navigate to: `http://[your_server_IP]:5601`

2. **Login Credentials:**
   - Use the default credentials: `admin` / `secret_password`. If you have changed the credentials, use the updated ones.

3. **Explore Dashboards:**
   - Navigate through the Wazuh dashboard to explore different sections like:
     - Security Events
     - Vulnerabilities
     - File Integrity Monitoring
     - Network Intrusion Detection

4. **Customize Dashboards:**
   - Create custom visualizations and dashboards to tailor Wazuh to your specific needs.

---

## Resources

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Wazuh GitHub Repository](https://github.com/wazuh/wazuh)
- [VirusTotal](https://www.virustotal.com/)
- [Emerging Threats Rules](https://rules.emergingthreats.net/open/)
- [Suricata Documentation](https://suricata.io/docs/)

---
