<h1>Active Directory Home Lab</h1>

<h2>Description</h2>
This project recreates a small-scale Security Operations Center (SOC) workflow in a virtual Active Directory environment. It integrates Splunk, Shuffle SOAR, and Slack to detect unauthorized logins and automate incident response. When suspicious activity is detected, Splunk triggers a Shuffle playbook that alerts a SOC analyst via Slack and upon approval automatically disables the compromised account in Active Directory.
<br />

<h2>Tools & Environment</h2>

- <b>Draw.io – network and workflow diagramming</b>
- <b>Slack – SOC communication and alerting</b>
- <b>Splunk – log ingestion and alert generation</b>
- <b>Shuffle – automation and Active Directory integration</b>
- <b>Vultr – virtual machine hosting</b>
- <b>Windows Server 2022 & Windows 10 – Active Directory and test clients</b>
- <b>Ubuntu 22.04 LTS – Splunk server</b>

<h2>Project walk-through:</h2>

<p align="center">
<b>Creating a Diagram with Draw.io:</b> <br/>
<img src="https://imgur.com/xcrLGXb.png" height="80%" width="80%" alt="Creating a Diagram with Draw.io"/>
<br />
</p>

<p>
This diagram outlines the overall security workflow for detecting and responding to unauthorized logins within an Active Directory environment. The lab consists of three <b>Vultr-hosted virtual machines</b>—a Domain Controller, a Test Machine, and a Splunk Server—plus an external Attacker Machine used to simulate intrusion attempts.<br/><br/>

When the attacker logs in using valid but unauthorized credentials, the event is captured and forwarded to <b>Splunk</b> for analysis. Once Splunk detects the login, it triggers a <b>Shuffle</b> playbook that sends an alert to <b>Slack</b> for analyst review.<br/><br/>

The playbook then prompts the analyst via email to confirm whether the compromised account should be disabled:
<ul>
<li><b>Yes</b> — Shuffle disables the user in Active Directory and sends a confirmation message to Slack.</li>
<li><b>No</b> — The workflow ends without taking action.</li>
</ul>
</p>

<p align="center">
<b>Setting up Virtual Machines on Vultr and Testing Communications:</b><br/>
<img src="https://imgur.com/AEnUhC5.png" height="80%" width="80%" alt="Setting up virtual machines on Vultr and testing communications"/>
<br/>
</p>

<p>
The lab environment consists of three virtual machines hosted on <b>Vultr Cloud</b>:
</p>

<ul>
  <li><b>[AAmod-ADDC01] – Domain Controller</b>  
    <ul>
      <li>Server Type: Shared CPU</li>
      <li>Cores: 2 vCPUs</li>
      <li>Memory: 4 GB</li>
      <li>Storage: 80 GB</li>
      <li>OS: Windows Server 2022 x64</li>
    </ul>
  </li>

  <li><b>[Cloud Instance] – Test Machine</b> (joined to the domain)  
    <ul>
      <li>Server Type: Shared CPU</li>
      <li>Cores: 1 vCPU</li>
      <li>Memory: 2 GB</li>
      <li>Storage: 55 GB</li>
      <li>OS: Windows Server 2022 x64</li>
    </ul>
  </li>

  <li><b>[AAmod-Splunk] – Splunk Server</b>  
    <ul>
      <li>Server Type: Shared CPU</li>
      <li>Cores: 4 vCPUs</li>
      <li>Memory: 8 GB</li>
      <li>Storage: 160 GB</li>
      <li>OS: Ubuntu 22.04 LTS x64</li>
    </ul>
  </li>
</ul>

<p align="center">
<b>Debugging "Destination Host Unreachable" After Setting Up VPC on Splunk Machine:</b><br/>
<img src="https://imgur.com/gjhTuRN.png" height="80%" width="80%" alt="Debugging 'Destination Host Unreachable' after setting up VPC on Splunk Machine"/>
<br/>
</p>

