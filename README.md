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
<img src="https://i.imgur.com/u3AlzvB.png"/>
This was all possible due to the power of cloud computing, and for this lab we are using Microsoft Azure. Through Azure, we can create a machine (computer) separate from the one we use at home. Since this lab will be done on that separate machine, our own computer will be safe from attackers. Azure offers a free trial up to $200, which is more than enough for this lab. After creating an account and providing your information, you will have access to the Azure homepage. The search bar at the top will be helpful for finding all the different services that will be used in this lab.
  
<h3>2. Create a virtual machine</h3>
Now, let’s set up our virtual machine, or honeypot (a virtual trap to lure attackers in).
<br />
<br />
<img src="https://i.imgur.com/cQiMTBd.png"/>
<img src="https://i.imgur.com/GXBbwvk.png"/>

The above pictures show the settings that I configured in my environment. You’ll be prompted to
choose a resource group, which is where we will put all our lab creations under. After, you’ll
choose a name for the machine and the region it will be hosted in. I put mine in Central US
because it supports Windows 10 Pro. Please note that different regions support different
operating systems and machine sizes.
<br />
<br />
You can leave the default settings for everything else. When configuring the Administrator
account, please pick a username and password that you’ll remember. I recommend that they be
something that you don’t already use in real life, because attackers will be actively trying to
guess the correct ones.
<br />
<br />
On the Networking tab, you’ll be prompted to create a virtual network that your machine will
access the internet with, and an IP address that it will use to connect to the internet with. As a
result, your actual internet connection will not be affected by anything that happens within the
virtual machine.
<br />
<br />
Click “Advanced” for the “NIC network security group.” This is essentially a type of firewall for
the virtual machine to block any unknown or malicious people who want to get in. Normally, this
is a good thing. For this lab, however, we must configure it in a way that is intentionally not
secure so that people can try to connect to it.

<img src="https://i.imgur.com/R71Dg75.png"/>

Afterwards, you will be prompted to create a new network security group. These are essentially
the instructions for the firewall to block certain IP addresses from connecting to your computer.
Delete the default inbound rule and create a new inbound rule with the same settings in the
picture above – any source, any (*) source port range, any destination, custom service, and any
(*) destination port range. Click Add and then OK.
<br />
<br />
At this point, you can create the virtual machine; the rest of the settings don’t pertain to the lab.

<h3>3. Create a Log Analytics workspace</h3>
While the virtual machine is being created, we’re going to create a new Log Analytics
workspace. A log is a record of an event, such as information about someone using the wrong
password for an account. For a typical home computer, these are stored somewhere in the local
storage. With Log Analytics, we can retrieve the logs from our virtual machine and store them in
the Azure cloud without having to go inside the computer manually

<br />
<br />
<img src="https://i.imgur.com/OPnYU0F.png"/>

<h3>4. Configure Microsoft Defender for the Cloud settings</h3>
To ensure that Log Analytics gathers all the required logs, we must configure some settings in
the Microsoft Defender for the Cloud service.
<br />
<br />
<img src="https://i.imgur.com/Hk4VsRw.png"/>

On the left-hand side, click “Environment settings.” Click the drop-down arrow next to your
subscription (in my case, “Azure subscription 1”) to expand and reveal the Log Analytics
workspace you just created. Click on it.
<br />
<br />
<img src="https://i.imgur.com/d1fkJX4.png"/>

In the first menu that appears, “Defender plans,” uncheck “SQL servers on machines.” We won’t
be needing it for this lab. Afterwards, click “Data collection” on the left-hand side. Ensure that
you save the configuration.
<br />
<br />
<img src="https://i.imgur.com/mXDkGQ5.png"/>
In this menu, we want to configure “All Events.” This ensures that Log Analytics will collect all
the data we require for the lab.

<h3>5. Connect Log Analytics workspace to your virtual machine</h3>
For Log Analytics workspace to start collecting logs from our virtual machine, we must connect
the two together. Open the Log Analytics dashboard again and open your workspace. There will
be a tab labelled “Virtual Machines (deprecated)” that you will click on. 
<br />
<br />
<img src="https://i.imgur.com/0IJW6Lv.png"/>

Select your virtual machine and click “Connect.”
<br />
<br />
<img src="https://i.imgur.com/smiEiOT.png"/>

This may take some time, so feel free to move onto the next step.

