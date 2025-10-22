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
<b>Confirmation of Joining Domain:</b><br/>
<img src="https://imgur.com/lsTfLg0.png" height="80%" width="80%" alt="Confirmation of Joining Domain"/>
<br/>
</p>

<p>
After joining the <b>Cloud Instance (Test Machine)</b> to the <b>AAmod.local</b> domain, I verified the connection through <b>Active Directory Users and Computers (ADUC)</b> and confirmed that domain logins were successful. This confirmed full AD functionality and communication across the network.<br/><br/>

If DNS issues occurred during the join process, I manually set the Domain Controller’s VPC IP as the <b>Preferred DNS Server</b> within the client’s network adapter settings to ensure proper domain resolution.
</p>

<p align="center">
<b>Installing Splunk Enterprise on Ubuntu:</b><br/>
<img src="https://imgur.com/G2PjT5w.png" height="80%" width="80%" alt="Splunk Enterprise Trial for Ubuntu"/>
<br/>
</p>

<p>
Next, I deployed <b>Splunk Enterprise</b> on the <b>Ubuntu Server</b> to collect and analyze Windows event logs from the Domain Controller and Test Machine. The installation was completed using the Linux <code>.deb</code> package via the <code>wget</code> command provided by Splunk’s website.<br/><br/>

For this personal lab, I used the <b>Splunk Enterprise Trial</b> to configure data forwarding, event indexing, and alert creation for login activity within the Active Directory environment.
</p>

<p align="center">
<b>Locating Splunk Binary:</b><br/>
<img src="https://imgur.com/ao1kTuA.png" height="80%" width="80%" alt="Locating Splunk binary"/>
<br/>
</p>

<p>
After installation, I located the <b>Splunk binary</b> on the Ubuntu server and initialized Splunk using <code>./splunk start</code>. Once the license agreement was accepted, I created an administrator account and enabled the web interface on port <b>8000</b>.<br/><br/>

To ensure connectivity, I configured firewall rules to allow TCP traffic on port <b>8000</b> from my host IP and added a <code>ufw allow 8000</code> rule inside the VM. This allowed access to the <b>Splunk Enterprise Web UI</b> for further setup.
</p>

<p align="center">
<b>Configuring Splunk Universal Forwarder:</b><br/>
<img src="https://imgur.com/L5q7nt5.png" height="80%" width="80%" alt="Splunk Universal Forwarder Setup"/>
<br/>
</p>

<p>
To collect Windows event logs, I installed the <b>Splunk Universal Forwarder</b> on the <b>Cloud Instance (Test Machine)</b> and linked it to the Splunk indexer (<b>AAmod-Splunk</b>) over port <b>9997</b>.<br/><br/>

During installation, I specified the receiving indexer’s address and confirmed connectivity. The forwarder was configured to send <b>Security Event Logs</b> using the <b>inputs.conf</b> file and the previously created <b>aamod-ad</b> index. This allowed Windows telemetry to stream directly into Splunk Enterprise for monitoring and alerting.
</p>

<p align="center">
<b>Adding inputs.conf to Splunk Universal Forwarder:</b><br/>
<img src="https://imgur.com/AZ3o0eB.png" height="80%" width="80%" alt="Adding inputs.conf to SplunkUniversalForwarder local"/>
<br/>
</p>

<p>
To enable telemetry forwarding, I configured the <b>inputs.conf</b> file on both the <b>AAmod-Splunk (Ubuntu)</b> and <b>AAmod-ADDC01 (Domain Controller)</b> systems.<br/><br/>

Within the <code>local</code> directory of the <b>Splunk Universal Forwarder</b>, I created and modified the <code>inputs.conf</code> file with the following configuration:
</p>