<p>
After deploying the virtual machines, I configured network security through the <b>Vultr Firewall</b> and <b>Virtual Private Cloud (VPC)</b>. By default, Vultr allows SSH connections from any source, so I restricted access to only my IP address and added inbound rules for <b>RDP (3389)</b> to securely access the Windows hosts.<br/><br/>

A <b>VPC</b> was created to allow private communication between the Domain Controller, Test Machine, and Splunk server. The VMs could be accessed either through <b>Remote Desktop</b> or the <b>Vultr web console</b>—though RDP offered better performance for system configuration.<br/><br/>

During initial testing, I encountered a <i>"Destination Host Unreachable"</i> error when pinging from the Splunk server to other machines, despite all systems being on the same VPC. This issue was later traced to a network misconfiguration, which I corrected by assigning proper IP addresses within the VPC subnet.
</p>

<p align="center">
<b>Splunk Machine Receiving Pings:</b><br/>
<img src="https://imgur.com/Lh1C1RI.png" height="80%" width="80%" alt="Splunk Machine receiving pings"/>
<br/>
</p>

<p>
To resolve the connectivity issue, I inspected the network configuration on the <b>Cloud Instance (Test Machine)</b> using <code>ipconfig</code>. The system had self-assigned an <b>APIPA address (169.254.x.x)</b> instead of the expected <b>VPC IP (10.1.96.3)</b>.<br/><br/>

The fix involved manually assigning the correct IP and subnet mask through the adapter settings (<i>Ethernet Instance 0 2 → Properties → IPv4 → Use the following IP address</i>). After applying the proper configuration, the <b>Splunk server</b> successfully received ICMP replies from the Test Machine, confirming internal VPC communication. The same correction was applied to the Domain Controller.
</p>


<p align="center">
<b>Installing Active Directory on AAmod-ADDC01:</b><br/>
<img src="https://imgur.com/wXBGvcy.png" height="80%" width="80%" alt="Installing Active Directory on AAmod-ADDC01"/>
<br/>
</p>

<p>
With all virtual machines configured, I installed and set up <b>Active Directory Domain Services (AD DS)</b> on <b>AAmod-ADDC01</b>. The server was then promoted to a <b>Domain Controller</b> for a new forest named <b>AAmod.local</b>.<br/><br/>

The installation followed the standard Windows Server process, requiring minimal configuration changes beyond selecting <i>“Add a new forest”</i> and setting a secure DSRM password. After the promotion completed, the domain was ready for client machines to join.
</p>


<p align="center">
<b>Adding First User:</b><br/>
<img src="https://imgur.com/FRa8dGk.png" height="80%" width="80%" alt="Adding First User"/>
<br/>
</p>

<p>
After completing the Domain Controller setup, I created an initial test user account in <b>Active Directory Users and Computers (ADUC)</b>. This account was used to verify authentication and connectivity from the client machine.
</p>

<p align="center">
<b>Joining Cloud Instance (Test Machine) to the New Domain:</b><br/>
<img src="https://imgur.com/fbhuSms.png" height="80%" width="80%" alt="Joining Cloud Instance (Test Machine) to new domain"/>
<br/>
</p>

<p>
The <b>Cloud Instance (Test Machine)</b> was then joined to the <b>AAmod.local</b> domain. This confirmed proper DNS resolution and domain connectivity between the Windows client and the Domain Controller. Once joined, login with the new domain credentials succeeded, validating the AD environment’s functionality.
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
With these new rules added, I was finally able to access the Splunk Enterprise website. On the website I first changed my timezone to (GMT-04:00) so that any alerts would be in a timezone relevent to myself and I installed "Splunk Add-on for Microsoft Windows" through the apps section. Next I went into "Settings" -> "Indexes" -> "New Index" and created a new index called "aamod-ad". Additionally, I went into "Settings" -> "Fowarding and receiving" -> "Configure receiving" -> "New Receving Port" to listen in on port 9997 (Splunk default).
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
These next steps are to done on both AAmod-Splunk(Ubuntu) and AAmod-ADDC01(Domain Controller) to allow telemery to Splunk, and the bit added to the "inputs.conf" file is the index that I created earlier.
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

