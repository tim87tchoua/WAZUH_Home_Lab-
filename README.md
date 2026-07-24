# WAZUH_Home_Lab
WAZUH Home Lab -SIEM and File Integrity Monitoring
1. Overview

Wazuh is a free, open-source security platform that offers:
	Log analysis
	 File integrity monitoring
	 Intrusion detection
	Vulnerability detection
	 Real-time alerting
This guide demonstrates how to build a basic Wazuh setup for learning and experimentation.

2. Lab Architecture
Component	Host	Role

Wazuh Manager	Ubuntu (VirtualBox)
	Collects, analyzes, and stores data from agents

Wazuh Agent	Windows (host machine)	Sends logs and system events to the Wazuh manager


Network Configuration:
Use Bridged Adapter in VirtualBox to place the Ubuntu server on the same network as the host. This allows access between host and guest.
3. Prerequisites
● VirtualBox installed
● Ubuntu Server 24.04+ installed in VirtualBox (bridged networking) (lsb_release -a, cat /etc/os-release, or hostnamectl.)
● Internet access on Ubuntu VM
● Administrative access on the Windows host
● Optional: basic knowledge of Linux and system administration


4. Installing the Wazuh Manager (Ubuntu)
Run the following steps on your Ubuntu VirtualBox server.
4.1 Add Wazuh GPG Key
Via a terminal window
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh-archive-keyring.gpg
This adds the GPG key to verify Wazuh packages.
That command downloads and installs Wazuh's official GPG signing key onto our Linux system so our package manager (like apt) can verify that any downloaded Wazuh software is authentic and hasn't been tampered with.
Here is a quick breakdown of what each part of that command actually does:
Step-by-Step Breakdown
Bash: curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH
•	curl -s: Quietly (-s for silent) fetches the GPG key file directly from Wazuh's official repository server.
Bash: | sudo gpg --dearmor -o /usr/share/keyrings/wazuh-archive-keyring.gpg
•	| Takes the output from curl (the raw GPG key) and passes it directly into the next command as input.
•	sudo: Runs the command with administrative privileges, which is required because /usr/share/keyrings/ is a protected system directory.
•	gpg --dearmor: Converts the incoming key from human-readable text (ASCII-armored format, starting with -----BEGIN PGP PUBLIC KEY BLOCK-----) into a binary format that Debian and Ubuntu package managers require.
•	-o /usr/share/keyrings/wazuh-archive-keyring.gpg: Saves that converted binary key file to /usr/share/keyrings/, which is the modern standard location for third-party APT repository keys.
Why is this necessary?
When you add the Wazuh repository to your package manager later, apt will use this stored key to double-check the cryptographic signature of every package you install or update.
If a package's signature doesn't match this key, your system will block the installation—protecting you from corrupted downloads or potential security attacks.


4.2 Download and Execute Wazuh Installation Script
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh && sudo bash
./wazuh-install.sh -a -i
● -a: Installs all components (manager, indexer.)
● -i: Runs in interactive mode
This command sequence downloads and immediately runs the official automated installer to set up a complete Wazuh system (version 4.12) on a single machine.
It is used for quickstart or single-node deployments to spin up an entire security monitoring stack. 
Medium
Breakdown of the Command
1. Download Stage
Bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
•	curl: Downloads the script file.
•	-s (silent): Suppresses the progress bar and technical status messages.
•	-O (capital O): Saves the file locally using the remote filename (wazuh-install.sh) rather than printing it to your screen.
2. Execution Stage
Bash
&& sudo bash ./wazuh-install.sh -a -i
•	&&: Tells your terminal to run the next command only if the curl download succeeds.
•	sudo bash ./wazuh-install.sh: Runs the installation script as the root user.
•	-a (All-in-One): Instructs the installer to deploy all core components—Wazuh Manager, Indexer, and Dashboard—together on this single host.
•	-i (Indexer): explicitly targets or forces configuration specific to the Wazuh Indexer component alongside the installation process.
What happens when you run this?
1.	System Check: Checks if your host meets system requirements (RAM, CPU, supported Linux OS).
2.	Component Deployment: Downloads and installs:
o	Wazuh Indexer: Stores and indexes log data.
o	Wazuh Manager: Analyzes incoming events and triggers alerts.
o	Wazuh Dashboard: Web interface to view threat alerts and metrics. 
Reddit
3.	Generates Credentials: Automatically generates TLS/SSL certificates and outputs a randomly generated admin password on your screen.
⚠️ Important: Make sure to write down the admin password and web dashboard URL printed at the end of the installation process! You will need them to log into the web interface. 
Reddit


The script installs all required services and configures them automatically.

5. Accessing the Wazuh Dashboard
After installation:
Check your Ubuntu VM’s IP address: ifconfig
On your Ubuntu server, open a browser and go to:
https://<ubuntu-vm-ip>
1. Accept any browser security warning due to the self-signed certificate.
2. Log in using the credentials displayed at the end of the installation script.

6. Installing the Wazuh Agent (Windows Host)

1. Download the latest Wazuh agent MSI installer from the official documentation:
https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html
Wazuh Agent for Windows
Key Compatibility Rules
A.	Version Alignment: Always ensure your Wazuh Manager version is equal to or newer than your Wazuh Agent version. For example, a 4.14 Manager can manage a 4.12 Agent, but a 4.10 Manager should not manage a 4.14 Agent. 
B.	Container Deployments: If running the manager via Docker/Kubernetes, the host Ubuntu version matters less as long as it has Docker Engine installed.
2. Install the MSI package on your Windows system using the default settings.
7. Registering the Agent with the Manager

7.1 Generate Agent Key on Ubuntu Manager
Run the agent management utility:
sudo /var/ossec/bin/manage_agents
 
● Select A to add an agent.
● Assign a name (e.g., WINDOWS.AGENT).
● Leave IP address blank unless static assignment is needed.
NB: For our LAB add your windows IP address
● After creation, select E to extract the key. 
● Copy the key output. And Quit (Q)
7.2 Apply Key in the Windows Agent
1. Open Wazuh Agent Manager GUI from the Start Menu.
2. Paste the copied key into the appropriate field.
3. Save and apply the key.
4. Add the manager's IP address (IP address of your Ubuntu manager).
5. Restart the agent service.
 
You can then go to the WAZUH dashboard and see the agent onboarded.
8. File Integrity Monitoring on Windows

Wazuh supports real-time monitoring of file and folder changes using Syscheck.
8.1 Edit Agent Configuration
Open the following configuration file:
C:\Program Files (x86)\ossec-agent\ossec.conf (In linux: /var/ossec/etc/ossec.conf)
Add the following entry inside the Directory block:
<directories realtime="yes">C:\Users\abc\Test</directories>
This monitors the specified folder in real-time.
8.2 Restart the Agent
After saving the changes, restart the Wazuh agent service to apply the configuration.

9. Verifying Setup

1. Open the Wazuh Dashboard in your browser.
2. Navigate to Agents → ensure the Windows agent is listed and status is Active.
3. Go to the Integrity Monitoring section.
4. Perform actions (create/modify/delete files) in the monitored folder.
5. Confirm that alerts appear in the dashboard.

