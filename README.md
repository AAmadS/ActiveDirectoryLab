<h1>Active Directory Home Lab</h1>

<h2>Description</h2>
This project simulates a real-world SOC workflow for detecting and responding to unauthorized logins in an Active Directory environment. Using Splunk, Slack, and Shuffle automation, the system alerts a SOC analyst of suspicious logins, enabling them to decide whether to disable the compromised account. If approved, the account is automatically disabled, and a confirmation is sent via Slack.
<br />

<h2>Languages and Utilities Used</h2>

- <b>Draw.io</b>
- <b>Splunk</b>
- <b>Shuffle</b>
- <b>Vultr</b>

<h2>Environments Used </h2>

- <b>Windows 10</b> 2022 x64
- <b>Ubuntu</b> 22.02 x64

<h2>Project walk-through:</h2>

<p align="center">
<b>Creating a Diagram with Draw.io:</b> <br/>
<img src="https://imgur.com/SFquO3T.png" height="80%" width="80%" alt="Creating a Diagram with Draw.io"/>
<br />
</p>

<p>
This diagram illustrates a security workflow for detecting and responding to unauthorized logins in an Active Directory environment. The setup includes three virtual machines hosted on VULTR—a Domain Controller, a Test Machine, and a Splunk server—along with an attacker machine simulating an external threat actor.

The attacker, using valid credentials, successfully authenticates to the Test Machine. This event generates telemetry, which is forwarded to the Splunk server. Upon detecting the login, Splunk triggers an alert that is sent to Slack and also initiates a Shuffle automation playbook.

The Successful Unauthorized Login Playbook sends an email to the SOC Analyst, asking whether the suspicious user account should be disabled.

- If Yes, Shuffle proceeds to disable the domain account in Active Directory and sends a confirmation notification to Slack.
- If No, no further action is taken.
</p>

<br />
<p align="center">
<b>Setting up virtual machines on Vultr and testing communications:</b><br/>
<img src="https://imgur.com/AEnUhC5.png" height="80%" width="80%" alt="Setting up virtual machines on Vultr and testing communications"/>
<br />
</p>

<p>
 
The virtual machines consist of:
- Two Windows Servers
  - [AAmod-ADDC01] Domain Controller Specs: 
    - Server Type: Shared CPU
    - Cores: 2vCPUs
    - Memory: 4GB
    - Storage: 80GB
    - Windows Standard Version: 2022 x64
  - [Cloud Instance] Test Machine Specs (forgot to change name):
    - Server Type: Shared CPU
    - Cores: 1vCPUs
    - Memory: 2GB
    - Storage: 55GB
    - Windows Standard Version: 2022 x64
- [AAmod-Splunk] One Ubuntu Server:
  - Splunk Machine Specs:
    - Server Type: Shared CPU
    - Cores: 4vCPUs
    - Memory: 8GB
    - Storage: 160GB
    - Ubuntu Version: 22.02 x64

<p align="center">
<br/>
<b>Debugging "Destination Host Unreachable" after setting up VPC on Splunk Machine:</b><br/>
<img src="https://imgur.com/gjhTuRN.png" height="80%" width="80%" alt="Debugging 'Destination Host Unreachable' after setting up VPC on Splunk Machine"/>
<br/>
 
</p>

After setting up my virtual machines I then went over to the Network -> Firewall section to create a group Firewall. By default a rule if automatically created for you which is to accept SSH connections from anywhere. This is simply not secure and I changed it to only accept SSH connections from my IP address along with MS RDP (Microsoft Remote Desktop Protocol) inbound. Next I setup a VPC so that all my virtual machines on the same VPC could communicate with each other internally.

There are two ways you can access the virtual machines, one is through Windows remote desktop and the other is through the terminal button on the vultr website. For the sake of exploring and gainning expereince I went and preformed the "ipconfig" command on both the Cloud Instance (Test Machine) and AAmod-ADDC01. Remote desktop was definitely easier because of performance and not having to use alternative shortcuts. In the website console vultr gives you additional buttons on the side of the screen allowing you to preform key combinations like "ctrl+alt+delete", but as convieant as this option is, its simply very slow. To configure the AAmod-Splunk I decided to ssh into the virtual machine, after logging in I realized that I hadn't setup my firewall and VPC like the other virtual machines yet. After doing so the virtual machine reset and once I relogged I tried pinging another virutal machine but this where I ran into my first issue. Since all my virtual machines are on the same VPC I wasnt expecting to see a "Destination Host Unreachable" error when I tried pinging another machine from my Splunk terminal. 

</p>

<p align="center">
<br />
<b>Splunk Machine receiving pings:</b> <br/>
<img src="https://imgur.com/Lh1C1RI.png" height="80%" width="80%" alt="Splunk Machine receiving pings"/>
<br />
</p>
 
<p>

To trouble shoot this issue I logged into my Cloud Instance (Test Machine) and preformed "ipconfig" in the terminal as seen in the screenshot above. If you look at the "Authoconfiguration IPv4 Address" under the "Ethernet Instance 0 2" section you can see that the IP address is 169.254.98.164 when it should be 10.1.96.3 which came from the VPC I set setup earlier. In order to fix this I right clicked the internet icon bottom right -> "Open Network & Internet Settings" -> "Change Adapter Options" -> right click "Ethernet Instance 0 2" -> "Properties" -> double click "Internet Protocol version 4 (TCP/IPv4)" -> click "Use the following IP address" and placed the VPC IP address and subnet mask given from vultr earlier. After, I went back to my ssh on my Splunk machine did "ping 10.1.96.3" and started receving pings. After, I went and did the same to my AAmod-ADDC01 machine. 

</p>

<p align="center">
<br />
<b>Installing Active Directory on AAmod-ADDC01:</b> <br/>
<img src="https://imgur.com/wXBGvcy.png" height="80%" width="80%" alt="Installing Active Directory on AAmod-ADDC01"/>
<br />
</p>

<p>
With the three virtual machines now setup, its time to install & configure active directory on AAmod-ADDC01, promote it to a domain controller and configure the target machine to join the new domain. Since I didnt change many settings except for the checking the box "Active Directory Domain Services" as seen in the screen shot I decided not to include much about that step. The same goes for promoting the server, after clicking next a couple times, and installing Active Directory. A notification appears in the top right of the Server Manager window that has an option to "Promte this server to a domain controller". In this new wizard the only thing I changed other than adding a password was in the first step "Deployment Configuration". In this step I made sure to check the "Add a new forest" option and set my root domain name to "AAmod.local".

</p>

<p align="center">
<br />
<b>Adding First User:</b> <br/>
<img src="https://imgur.com/FRa8dGk.png" height="80%" width="80%" alt="Adding First User"/>
<br />
</p>

<p>
After the domain controller setup I quickly made a user for testing later.
</p>


<br />


<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