<p align="center">
<br />
<b>Telemetry Confirmation on Splunk Enterprise</b> <br/>
<img src="https://imgur.com/CC4mHKI.png" height="80%" width="80%" alt="Telemetry Confirmation on Splunk Enterprise"/>
<br />
</p>

<p>
 Since telemetry is working I decieded to loosen up my firewall by changing the TCP (MS RDP) rule to allow a source from anywhere. Since this is just a project for experimenting this wasn't a big deal. I also setup my first alert on Splunk as shown below.
</p>
<ul>
 <li>index="aamod-ad" EventCode=4624 (Logon_Type=7 OR Logon_Type=10) Source_Network_Address=* Source_Network_Address!="-" Source_Network_Address!=40.* |stats count by _time,ComputerName,Source_Network_Address,user,Logon_Type</li>
</ul>

<p align="center">
<br />
<b>First Shuffle alert from Splunk after connecting webhook</b> <br/>
<img src="https://imgur.com/zb9B9sO.png" height="80%" width="80%" alt="First Shuffle alert from Splunk"/>
<br />
</p>

<p>
 Now is when I intergrate Slack and Shuffle for automation and response. As well as build a responsive playbook to disable any domain user if an unauthorized login was detected. For context the IP shown on the right on this alert was from logging in to the Cloud Instance (Test Machine) with a VPN active. I used PrivadoVPN since they have a free plan and I only needed a VPN active to generate some alerts for Splunk.
</p>

<p>
Connecting Webhook to Splunk:
</p>
<ol>
<li>In a fresh Shuffle workflow, drag in a webhook trigger and copy the Webhook URI</li>
<li>Next go back to Splunk Enterprise, edit the alert already made and add a new trigger for Webhook and paste in the URI taken from shuffle</li>
<li>Now go back into shuffle and press start on the webhook node, this will allow shuffle is able to recieve any alerts from splunk </li>
</ol>

<p>
This is the manual way of connecting the Shuffle bot to Slack because in my case the "One-click Authentication" was giving me errors.<br>

Connecting Slack to Shuffle:
</p>
<ol>
<li>First log into your Slack account and make sure you have a workspace.</li>
<li>Open https://api.slack.com/apps -> "Create New App" -> <i>"From Scratch"</i>.</li>
<li>After naming the app go to "OAuth & Permissions" -> "Redirect URLs -> "Add New Redirect URL" -> paste https://shuffler.io/set_authentication -> "Add" -> "Save URLs".</li>
<li>Now add the bare minimum "Bot Tokens Scopes", in my case I only used "chat:write" , "app_mentions:read" , "users:read". </li>
<li>Now at the top of "OAuth & Permissions -> "Install to Workspace -> "Allow". </li>
<li>Go back to "Basic Information" -> "App Credentials" and bring the "Client ID" and "Client Secret" over to Shuffle.</li>
<li>After dragging in the Slack node and clicking "Authenticate" you will see slots to put your "Client ID and "Client Secret".</li>
<li>Now add the <b>EXACT</b> same scopes you added on the Slack bot to the Slack node.</li>
<li>Lastly refresh your runs by clicking the tab on the right called "Explore Runs" so that data can populate in the Slack node.</li>
</ol>

<p>
Setting Up the Slack node:
</p>

<ol>
<li>First grab the channel ID for the channel the bot is in. This can be found in the URL of the channel in the browser version of Slack, it looks something like C07N2L4D1O0</li>
<li>Enter channel ID into Node under "Channel"</li>
<li>In the "Text" section add this "Alert: $exec.search_name \n Time:$exec.result._time \n User: $exec.result.user \n  Source IP: $exec.result.Source_Network_Address". The the portions that have "$exec" can be auto             generated by clicking the plus icon asuming your Slack node got populated after refreshing.</li>
</ol>

