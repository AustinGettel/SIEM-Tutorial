# SIEM-Sentinel-Tutorial
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

Good, it failed. Back to the VM. In the search bar type wf.msc

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

Paste the script. Save on the desktop as Log_Exporter

<!---or don't, save it whatever you like. it doesn't matter--->

<!---actually please save. I now have had to go through this lab from this point 2 times, and lost my prgoress both times, and am starting a 3rd time, because I didn't save it right--->

Now we need to head back to [ipgeolocation.io](https://ipgeolocation.io/). Make a free account so you can get an API key. If you don't do this, you won't be able to get the location data, and will not be able to log the attacks on a map. Just replace the API key from the first line with your new key.

![api key](https://user-images.githubusercontent.com/106196315/172697591-e092e646-6e31-4570-8092-a7fa5788cb51.png)

It's now time to run the script. You will see while it's running that it's collecting the 4625 EventIDs that we saw earlier from the Event Viewer, but also with location data provided from [ipgeolocation.io](https://ipgeolocation.io/).

Now go to the hidden folder ProgramData under your C drive. Here is where our script will create a text document log file with all the results. It will be called failed_rdp.log

![hidden programdata](https://user-images.githubusercontent.com/106196315/172700908-e605c6a6-ec35-41d6-b620-83d3896d965a.png)

Open the log, and ctrl+a & ctrl+c to copy it's contents. Back on your computer make a new text document, title it failed_rdp.log, paste the contents from the log on the VM, and save this file on your desktop.

Now back on azure, go to your law, and on the left menu select custom logs.

![workspace custom logs](https://user-images.githubusercontent.com/106196315/172884804-e4f3a051-698d-4b9c-9f10-7663f96b919d.png)

Add a custom log, and azure will ask for a sample log. Select the failed_rdp.log file that we just made on your desktop that is a copy of the VM. The next screen will show the log sample you gave for review. The collection path is asking how to find this log on the VM, select Windows, and type 

C:\ProgramData\failed_rdp.log

<!---typing is very important here, any mistake will prevent the query on the next step from being able to collect the log data--->

![collection path](https://user-images.githubusercontent.com/106196315/172886784-8bca84eb-8c8c-4f68-8950-3d8654a206b9.png)

<!---can you imagine why I hard time on my first try--->

Name your log FAILED_RDP_WITH_GEO (the CL will add itself on the end making the full title FAILED_RDP_WITH_GEO_CL)

On the left menu go to logs.

The query will for our custom log will take some time to load. While waiting, we should try a queary on the Windows Security Event logs we looked at earlier. On line 1 of the query type

SecurityEvent | where EventID == 4625

![security event](https://user-images.githubusercontent.com/106196315/172890845-c85ae1df-e92d-47b1-ab87-a6069d57d302.png)

This will show you a list of all the failed attempts to sign in on your honeypot, just without the geolocations from the script.

Take a coffee break, and in about 10 minutes try the custom log we made.

<!---took me 2 days to get back to this. hopefully not so long for you--->

Now we need to extract the data from the RawDate feild to have their own fields. To do so, right click on one of the logs, and select Extract fields.

![extract fields](https://user-images.githubusercontent.com/106196315/173198054-af23e147-7612-44fe-995e-f50f88536bc3.png)

To extract the raw data and turn into a field highlight the data after the :

![highlighted field rawdata](https://user-images.githubusercontent.com/106196315/173198123-365a95d3-55c4-4292-96c3-5c3fd55a98d7.png)

Title the field latitude and change the field type to numeric. Review the search results, and if they all look right then save extraction. 

![field value](https://user-images.githubusercontent.com/106196315/173198186-1f09633e-2cfd-424f-9a00-9fc71a51d1fa.png)

<!---If you're like me, and taking your sweet, sweet time doing this (or really like me and getting very aggrevated at the limited time to do this), then you may also have an obscene amount of hits to work with. I had over 15k events coming up in my query, and the custom fields failed to learn anything. So, turn off the script, and came back in a day. start it up, wait till you have a few hits and it's updating your text doc log on the VM. Give Azure another 10 minutes, and try again. you should have a much smaller query and azure will be able to learn for you.--->

Now repeat with longitude (numeric value), destinationhost (this is our target VM (text value)), username (text value), sourcehost (text value), state (text value), country (text value), label (this includes the country again and the IP address and will be used to label locations on the map (text value)), and the timestamp (date/time value).

When I went to extract longitude, you'll see it's accuractly extracting longitudes with - before them, but if they didn't have the - they're finding the next section with - in my caser I got 76 matches with -vm.

![traingin the algo](https://user-images.githubusercontent.com/106196315/173200033-9f7b3b9c-8019-494e-90be-752b79fe7e11.png)

To fix this, we'll train the algorithm to find what we want. Click the circle icon on the right on one of the searches with a bad result. Select Modify this highlight.

![traingin the algo 2](https://user-images.githubusercontent.com/106196315/173200102-66f75410-d097-4997-b169-ed1daf64a0cf.png)

Just like before, highlight the actual longitude, and hit extract. Just like that, all of my search reults are good. Be sure to check yours for more errors. Also check to make sure there no results with nothing highlighted. Fix those the same way we fix the others.

Now that we're done, if you press run again you'll see the fields that we made, but they're empty. As more attacks come in, they data will be parsed out into our custom fields. You may even see some attacks with some of the fields filled in as these attacks were happening while we were making our custom fields.

<!---in the future if data is getting messed up, go to custom logs > custom fields and delete the data field that is getting messed up. Now just redo the extract, and you should be good--->

Now, it's time to set up our SIEM and make the map in Sentinel!

Open a new tab and go to [portal.azure.com](https://portal.azure.com). Click on your law, on the left menu click on workbooks, and add workbook. Click edit, and remove the two default widgets by hitting the three dots, select remove, and confirm.

![remove default widget](https://user-images.githubusercontent.com/106196315/173201784-73d012c7-0885-4eb2-b924-625d7bc8afbb.png)

Now we need to add a query. Click add, and select add query.

![add query](https://user-images.githubusercontent.com/106196315/173201828-40bf8cc9-079c-429b-be6c-f419b734ac38.png)

For our queary we need our custom log FAILED_RDP_WITH_GEO_CL

We want to summarize the counts of each attack with our custom fields.

We don't want our own log in failures (if any) to count so 

| where destionhost_CF != "samplehost"

Now we also need to to exlude any hits with a blank source host so

| where destionhost_CF != ""

I did another to exlcude any fields with blank label data, as that was the last custom field I made

| where label_CF != ""

Or just copy and paste the following

FAILED_RDP_WITH_GEO_CL | summarize event_count=count() by sourcehost_CF, latitude_CF, longitude_CF, country_CF, label_CF, destinationhost_CF
| where destinationhost_CF != "samplehost"
| where sourcehost_CF != ""
| where label_CF != ""

Run query. Does it look like it's pulling hits with all the data? Good. Under the visualization field change it to map.

There are two ways we can map out our attacks. By longitude and latitude, or by country. You may have better luck with one of these settings than another, so try both.

1st, I'll show you the settings for longitude and latitude.

![layout settings - longitude and latitude](https://user-images.githubusercontent.com/106196315/173202424-473a19fa-29e1-4fad-b9aa-bfead1cae6f8.png)

You also need to set your metric settings.

![Metric Settings](https://user-images.githubusercontent.com/106196315/173202547-5dbff7fe-bad7-4c92-ae4c-12912c697999.png)

Now, if you'd like to try by country.

![layout settings - country](https://user-images.githubusercontent.com/106196315/173202560-7b1aa0f2-e5cf-47b0-8093-b23aa0f754eb.png)

My map results by longitude and latitude

![map results - longitude and latitude](https://user-images.githubusercontent.com/106196315/173202577-7bf46e36-6d6c-4a06-9234-58958947aef3.png)

My map results by country

![map results - country](https://user-images.githubusercontent.com/106196315/173202612-34ebf620-fc7a-4807-9944-f1f0ee14c7fa.png)

As you can see, a little different.