<pre><code>[WinEventLog://Security]
index = aamod-ad
disabled = false
</code></pre>

<p>
This ensured that all Windows <b>Security Event Logs</b> were forwarded to the <b>aamod-ad</b> index on the Splunk server. After saving the file, I restarted the <b>SplunkForwarder</b> service and confirmed connectivity by allowing inbound traffic on port <b>9997</b> using:
</p>

<pre><code>ufw allow 9997</code></pre>

<p align="center">
<b>Telemetry Confirmation on Splunk Enterprise:</b><br/>
<img src="https://imgur.com/CC4mHKI.png" height="80%" width="80%" alt="Telemetry Confirmation on Splunk Enterprise"/>
<br/>
</p>

<p>
After configuring the <b>Splunk Universal Forwarders</b>, event telemetry successfully appeared in <b>Splunk Enterprise</b>, confirming data flow from both the Domain Controller and Test Machine.<br/><br/>

To validate functionality, I created my first alert in Splunk to detect successful logins from remote sources. The query below identifies <b>EventCode 4624</b> (logon success) with specific logon types and filters out known addresses:
</p>

<pre><code>index="aamod-ad" EventCode=4624 (Logon_Type=7 OR Logon_Type=10) 
Source_Network_Address=* Source_Network_Address!="-" Source_Network_Address!=40.* 
| stats count by _time, ComputerName, Source_Network_Address, user, Logon_Type
</code></pre>

<p>
Since this project was developed in a controlled lab environment, I temporarily relaxed inbound RDP rules for easier remote access and testing.
</p>


<p align="center">
<b>First Shuffle Alert from Splunk After Connecting Webhook:</b><br/>
<img src="https://imgur.com/zb9B9sO.png" height="80%" width="80%" alt="First Shuffle alert from Splunk"/>
<br/>
</p>

<p>
At this stage, I integrated <b>Splunk</b> with <b>Shuffle SOAR</b> to automate incident response. When Splunk detects an unauthorized login, it triggers a <b>webhook</b> that activates a Shuffle playbook. The playbook then sends alerts to <b>Slack</b> and can automatically disable compromised accounts in <b>Active Directory</b>.<br/><br/>

To generate realistic alerts, I used <b>PrivadoVPN</b> to simulate external logins from a different IP address. This confirmed that Splunk successfully forwarded the alert data to Shuffle in real time.
</p>

<h4>Connecting Webhook to Splunk</h4>
<ol>
<li>Create a new Shuffle workflow and copy its <b>Webhook URI</b>.</li>
<li>In Splunk, edit the existing alert and add a <b>Webhook Trigger</b> using that URI.</li>
<li>Start the webhook node in Shuffle to confirm it receives alerts from Splunk.</li>
</ol>

<h4>Connecting Slack to Shuffle (Manual Setup)</h4>
<p>
Due to issues with one-click authentication, I manually connected Slack by creating a custom Slack app. The process included setting redirect URLs, assigning minimal <b>bot token scopes</b> (<code>chat:write</code>, <code>app_mentions:read</code>, <code>users:read</code>), and authenticating through Shuffle using the <b>Client ID</b> and <b>Client Secret</b> from Slack’s API portal.
</p>

<h4>Configuring the Slack Node</h4>
<ol>
<li>Locate the Slack channel ID from the browser URL (e.g., <code>C07N2L4D1O0</code>).</li>
<li>Enter the ID in the “Channel” field of the Slack node.</li>
<li>Set the message text to display key Splunk alert fields:</li>
</ol>

<pre><code>Alert: $exec.search_name
Time: $exec.result._time
User: $exec.result.user
Source IP: $exec.result.Source_Network_Address
</code></pre>

<p>
This configuration allowed the Shuffle bot to automatically post real-time alerts to Slack whenever Splunk detected suspicious authentication events.
</p>


<p align="center">
<b>Shuffle Test — End-to-End Alert Flow:</b><br/>
<img src="https://imgur.com/9Tcdx3U.png" height="80%" width="80%" alt="Shuffle Test p1: Alert taken from Splunk"/><br/>
<img src="https://imgur.com/eobmafC.png" height="80%" width="80%" alt="Shuffle Test p2: Slack Node getting Alert"/><br/>
<img src="https://imgur.com/HlCHg9O.png" height="80%" width="80%" alt="Shuffle Test p3: Alert posted to Slack"/><br/>
</p>

<p>
These screenshots demonstrate a successful end-to-end test of the automation pipeline:
</p>

<ol>
<li><b>Splunk</b> detects a successful remote login and triggers the <b>webhook</b>.</li>
<li>The webhook forwards event data to <b>Shuffle</b>, which processes it through the configured playbook.</li>
<li>The <b>Slack Node</b> sends a formatted alert to the designated Slack channel via the <b>Shuffle Bot</b>.</li>
</ol>

<p>
To generate this alert, I simulated a remote login using a VPN and the test domain user <b>JJohnson</b>. The entire chain—Splunk → Shuffle → Slack—worked as intended, confirming the automation logic and event visibility in real time.
</p>


<p align="center">
<b>Shuffle Email Notification Confirmation:</b><br/>
<img src="https://imgur.com/FoCO7aW.png" height="80%" width="80%" alt="Shuffle Email Notification Confirmation"/>
<br/>
</p>

<p>
Next, I configured <b>Shuffle</b> to send an email prompt to the SOC analyst asking whether to disable the compromised account. This was achieved using the <b>User Input</b> node, which emails a simple decision form to the analyst.
</p>

<ol>
<li>Add a <b>User Input</b> node to the workflow.</li>
<li>Set the prompt text to: <code>Would you like to disable the user? Start parameters: $exec</code>.</li>
<li>Enable the <b>Email</b> option to send the request to the analyst’s inbox.</li>
</ol>

<p>
For testing, I used a temporary email address to confirm delivery and functionality. The next step connected this user action to the <b>Active Directory</b> node, enabling automated account disablement upon analyst approval.
</p>

<p align="center">
<b>Shuffle Final Workflow Visual:</b><br/>
<img src="https://imgur.com/ahSMxQa.png" height="80%" width="80%" alt="Shuffle Final Workflow Visual"/>
<br/>
</p>

<p align="center">
<b>Slack Notification of Account Disabled:</b><br/>
<img src="https://imgur.com/soxKnDN.png" height="80%" width="80%" alt="Slack Notification of account disabled"/>
<br/>
</p>

<p>
This final stage completes the automated response cycle. Once the SOC analyst approves the disable action via email, Shuffle connects to <b>Active Directory</b> and disables the specified domain account automatically. A confirmation message is then posted to <b>Slack</b> to verify completion.
</p>

<h4>Key Configuration Steps:</h4>
<ol>
<li>Link the <b>User Input</b> node to an <b>Active Directory</b> node within Shuffle.</li>
<li>Configure the AD node with the correct <b>LDAP port (389)</b> and <b>base distinguished name (base_dn)</b>.  
   The <code>base_dn</code> can be retrieved by running <code>Get-ADDomain</code> in PowerShell on the domain controller.</li>
<li>Select the “<b>Disable User</b>” action and set <code>SamAccountName = $exec.result.user</code>.</li>
<li>Add a conditional branch that checks whether the <code>userAccountControl</code> attribute contains “ACCOUNTDISABLED.”</li>
<li>If true, trigger a <b>Slack node</b> message confirming:  
   <code>Account: $exec.result.user has been disabled.</code></li>
</ol>

<p>
This logic ensures that account disablement is both automated and verified, demonstrating how a <b>SOAR playbook</b> can integrate Splunk, Shuffle, and Active Directory to simulate real-world incident response workflows.
</p>

<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
