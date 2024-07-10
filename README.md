<h1>Microsoft Azure Honeypot</h1>
<a href="https://drive.google.com/file/d/1HmgOkRuU45BdccCwlngrir2C7a-qNstd/view?usp=sharing">.pdf version</a>

<h2>Description</h2>
In this project, I created a virtual machine hosted on Microsoft Azure to act as a honeypot forattackers attempting to use the Remote Desktop Protocol (RDP) to log in. All firewalls on the virtual machine were disabled, and the Azure security group associated with it allowed any traffic to pass through. As a result, people from all over the world attempted thousands of RDP connections to the VM. Using a PowerShell script, log data from Windows Event Viewer was extracted and sent to Azure Log Analytics for processing. A query was used to filter all data that Azure Sentinel, the SIEM, could use to display each event on a world map.

<br />
<br />
<p align="center">
<img src="https://i.imgur.com/v7wov2g.png"/>

This document details my process of setting everything up. All credit goes to Josh Madakor and his excellent <a href="https://www.youtube.com/watch?v=RoZeVbbZ0o0">YouTube guide</a>. Unfortunately, the video is outdated, and Microsoft has made many changes to Azure since its release. I was able to bridge the gaps and set up the lab with the same functionality. Please note that the rest of the document is written in a way that most people should be able to follow. Thank you for taking a look!

<h2>Skills Used</h2>

- SIEM
- Firewalls
- Microsoft Azure
- Log Analysis
- Remote Desktop

<h2>Environments Used</h2>

- Microsoft Azure
- Windows 10 Pro

<h2>Walkthrough</h2>

<b>Steps</b>
1. Create a new Microsoft Azure account and start the free pay-as-you-go trial <br />
2. Create a virtual machine <br />
3. Create a Log Analytics workspace <br />
4. Configure Microsoft Defender for the Cloud settings <br />
5. Connect Log Analytics workspace to your virtual machine <br />
6. Configure Azure Sentinel (SIEM) <br />
7. Log into your virtual machine <br />
8. Turn off the virtual machine’s firewall <br />
9. Set up the PowerShell script to collect log information <br />
10. Get geolocation.io API key <br />
11. Create a custom log in Log Analytics workspace <br />
12. Configure Azure Sentinel notebook <br />
13. Wait for people to discover your honeypot and attempt to RDP into it <br />
14. Delete all resources when finished <br />

<h3>1. Create a new Microsoft Azure account and start the free pay-as-you-go trial</h3>
<p align="center">
<img src="https://i.imgur.com/u3AlzvB.png"/>
  
<h3>2. Create a virtual machine</h3>
<h3>3. Create a Log Analytics workspace</h3>
<h3>4. Configure Microsoft Defender for the Cloud settings</h3>
<h3>5. Connect Log Analytics workspace to your virtual machine</h3>
<h3>6. Configure Azure Sentinel (SIEM)</h3>
<h3>7. Log into your virtual machine</h3>
<h3>8. Turn off the virtual machine’s firewall</h3>
<h3>9. Set up the PowerShell script to collect log information</h3>
<h3>10. Get geolocation.io API key</h3>
<h3>11. Create a custom log in Log Analytics workspace</h3>
<h3>12. Configure Azure Sentinel notebook</h3>
<h3>13. Wait for people to discover your honeypot and attempt to RDP into it</h3>
<h3>14. Delete all resources when finished</h3>

<!--

<br />
<br />
<p align="center">
<img src="" height="80%" width="80%"/>

<p align="center">
<img src=""/>

</p>
--!>
