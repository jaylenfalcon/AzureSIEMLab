<h1>Microsoft Azure and Sentinel SIEM Lab</h1>

<h2>Description</h2>
In this lab, we are going to be setting up a virtual machine within Microsoft Azure that will act as a honeypot to capture and log security events using the Microsoft Sentinel SIEM. We will also be using a PowerShell script and a geolocation API to map where attacks originate from. Please let me know if you have any questions!


<h2>Languages and Utilities Used</h2>

- <b>PowerShell</b> 
- <b>Microsoft Sentinel</b> 
<h2>Environments Used </h2>

- <b>Microsoft Azure</b>
- <b>Windows</b>

<h2>Project walk-through:</h2>

<h3>Creating the Virtual Machine</h3>
<p align="center">
<img src="https://i.imgur.com/NG55lKJ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
We want to create an enticing machine that is open to all traffic so that we can generate a lot of alerts and use the PowerShell script to map the location of the attacks. In the portal navigate to “Virtual Machines” and then go ahead and “Create New”.<br/>
<h3>VM Basics</h3>
<p align="center">
<br/> We have to start by creating a Resource group for the VM so hit “Create new” under the resource dropdown and make it whatever, I went with SIEMLab.<br/>
<br/> Next we have to name the VM, I went with honeyhoney-vm as this is a honeypot and the Region I set it to was “US West 3”.
Next you will have to change the Image of the VM and I went with Windows 10 Pro as we will be using the Event Viewer and PowerShell.
After those changes it should look like this:
<img src="https://i.imgur.com/fqaFbFl.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />
Next we just have to create an account to login to the VM. Make this something you can remember as we will have to use it to log into the VM remotely.
Make sure to check the licensing box also. Once done, it should look like this:<br/>
<img src="https://i.imgur.com/ootnevg.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<h3>Networking</h3>
  <p align="center">  
 <br/>Now we can move on to the network settings of the virtual machine.
To open up this machine we have to create a new firewall rule so go the “NIC Security Group” section and change it from “Basic” to “Advanced” then “Create new” under “Configure network security group”.
This will bring up the current firewall rules and we will be deleting the default rule by clicking the three dots/breadcrumbs > “Remove”.
Once done click “+ Add an inbound rule” and change it to match the following:<br/>
<img src="https://i.imgur.com/BpXPTBm.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />This will allow all traffic to get through to the machine, so any ICMP pings, SYN scans, port scans etc. will not be dropped by the machine making it an enticing target for attackers.<br/>
<br/> This is all we need to do to setup this virtual machine so go ahead and click “Review + Create” and get the VM spun up as it will take awhile to deploy.<br/>
<h3>Log Analytic Workspace Setup</h3>
  <p align="center">  
<br/>Go ahead and search for Log Analytics in the Azure search bar and create a new Workspace.<br/>
<br/> Under the Resource group dropdown we will select the group we made in the previous step which is SIEMLab in my case and then under “Instance details” we have to create a name, I went with “honey-law” for Log Analytics Workspace. Then be sure to set the region to whatever it was set to in the previous step (it was “US West 3” for me). It should look something like this:
<img src="https://i.imgur.com/K1fRBvm.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
Once that’s all entered we can go ahead and hit “Review + Create” and then click “Create” to get the L.A.W completed.<br />
<br/> Now we have to finish a few settings to get it to pull logs from our VM. We start by going to “Microsoft Defender for Cloud” within Azure (it’s going to ask if you want to upgrade but just click skip) then under “Management” we go to “Environment Settings”. <br/>
<br/>Once there just click the “Azure Subscription” to reveal the log analytic workspace we just created, click it and it will open up the Defender settings. We just need to turn the “Foundational CSPM” on and we don’t need any servers or SQL servers running.<br/>
<br/>Once turned on go ahead and click Save.
Now we just have to connect the log analytic workspace to our virtual machine so head back the Log Analytics Workspaces page within Azure, click the L.A.W we just made then under “Classic” we see “Virtual Machines (deprecated)”, click this then click on the VM we made and up top is a “Connect” button, go ahead and click that.<br/>
<h3>Microsoft Sentinel</h3>
  <p align="center"> 
<br/>Once the Log Analytic Workspace is connected we can search within Azure for Sentinel. Bring up the Microsoft Sentinel service and click “Create Microsoft Sentinel” and once you do you should see the L.A.W we created, click it and hit add at the bottom.
It will take a few minutes for Sentinel to get added but once it does you will see the Log Analytic Workspace under the Sentinel dashboard. While everything gets connected and the VM is starting to spin up we can move onto the Powershell script we will be using on the VM.<br/>
<h3>Remote Desktop into Virtual Machine</h3>
<p align="center"> 
<br/>Before we can continue we have to get into the VM we created. Within Azure under the Virtual Machines page we can get the Public IP for the VM. Go ahead and copy that and login to it using the built-in remote connection on windows, Remote Desktop Connection (RDC):<br/>

<img src="https://i.imgur.com/vblqIiZ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />Then login with the credentials we made when we created the VM:<br/>
<img src="https://i.imgur.com/D6fWxGh.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br/> Once we enter that we should see the Windows login screen appear in our Remote Connection window:<br/>
<img src="https://i.imgur.com/cVV1EcC.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br/> The first time you login to the VM it will ask you for services and preferences, just select NO for everything.<br/>
<h3>PowerShell Script for GeoLocation</h3>
   <p align="center"> 