<h3>6. Configure Azure Sentinel (SIEM)</h3>
Sentinel is where we will be querying the data we get from our Log Analytics workspace and
display it on the world map. A query is a way to retrieve specific information from a set of data,
like our logs. For example, we will later use a query that retrieves the country of the attackers.
Activating Sentinel is quick and painless – simply navigate to the Sentinel dashboard and click
“Create Azure Sentinel.” After, click your Log Analytics workspace and add it to your Sentinel.
<br />
<br />
<img src="https://i.imgur.com/KsFQWnV.png"/>

<h3>7. Log into your virtual machine</h3>
To log into your virtual machine, use a program called “Remote Desktop Connection” on your
computer; it’s installed on there by default.
<br />
<br />
<img src="https://i.imgur.com/i6cQA7k.png"/>

Once you open the program, enter the public IP address of your virtual machine. To log in, use
the username and password that you configured when first creating the virtual machine. You can
find the public IP address in the Virtual Machine dashboard.
<br />
<br />
<img src="https://i.imgur.com/oqFGjF7.png"/>

Ensure that the machine status is “Running.” It may take a few minutes for you to be able to log
in and start using the machine.

<h3>8. Turn off the virtual machine’s firewall</h3>
We must turn off the firewall on the virtual machine. This will enable attackers from all over the
world to try and access it. Search up “wf.msc” in the Start Menu and click on “Windows Firewall
Properties.”
<br />
<br />
<img src="https://i.imgur.com/YFTbByz.png"/>

Ensure that you disable the domain, private, and public firewalls. Click “Apply” and then “OK.”

<h3>9. Set up the PowerShell script to collect log information</h3>
PowerShell is the Windows version of a shell. A shell is essentially a window where you can
input commands to tell the computer to do something. A script is a set of instructions that the
shell will follow automatically.
<br />
<br />
In Josh Madakor’s video, he provides us with a script that he wrote, which can be found here.
This script processes the raw data from the logs and turns it into something more humanreadable.
<br />
<br />
The easiest way to get this script is to copy the whole script in the GitHub link above and paste it
directly into the virtual machine. To do this, open PowerShell ISE in your virtual machine. You
may need to right-click and select “Run as Administrator” for it to work properly. Paste in the
code here and save the file onto your Desktop.
<br />
<br />
<img src="https://i.imgur.com/TGivCkM.png"/>

<h3>10. Get geolocation.io API key</h3>
geolocation.io is a free online tool that enables us to input an IP address and look up the country,
state, latitude, and longitude associated with it. To get it to work with our script, you must sign
up and get your API key. This key allows you to make use of the location tool in the script.
On the first and second line of the PowerShell script, it mentions:
<br />
<br />
# Get API key from here: https://ipgeolocation.io/
$API_KEY = "d4600b4efdef42b39828f5155041a457"
<br />
<br />
<img src="https://i.imgur.com/zxuAI3W.png"/>

Once you create an account on the geolocation.io website, you will be presented with your API
key. Replace the example API key in your script with your actual key – it will look like an odd
string of letters and characters like in the script’s example. Ensure that you save the changes.
<br />
<br />
On the top of the PowerShell ISE window, there will be a green button to start the script. Press
the button so we can start collecting data. If you want to test if it works, you can open another
Remote Desktop window on your actual computer. However, this time you must intentionally put
in a wrong username and/or password so that it shows up on your virtual machine’s log. If
configured correctly, there will be a log entry in the console of the PowerShell ISE.

<h3>11. Create a custom log in Log Analytics workspace</h3>
We’ll be creating a custom log in our Log Analytics workspace so that the formatting will be
consistent with the logs in our virtual machine. Open your workspace in Log Analytics and select
the “Tables” tab. Click “Create” and then “New custom log (MMA-based).” MMA is a Microsoft
service that monitors resources.
<br />
<br />
<img src="https://i.imgur.com/I7zgB2H.png"/>

Inside your virtual machine, search “Run” in the start menu and enter “C:\ProgramData\” as the
destination. A File Explorer window will open and you should see a file named “failed_rdp.log.”
Open it and copy the entire contents (Ctrl+A and Ctrl+C is the shortcut for this).
<br />
<br />
Back on your actual computer, open Notepad and paste the contents inside. Click “File” then
“Save as.” Name it “failed_rdp.log” and ensure that you save as “All files” type.
<br />
<br />
The file you just created will be the sample log that Log Analytics is asking for.
<br />
<br />
<img src="https://i.imgur.com/omRf36H.png"/>
<br />
<br />
On the next page, verify that the data is properly formatted like in the picture below.
<br />
<br />
<img src="https://i.imgur.com/nhenJ2D.png"/>
<br />
<br />
On the next page, set “C:\ProgramData\failed_rdp.log” as the Path and “Windows” as the Type.
Name and create the custom log – take note of the name for the next step. It will take a while for
data to be synced between your virtual machine and your workspace. For me, it took at least 30
to 40 minutes for logs to start appearing.
To check if the data is done processing, you can open the “Logs” tab on your workspace and
simply query with the name of your custom log (“FAILED_RDP_WITH_GEO_CL” in my case).
Click “Run.” If the data is available, it will show up below.
<br />
<br />
<img src="https://i.imgur.com/87xficz.png"/>

