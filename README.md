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
  - [Cloud Instance] Test Machine Specs (forgot to rename):
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
After the domain controller setup I made a user for testing and authenticating my Cloud Instance (Test Machine). Now to join the Cloud Instance (Test Machine) to the new domain.
</p>

<p align="center">
<br />
<b>Joining Cloud Instance (Test Machine) to new domain</b> <br/>
<img src="https://imgur.com/fbhuSms.png" height="80%" width="80%" alt="Joining Cloud Instance (Test Machine) to new domain"/>
<br />
</p>

<p>
Steps for joining test machine to new domain:
<ol> 
<li>Type "This PC" in the task bar search</li>
<li>Right click, select "properties"</li>
<li>Left click  "Rename this PC (advanced)" under "Related Settings"</li>
<li>In the "Computer Name" tab click "change"</li>
<li>In the "Member of" section left click "Domain" and add the domain name, in my case it was "AAmod"</li>
<li>Once you click "OK" you will be promoted to put in your administative credentials and you should see a pop up saying "Welcome to [name] domain</li>
</ol>
</p>

<p align="center">
<br />
<b>Confirmation of Joining Domain</b> <br/>
<img src="https://imgur.com/lsTfLg0.png" height="80%" width="80%" alt="Confirmation of Joining Domain"/>
<br />
</p>


</p>
Incase of DNS issues:
<ol>
<li>Right click the network icon in the bottom right of the task bar and left click "Open Network and Internet Settings.</li>
<li>Under "Advanced network settings" left click "Change adapter options"</li>
<li>Double left click "Ethernet Instance 0 2"</li>
<li>Left click "Properties"</li>
<li>Double left click "Internet Protocol Version 4 (TCP/IPv4)"</li>
<li>Under "Use the following DNS server addresses:" enter the VPC of the domain controller into the "Perferred DNS server": field</li>
</ol>
</p>

<p align="center">
<br />
<b>Splunk Enterprise Trial for Ubuntu</b> <br/>
<img src="https://imgur.com/G2PjT5w.png" height="80%" width="80%" alt="Splunk Enterprise Trial for Ubuntu"/>
<br />
</p>

<p>
Here is where I install and configure Splunk on the Ubuntu server, configure Windows end points to send telemetry over to the splunk server and create a Splunk alert to detect successful Splunk authentications. Since is personal project I'm just using a trial version of Splunk Enterprise for my Ubuntu VM. On the Splunk Enterprise webpage the "Copy wget link" gives you a command you can directly put into your linux terminal to download Splunk. In my case I used the .deb file format.
</p>

<p align="center">
<br />
<b>Locating Splunk binary</b> <br/>
<img src="https://imgur.com/ao1kTuA.png" height="80%" width="80%" alt="Splunk Enterprise Trial for Ubuntu"/>
<br />
</p>

<p>
Here I'm simply locating the Splunk binary and runnning the command "./splunk start". After performing this command I was met with the licensing agreement and then I set up an administrator account. Before I'm able to access the Splunk Enterprise website on port 8000, I first needed to do a few things.
</p>
<ul>
<li>Add a firewall rule on Vultr to accept TCP from port 8000 from my IP address</li>
<li>In my Ubuntu VM type "ufw allow 8000"</li>
</ul>
<p>
With these new rules added, I was finally able to access the Splunk Enterprise website. On the website I first changed my timezone to GMT so that any alerts would be in a timezone relevent to myself and I installed "Splunk Add-on for Microsoft Windows" through the apps section. Next I went into "Settings" -> "Indexes" -> "New Index" and created a new index called "aamod-ad". Additionally, I went into "Settings" -> "Fowarding and receiving" -> "Configure receiving" -> "New Receving Port" to listen in on port 9997 (Splunk default).
</p>

<p align="center">
<br />
<b>Splunk Universal Forwarder Setup</b> <br/>
<img src="https://imgur.com/L5q7nt5.png" height="80%" width="80%" alt="Splunk Universal Forwarder Setup"/>
<br />
</p>

<p>
Now to setup Splunk Universal Forwarder on my Cloud Instance (Test Machine). First I went over to the Splunk website and downloaded the 64-bit installer and copied it over to my Cloud Instance (Test Machine). In the installer, at the "Receiving Indexer" setup I added my AAmod-Splunk VPC on port 9997 (default). 
</p>

<p align="center">
<br />
<b>Adding inputs.conf to SplunkUniversalForwarder local</b> <br/>
<img src="https://imgur.com/AZ3o0eB.png" height="80%" width="80%" alt="Adding inputs.conf to SplunkUniversalForwarder local"/>
<br />
</p>

<p>
 These next steps allow telemery to Splunk, and the bit added to the "inputs.conf" file is the index that I created earlier.
</p>

</p>
Adding "inputs.conf" to SplunkUniversalForwarder local folder:
<ol>
<li>Left click "File Explorer"</li>
<li>Left click "This PC"</li>
<li>Double left click "System Drive (C:)"</li>
<li>Double left click "Program files"</li>
<li>Double left click "SplunkUniversalForwarder"</li>
<li>Double left click "etc"</li>
<li>Double left click "system"</li>
<li>Double left click "default"</li>
<li>Right click and copy "inputs.conf"</li>
<li>left click "system" at the top to go back</li>
<li>Double click "local"</li>
<li>Right click and paste the "inputs.conf" file</li>
<li>In the search bar on the task bar search "notepad" and open</li>
<li>On the top left, left click "File" -> "New" and navigate to the "inputs.conf" file and double left click it to open</li>
<li>Scroll to the very bottom and add <br />
"[WinEventLog://Security] <br />
index = aamod-ad <br />
disabled = false" </li>
</ol>
</p>

</p>
Restarting "SplunkForwarder":
<ol>
<li>In the task bar search type "services" and left click</li>
<li>Scroll down until you find "SplunkForwarder" and double left click</li>
<li>left click the "Log On" tab</li>
<li>left click the "Local System account" check box and left click "Apply"</li>
<li>In the "Services" window, right click "SplunkForwarder" and left click "Restart"</li>
<li>If a warning pops up left click "OK"</li>
<li>Right click "SplunkForwarder" and left click "Start"</li>
</ol>
</p>

</p>
Configure Port 9997:
<ol>
<li>ssh into the AAmod-Splunk Ubuntu VM</li>
<li>type "ufw allow 9997" press Enter</li>
</ol>
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
