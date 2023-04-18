# SIEM-Tutorial-using-Sentinel
<p align="center">
<img src="https://imgur.com/NFhd2Vo.png" alt="Microsoft Sentinel Logo"/>
</p>

<h1>Using Azure Sentinel to monitor live attacks</h1>
This tutorial outlines the implementation of Azure Sentinel and the creation of a honeypot.<br />



<h2>Environments and Technologies Used</h2>

- Microsoft Sentinel (SIEM)
- Microsoft Azure (Virtual Machine)
- Remote Desktop

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)

<h2>Deployment and Configuration Steps</h2>

What we'll being doing is creating a virtual machine using Azure, which will act as our honeypot. Then we will use Sentinel to monitor and log attacks coming from all over the world trying to gain access to our vulnerable machine. So first, lets create our virtual machine that will be exposed to the internet.
<p>
<img src="https://imgur.com/2oBZlEv.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
</p>
<br />

Continue to "Networking" and scroll to "NIC network security group" and choose advanced. Click on "Create New". Remove the default inbound rule and then add a new inbound rule.  
<p>
<img src="https://imgur.com/2YE7ECP.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
Remove the default inbound rule. We will create a new inbound rule which will allow all traffic from the internet into the VM. This will make it easily discoverable. Create the VM and let's move on to the next step.
<p>
<img src="https://imgur.com/R5EZQlp.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<br />

Next, we're going to create a Log Analytics Workspace so that we can ingest logs from our VM. Then we're going to create our own custom log that contains geographic information. Logs will be stored here and our SIEM will connect to this workspace and then be able to display geodata on a map. So, naviagte to Log Analystics Workspace on Azure and create one.
<p>
<img src="https://imgur.com/a5wLtZJ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>

</p>
<br />
Next, we have to enable the gathering of our VM's logs into the workspace. Navigate to "Microsoft Defender for Cloud" then click "Environment Settings". Choose the workspace we created and go to Defender plans and turn off "SQL servers on machines". Save changes.
<p>
<img src="https://imgur.com/kdBFwyt.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
After, go to "Data Collection" and choose "All Events". Save changes.
</p>
<img src="https://imgur.com/5FCuPYG.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

Next, we are going to connect our VM to our workspace. Naviagate to Log Analytics Workspace, then look for Virtual machines. Connect the VM. After that's finished connecting, we're going to set up Sentinel. (Its been renamed Microsoft Sentinel at the time of this being made.) Type in Microsoft Sentinel in Azure and create. Add it to our workspace.
<p>
<img src="https://imgur.com/VuTqDrL.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://imgur.com/RYxtGU0.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<br />

Cool! Now we're going to connect to our VM with RDP using the credentials created when we created the VM. Also, go ahead and purposely fail a logon to the VM. Inside the VM, navigate to Event Viewer. Click "Windows Logs", then "Security". We can click on any of the events and get some more information. We are going to look at one with an Event ID of 4625. This shows our failed logon.
<p>
<img src="https://imgur.com/BWcG9aI.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://imgur.com/rIKp47N.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<br />

Okay, next we're going to turn off Windows Firewall on the VM so it can get discovered faster on the internet. Naviagte to Windows Defender Firewall and turn them off for the Domain, Private, and Public profiles. Before doing this, I initiated a ping -t command on my host computer to the VM to show that it was discoverable.
<p>
<img src="https://imgur.com/z1QtU7d.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://imgur.com/N8bdIxu.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<br />

Next, go here: https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1 and copy the Powershell Script. Open up PowerShell ISE on the VM and paste the script there. The API key will be different for everyone so go to: https://ipgeolocation.io/ and sign up to get your own API key and then paste it into the script. After, run the script. The script looks through the event logs and grabs all the failed logons, gets the IP addresses of them and geodata, and creates a new log file.
<p>
<img src="https://imgur.com/FsZ8CNh.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://imgur.com/VYX7pgF.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<br />

After, we're going to go back to Azure and create a custom log in our Log Analytics Workspace so that it can bring the custom log with the geodata into our workspace. Go to "Tables" then create a custom log (MMA-based). For the file, we're going to use the failed_rdp file created from the script earlier that can be found in the programdata folder on the VM. Copy the contents of the file and paste it to a notepad on your host machine. Then we can use it as our sample log. For the Collection path, it should look somethiing like this: C:\ProgramData\failed_rdp.log. Then create the log.
<p>
<img src="https://imgur.com/vzQyNKG.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://imgur.com/QBzqVHv.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<br />
After 10 minute or so, run the custom log and we can see that they are coming through. Next, right-click on one of the logs and click "Extract fields...". Then for latitude, longitude, destination host, username, sourcehost, state, country, label, and timestamp, right- click the section and extract it. Search results show up after doing so.  
<p>
<img src="https://imgur.com/QnnYEjC.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://imgur.com/ZdBHQFG.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<img src="https://imgur.com/Wma1T5F.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<p>
<img src="https://imgur.com/3gnP0AD.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

When running the custom log again, click columns and check the fields we extracted so that the data is sparsed from the logs. After that, navigate to Sentinel and add a workbook. Then add a query. Copy the third slide below and then run.
<p>
<img src="https://imgur.com/MkfnDXt.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://imgur.com/1oetWky.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<img src="https://imgur.com/1oetWky.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

Click Visualization and choose "Map". Then for map settings, input the fields we extracted for longitude and latitude. Scroll further down and change metric label and metric value. Save the workbook. Now we wait for the VM to try to be logged into from the internet. We can wait a day or a couple hours. 
<p>
<img src="https://imgur.com/zC9NfUP.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
<img src="https://imgur.com/zXYFbCO.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<img src="https://imgur.com/SEdsGsT.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
<br />

Okay, coming back after about 7 hours, we can observe that attacks have come in from Vietnam, India, Russia, and Germany. The majority of attacks have come from Vietnam which is pretty interesting. If we kept the VM running longer, we can imagine that there would be more attacks logged. This goes to show that once something is on the internet, people will try to gain access no matter what. 
<p>
<img src="https://imgur.com/1GnV4KZ.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>
</p>
<p>
</p>
<br />
Hope you enjoyed this lab and learned something! 