<h3>12. Configure Azure Sentinel notebook</h3>
We’re almost done! You can minimize the virtual machine for now. On your actual computer, go
back to the Azure Sentinel dashboard.
We’ll be creating a Sentinel workbook. A workbook enables us to process the data from our Log
Analytics workspace and display it on the world map.
Select your Log Analytics workspace and click the “Workbooks” tab. Create a workbook. There
will be a couple of default widgets there – delete them and add a query.
<br />
<br />
<img src="https://i.imgur.com/MKbSlVv.png"/>
<img src="https://i.imgur.com/TKdk4Ay.png"/>
A menu like the one in the picture above will appear. Set the time range to “Last 30 days,” the
visualization to “Map,” and size to “Full.”
<br />
<br />
We’ll finally be adding our query. This query further processes the data from our Log Analytics
workspace for Sentinel to use. This query categorizes the data into groups such as timestamp,
latitude, longitude, and country.
<br />
<br />
Copy the following into the query box and ensure that you replace
***CUSTOM_LOG_NAME_HERE*** with the name of the custom table you created in Log
Analytics. Replace ***VM_PUBLIC_IP_HERE*** with the public IP address of your virtual
machine. This is to exclude the test failed logins you may have done while checking if the
PowerShell script works.
<br />
<br />
***CUSTOM_LOG_NAME_HERE***
|extend username = extract(@"username:([^,]+)", 1, RawData),
 timestamp = extract(@"timestamp:([^,]+)", 1, RawData),
 latitude = extract(@"latitude:([^,]+)", 1, RawData),
 longitude = extract(@"longitude:([^,]+)", 1, RawData),
 sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData),
 state = extract(@"state:([^,]+)", 1, RawData),
 label = extract(@"label:([^,]+)", 1, RawData),
 destination = extract(@"destinationhost:([^,]+)", 1, RawData),
 country = extract(@"country:([^,]+)", 1, RawData)
|where destination != "***USERNAME_YOU_TESTED_WITH_HERE"
|where sourcehost != "***VM_PUBLIC_IP_HERE***"
|summarize event_count=count() by timestamp, label, country, state, sourcehost, username,
destination, longitude, latitude
<br />
<br />
Run the query, and the map should appear. In the map settings, configure them as follows to
correctly display your data.
<br />
<br />
<img src="https://i.imgur.com/56tLR2C.png"/>

<h3>13. Wait for people to discover your honeypot and attempt to RDP into it</h3>
The setup is complete! You may safely close the Remote Desktop connection – the virtual
machine and the script inside will keep running. It may take a few hours for attackers to start
attempting to connect into the virtual machine. If you log back on and find that the script stopped
running, you can simply start it up again to continue gathering data.
<br />
<br />
Once failed connections start to come in, they will be shown on the map. You may have to
refresh the map for it to update. On your virtual machine, the PowerShell ISE will also output
logs to the console.

<h3>14. Delete all resources when finished</h3>
When you’re finished experimenting with the lab, I recommend deleting all the resources. I’ve
had this honeypot up for two days at the time of writing this, and it’s only used up $3 of the $200
free trial credit. Though it would take a while for you to burn through the $200 in the free trial, it
would be unfortunate to forget about it and have an unwanted charge to your card later.

<h3>Final Words</h3>
This wouldn’t be possible if not for Josh Madakor and his spectacular guide on YouTube. I
learned a great deal from this project, most notably Microsoft Azure as a cloud service and how
you can set up an entire process for data to be collected, processed, then sent to a SIEM for
further analysis.
<br />
<br />
This was a lot of fun to work on and motivated me to work on further projects. I’m also
interested in expanding upon this one, possibly to include logs from other protocols such as SSH
or Telnet.
<br />
<br />
If you’ve made it to the end, thank you again for taking a look at my write-up on this project! I
hope it provides detailed insight on how I was able to apply a lot of my personal studies into a
real-world scenario. Most importantly, I hope you found it interesting!
