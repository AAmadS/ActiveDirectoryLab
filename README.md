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

- <b>Windows 10</b> (21H2)

<h2>Project walk-through:</h2>

<p align="center">
Creating a Diagram with Draw.io: <br/>
<img src="https://imgur.com/SFquO3T.png" height="80%" width="80%" alt="AD Project Steps"/>
<br />
 
<p>
 
This diagram illustrates a security workflow for detecting and responding to unauthorized logins in an Active Directory environment. The setup includes three virtual machines hosted on VULTR—a Domain Controller, a Test Machine, and a Splunk server—along with an attacker machine simulating an external threat actor.

The attacker, using valid credentials, successfully authenticates to the Test Machine. This event generates telemetry, which is forwarded to the Splunk server. Upon detecting the login, Splunk triggers an alert that is sent to Slack and also initiates a Shuffle automation playbook.

The Successful Unauthorized Login Playbook sends an email to the SOC Analyst, asking whether the suspicious user account should be disabled.

- If Yes, Shuffle proceeds to disable the domain account in Active Directory and sends a confirmation notification to Slack.
- If No, no further action is taken.
</p>

<br />
Select the disk:  <br/>
<img src="https://i.imgur.com/tcTyMUE.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Enter the number of passes: <br/>
<img src="https://i.imgur.com/nCIbXbg.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
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