<p align="center">
<br />
<b>Shuffle Test p1: Alert taken from Splunk</b> <br/>
<img src="https://imgur.com/9Tcdx3U.png" height="80%" width="80%" alt="Shuffle Test p1: Alert taken from Splunk"/>
<br />
</p>

<p align="center">
<br />
<b>Shuffle Test p2: Slack Node getting Alert</b> <br/>
<img src="https://imgur.com/eobmafC.png" height="80%" width="80%" alt="Shuffle Test p2: Slack Node getting Alert"/>
<br />
</p>

<p align="center">
<br />
<b>Shuffle Test p3: Alert posted to Slack</b> <br/>
<img src="https://imgur.com/HlCHg9O.png" height="80%" width="80%" alt="Shuffle Test p3: Alert posted to Slack"/>
<br />
</p>
<br />

<p>
The three screenshots are proof of the bot working. I first renabled my Splunk alert, turned on a VPN and logged into the JJohnson account on my Cloud Instance (Test Machine) to generate data. After the alert was triggered on Splunk the webhook brought that data over to Shuffle which then got tranfered to the Slack node. From the Slack node, it then got pushed to the Shuffle bot on Slack and posted the alert in chat.
</p>

<br/>

<p align="center">
<br />
<b>Shuffle Email Notification Confirmation</b> <br/>
<img src="https://imgur.com/FoCO7aW.png" height="80%" width="80%" alt="Shuffle Email Notification Confirmation"/>
<br />
</p>

<p>
 Now to setup the email notification in shuffle asking if I want to disable the user of interest.
</p>

<ol>
 <li>First drag in a "User Input".</li>
 <li>Type "Would like to disable the user? Start parameters: $exec".</li>
 <li>Check the "Email" check box</li>
</ol>

<p>
 To test, I went and made a temp mail and did a new run of the workflow.
</p>

<p>
 Now to connect this user action so that if I want to disable the user I can just click the link for disable and get a notification on Slack for confirmation. To do this I user Shuffle's Active Directory node.
</p>

<p align="center">
<br />
<b>Shuffle Final Workflow Visual</b> <br/>
<img src="https://imgur.com/ahSMxQa.png" height="80%" width="80%" alt="Shuffle Final Workflow Visual"/>
<br />
</p>

<p align="center">
<br />
<b>Slack Notification of account disabled</b> <br/>
<img src="https://imgur.com/soxKnDN.png" height="80%" width="80%" alt="Slack Notification of account disabled"/>
<br />
</p>

<ol>
 <li>First drag in the Active Directory node.</li>
 <li>Connect the "User Action" node to the "Active Directory" node</li>
 <li>Fill in the requested date. In my case everything except for "base_dn" was found on the Vultr website where I got my VMs.</li>
 <li>LDAP default port will be 389.</li>
 <li>To get the "base_dn" remote desktop into the domain controller.</li>
 <li>Open PowerShell and type "Get-ADDomain". At the very bottom you will see the "base_dn".</li>
 <li>Back in Shuffle in the "Active Directory" node, "Find Actions" -> "Disable User" and "Samacountname" -> type "$exec.result.user"</li>
 <li>Also allow port 389 in firewall rules.</li>
 <li>Back in Shuffle drag in another "Active Directory" Node.</li>
 <li>Under "Find Actions" -> select "User attributes" -> in "Search Base" enter the base_dn.</li>
 <li>Drag in a "Repeat back to me" node</li>
 <li>Under "Find Actions" -> select "Repead back to me".</li>
 <li>Under "Call" type "$get-user-attributes.attributes.userAccountControl".</li>
 <li>Drag a branch to a new Slack node</li>
 <li>Click the branch to add a new condition.</li>
 <li>Under "Source" type "$get-user-attributes.attributes.userAccountControl" -> Switch the middle to "contains" -> under "destination" type" ""ACCOUNTDISABLED""</li>
 <li>In the new Slack node enter the channel ID and in "Text" type "Account: $exec.result.user has been disabled."</li>
</ol>



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