<br/>I will be using the script created by Josh Madakor found on GitHub and this will be copied onto our virtual machine so just copy this web link and open it on Edge in the VM then just copy the code directly and then we will open PowerShell ISE on the VM.<br/>
<br/>Once PS ISE is open create a new script (page button with yellow bang, top left) and paste this code into that then save it to the desktop as Log_Exporter or something similar.<br/>
<br/>In order for the geolocation API to work we have to generate our own API key and the link to do so is in the first line of the script.
I found it was easier to do this on the host machine as the VM is very slow but once you navigate to ipgeolocation go to “Sign Up” and create a login.
Once created it will generate your API key, copy this and paste it over the existing key on Line 2 “$API_KEY” in the PowerShell script on the VM.<br/>
<br/>Once this is done we will run the script (green play button) to create a log file in C:\ProgramData which is a hidden folder so be sure you can see it in file explorer (View > check “Hidden items” box). This will show up as “failed_rdp”.<br/>
<img src="https://i.imgur.com/Dm3Oc4L.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br/>While the script is running go ahead and open another RDC window on your host machine and try logging into the VM with a bad password. As soon as you do you should see it pop up in purple text in the PowerShell window. This shows you that the script is working!<br/>
<h3>Pulling the Log Data into Log Analytics Workspace</h3>
       <p align="center"> 
<br/>Now we have to create a custom log in the Log Analytics Workspace within Azure and we essentially have to train log analytics what to look for as the log file is on our VM, not our host machine so in the VM open the “failed_rdp” file and copy its contents.<br/>
<br/>Then we will paste these into a text document on our host machine and save it to desktop as “failed_rdp.log”.<br/>
<br/>Lets head back to our Azure portal and go to the Log Analytics Workspace page and select the LAW we created earlier. Once in, under “Settings” go to Tables > Create > New custom log (MMA-based) then point it to the “failed_rdp.log” we just created on our host machine. This will pull the lines from the file like so:<br/>
<br />
<img src="https://i.imgur.com/zE1f2eV.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br />Under “Collection path” we have to put the path to the log file on our VM in here so select “Windows” then the path is “C:\ProgramData\failed_rdp.log” like so:<br/>
<img src="https://i.imgur.com/92hyEiu.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />
<br/> Then hit “Next” and name the log whatever, I used “FAILED_RDP_GEO” and then hit “Create”.<br/>
<br/>This will take quite awhile to sync up and the logs to be pulled from the VM so give it some time.
Once some time has passed we can check it in our Workspace by going to Logs then in the Query box we can search with our custom log so mine is “FAILED_RDP_GEO_CL” then hit “Run”. You should see the log entries pulled from failed_rdp.log on the VM like so:<br/>
<img src="https://i.imgur.com/LmlPsAA.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br/> The first time you login to the VM it will ask you for services and preferences, just select NO for everything.<br/>
<h3>Sentinel Mapping</h3>
   <p align="center"> 
<br/>Now that the logging is setup and our PowerShell script is up and running we can move to Sentinel and get the mapping setup.
Navigate back the Azure portal and go to the Microsoft Sentinel page. From here we want to go to “Workbooks” and “+ Add workbook”. The workbook comes with a couple default widgets so click “Edit” then click the three dots/breadcrumbs and remove them. Now we will add a query with the following code:<br/>
<br/>Once PS ISE is open create a new script (page button with yellow bang, top left) and paste this code into that then save it to the desktop as Log_Exporter or something similar.<br/>
<br/>FAILED_RDP_GEO_CL | extend username = extract(@"username:([^,]+)", 1, RawData), timestamp = extract(@"timestamp:([^,]+)", 1, RawData), latitude = extract(@"latitude:([^,]+)", 1, RawData), longitude = extract(@"longitude:([^,]+)", 1, RawData), sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData), state = extract(@"state:([^,]+)", 1, RawData), label = extract(@"label:([^,]+)", 1, RawData), destination = extract(@"destinationhost:([^,]+)", 1, RawData), country = extract(@"country:([^,]+)", 1, RawData) | where destination != "samplehost" | where sourcehost != "" | summarize event_count=count() by timestamp, label, country, state, sourcehost, username, destination, longitude, latitude
<br/>
<br/>If you used a different name for your custom log be sure to change it in this code or else you will receive an error. Go ahead and Run the query and it should fire with no errors:<br/>
<img src="https://i.imgur.com/fepB0qb.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br/>If there are no errors then all we have to do is change “Visualization” to “Map” and it will plot the locations for us like so:<br/>
<img src="https://i.imgur.com/htdaAsh.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br/>And it should fill in most of the settings for us but you will have to change the “Metric Label” to “country” but it should look like this:<br/>
<img src="https://i.imgur.com/PMUMYVd.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br/>If everything looks good go ahead and “Save and Close” the map settings then save the workbook. I put it as “Failed RDP Map” and be sure to use the same location (US West 3 for me) and put it in the lab resource group. Once saved be sure to change the “Auto Refresh” to 5 minutes and then we can let this sit for a day or two and log all the failed login attempts from around the world!<br/>
<h3>Conclusion</h3>
  <p align="center"> 
<br/>After a few days of letting this VM run I got quite a few attacks originating from Luxembourg as seen here on the map:<br/>
<img src="https://i.imgur.com/kEkxBS3.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br/>There were all sorts of account names tried which was pretty cool to see and shows you all the different things they will try to brute-force into an unprotected machine like this. Here’s a few snapshots of some the names tried:<br/>
<img src="https://i.imgur.com/SEAk67x.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


<!--
 ```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```
--!>
