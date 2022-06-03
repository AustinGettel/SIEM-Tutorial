# SIEM-Tutorial
SIEM Tutorial for Beginners | Set up live SIEM | Azure Sentinel MAP with LIVE CYBER ATTACKS

In this lab we're going to create our own SIEM environment on an Azure cloud to get experience working with our own. We're going to set up our VM environment, and turn off the firewalls to make our VM very appealing to global attackers. We'll then creat a log repository in Azure, i.e. Log Analytics Workspace, to process the logs from the VM. Lastly we will set up our SIEM by using Microsoft's Azure Sentinal to map our attackers.

<img src="https://user-images.githubusercontent.com/106196315/171706902-ea910635-abad-4d56-a417-820135868248.png" width="600" height="300" />

And this lab will be done with powershell. This will allow us to capture the attacker's IPs, which we can then send to a 3rd party API to find their locations, and send back to the VM to log the attacks along with geographical data.

To get started make you free [Microsoft Azure account here](https://azure.microsoft.com/en-us/free/).

<!---remember to tell everyone to delete the resources they make so it dosn't cost them--->

Once you're done signing up for azure go to your portal by going to [portal.azure.com](https://portal.azure.com). In the search bar at the top type in virtual machine, and click on the virtual machines options under services. 

![make new vm](https://user-images.githubusercontent.com/106196315/171706369-f79d24a3-8b89-463e-940b-5f5c5ae237a3.png)

Make a new resource group, call it honeypotlab.

Name your VM, I named mine honeypot-vm

I put my region as East US 2, and chose Zones 2.

Leave Security as standard. For the Image go with Windows 10. We don't need insurance.

Size will fluctuate, what it defaults to. 

<!--- I had the worst time trying to find a size that would work. I tried all the US regions, and no luck. I found a few sizes that were obcsenely large and would have eaten through my $200 credit in a matter of hours. I eventually found ones that worked, but that was after an hour of messing with my account subscription, changing to pay as I go, and just combing through the different regions and zones. I eventually found that is an issue with the US regions being at max capacity. If you are experiencing this issue, try changing to a none US region.--->

Do remember the username and password you chose, that will be needed to sign in.

![creating vm - basics](https://user-images.githubusercontent.com/106196315/171733302-ec534a17-a31a-4b0f-920e-5514d853fa01.png)

Hit the box for licensing and click next till you get to networking.

The NIC security group is a firewall. We're going to make a new one, so hit the radio button for advanced, and click on Create New.

![creating vm - create new security](https://user-images.githubusercontent.com/106196315/171733821-be7b0df3-a510-413d-b196-f312b0428f8e.png)

We want to make make our own inbound rule to make our VM susceptible to attacks; click the dot on the right and hit Remove, then add an inbound rule.

Replace the destination port ranges with an * for anything, any protocol, action needs to be allow, change priority to something low, like 100, name it.

<img src="https://user-images.githubusercontent.com/106196315/171736014-21fe5b50-3c12-4bbb-929f-575c947b74fb.png" width="300" height="600" />

Review and create.

While we're waiting on the VM to deploy, let's make our log analytic workplace. Type log analytic workplace into the search bar at the top, select it, and click create log analytic workplace.

![creater law](https://user-images.githubusercontent.com/106196315/171747911-cf237b8d-9873-4cde-9fdb-ef6a1dfac84f.png)

The workspace is what logs the attacks to our VM, and we'll connect it to our SIEM to display the attacks on a map. 

Use you're honeypotlab resource you created during mdeploying the vm, law-honeypot1 (law for Log Analytic Workspace). Keep the region what you used earlier, for me that's East US 2. Review + Create.

Let's move on while this is deploying. Back at the top type defemder, and select Microsoft Defender for the Cloud. If you get a page about upgrading just skip. On the left select Environment Settings, hit the Expand All button, and select the resource you made. We don't need the SQL server, so only turn Defender on for Servers. Hit the save icon at the top.

On the left go to Data collection, select all events, and save.

Go back to you LAW. It's time to connect the LAW to the VM. click on your LAW,  select VMs on the drop down, and click on your VM. Press the connect button.

![connect](https://user-images.githubusercontent.com/106196315/171750813-7de0de57-e1e2-48c9-9609-a2ebc6183b58.png)

wait for it to connect, then type sentinel into the top. Select Sentinel. Again, this is our SIEM. Create Microsoft Sentinel.

![create microsoft sentinel](https://user-images.githubusercontent.com/106196315/171751263-102ac68c-1a6d-4f2f-97b6-d2a900e124c4.png)

This page is to add sentinel to our workspace. Click on the workspace and hit the add button.

<!---WHY IS THE LAW I ALREADY SET UP NOT SHOWING?! If you're having trouble finding your workspace while setting up sentinel, click on create a new workspace. Now go through the set up just like you did earlier in the same group and with the same name. It should give you an error saying you can't use the same name. Good, now exit out of this and refresh the add sentinel to workspace page. Your workspace should now appear.--->

So, is your VM connected to your LAW and the LAW connected to Sentinel? Good. Time to log into the VM. Pull up the VM, and copy the public VM address. Now open remote desktop connection on your windows computer. Paste your IP and connect.

<!---you can change whatever settings you'd like here. for instance I didn't feel like having mine fullscreen--->

![enter your credentials](https://user-images.githubusercontent.com/106196315/171766752-635d82e3-47c8-4ce7-83f4-a018553f65d7.png)

On the credentials sign in page, if your Microsoft account is appearing like in the screenshot above, click the More Choices option, select use a different account, and log in with the username and password you made the VM with. Click Yes on the security warning.

Alright, now in your VM go ahead and set up Edge because we do have to use it later. Don't sign into anything.

Open the start menu, and type in Event Viewer. Event Viewer may take a minute to load, but once it does click windows logs on the left, and select Security.

<img src="https://user-images.githubusercontent.com/106196315/171781972-ecb92d19-b271-433a-a0e5-5467b8f68828.png" width="700" height="500" />

In the browser on the VM, go to [ipgeolocation.io](https://ipgeolocation.io/). This is the 3rd party API we're going to use to collect the geographic locations from our would-be attacker's IP addresses so we can plot them on a map in Sentinel. When we type an IP address in you can see below that it returns much more information then Window's Event Viewer.

<img src="https://user-images.githubusercontent.com/106196315/171783048-313e64f0-e6a2-4cb5-a646-0b197746dd49.png" width="300" height="450" />

Next step is turn off the VM's Windows Firewall. First, just to show that it's not open yet, I'm going to ping the VM from my computer. This should fail.

![ping vm](https://user-images.githubusercontent.com/106196315/171784713-86cca85a-3d3b-40fb-a273-e0f1ac68bf06.png)

<!--- the -t command after the IP was to keep the ping going untill stopped. I hit ctrl+c to stop the pings--->

Good, it failed. Back to the VM. In the search bar type wf.msc.

<img src="https://user-images.githubusercontent.com/106196315/171785210-b28aa35c-562d-4b77-98ad-d827bd352bee.png" width="550" height="450" />

Open windows firewall, and select Windows Defender Firewall Properties.

![firewall properties](https://user-images.githubusercontent.com/106196315/171785479-404b55e2-1c2a-4ced-a536-befaf320bff2.png)

Turn firewall state to off on all three profiles (Domain, Private, and Public). Hit Okay.

<img src="https://user-images.githubusercontent.com/106196315/171785825-200061d7-12e5-43dc-a29b-8af39d7ee80b.png" width="550" height="450" />

Back on my computer I'm going to ping the VM again.

![ping vm2](https://user-images.githubusercontent.com/106196315/171786100-53455a97-e9c0-4963-8730-a9c191aa9ba6.png)

Alright, this time it worked. The firewall is down! No more use for the cmd, gonna close that.

Now we need to download [this powershell script](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1) from github on to the VM. I didnot write this script, full credit goes to Josh Madakor.

On the VM we can close the event viewer and firewall. Download or copy the script. Open Windows Powershell ISE on your VM.

![powerrshell ise](https://user-images.githubusercontent.com/106196315/171787054-c4cf14e0-1fe1-431d-a83e-b45d05b77751.png)

Click the New Script icon at the top.

![new script](https://user-images.githubusercontent.com/106196315/171787358-54d56665-2c55-4a39-b2a8-d61901d19bc2.png)

Paste the script. Save on the desktop as Log_Exporter.

<!---or don't, save it whatever you like. it doesn't matter--->

