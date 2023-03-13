# Azure-Cloud-Detection-lab

<h2> Overview </h2>
The lab will helps us to have SIEM experience, SIEM stands for Security Information and Event Management System. This is a tool allows security professionals to analyze log data from a variety of sources within their network such as Firewalls, IDS/IPS, Identity solutions, and much more.

This allows the security professionals to monitor, prioritize and remediate potential threats in real-time. Azure Sentinel (now referred to as Microsoft Sentinel) is a cloud-based SIEM/SOAR solution providing capabilities such as security analytics, threat intelligence, threat response, and more in a single platform.

Detection in cybersecurity is the practice of analyzing an environment for any anomalous activity that could possibly result in a network being compromised.
<br />
<h2>Description</h2>
In this lab, we will:

- Configure and Deploy Azure Resources such as Log Analytics Workspace, Virtual Machines, and Azure Sentinel

- Implement Network and Virtual Machine Security Best Practices

- Utilize Data Connectors to bring data into Sentinel for Analysis

- Understand Windows Security Event logs

- Configure Windows Security Policies

- Utilize KQL to query Logs

- Write Custom Analytic Rules to detect Microsoft Security Events

- Utilize MITRE ATT&CK to map adversary tactics, techniques, detection and mitigation procedures

<h2>Utilities Used</h2>

- <b>Microsoft Azure cloud services - Log Analytics Workspace, Virtual Machines, and Azure Sentinel </b> 
- <b>Kusto Query Language </b>
- <b> Windows Remote Desktop </b>
- <b> MITRE ATT&CK Framework </b>
<br />
<h2> MICROSOFT SENTINEL LAB NETWORK DESIGN & TOPOLOGY  </h2>
<img src="https://static.wixstatic.com/media/1f97f7_b70c18318efd4e83ae32410cf1b69fd6~mv2.png/v1/fill/w_740,h_774,al_c,q_90/1f97f7_b70c18318efd4e83ae32410cf1b69fd6~mv2.webp"/>
<br />
<h2> Part 1: Setup Lab Resources </h2>
 Step 1: Logging into Azure account
 Log into your Azure account https://portal.azure.com/ or create your free azure account with the following link https://azure.microsoft.com/en-us/free/. This process will automatically set up your account and associated Azure Subscription.
<br />

Step 2: Create a Resource Group </h2>

When working with Azure Resources, Resource groups are logical containers for all our resources. We will first create a resource group to group all the resources we will use for this lab. These resources will include a Windows 10 VM, Log Analytics Workspace, and an Azure Sentinel Resource.

Search “Resource Group” in the Azure Portal Search Bar and Follow the on-screen prompts for the basic tab.

<img src="https://i.imgur.com/JiHHTrQ.png"/>

You can skip to “review and create” after basics and click the create.

<img src="https://i.imgur.com/35UAEMy.png"/>

Step 3: Deploy a Virtual Machine </h2>

In this lab, we will be collecting our data from a Windows Virtual Machine. To deploy a Virtual Machine in Azure follow the following steps.

Click Create:

<img src="https://i.imgur.com/cJervIZ.png"/>

Use the resource group created in the first step and fill out the required field to create your virtual machine.


<img src="https://i.imgur.com/gD6wTX8.png"/>
<img src="https://i.imgur.com/eIs3dP5.png"/>

Use all the default settings on the Basics Tab and fill in the appropriate field.

*Please remember your admin username and password as this is how you will authenticate to the VM.

For this lab the default settings in Disks, Networking, Management, Advanced, and Tags are sufficient. We will make the appropriate network changes later.

Click Review + create and create your virtual machine.

<img src="https://i.imgur.com/9xY6JuY.png"/>

This screen is what should occur when the virtual machine is created.

<img src="https://i.imgur.com/WZoZlWS.png"/>

If we go to back to our resource group we created earlier we can see the Virtual Network and NSG listed as resources.

<img src="https://i.imgur.com/DowltKa.png"/>

If we select our NSG we can see the default rules.

<img src="https://i.imgur.com/9oCVnOc.png"/>


Recall that when we deployed our virtual machine we enabled this setting:

<img src="https://static.wixstatic.com/media/0f83c5_1d450380b70247ef9f2758028e778585~mv2.png/v1/fill/w_740,h_297,al_c,lg_1,q_90/0f83c5_1d450380b70247ef9f2758028e778585~mv2.webp"/>


If you look at the first rule, you will see that inbound RDP traffic is allowed from any source to any destination. RDP is necessary to access our VM. However, with this current setting, anyone who obtains our public IP (which can be possibly be obtained via network scan) can potentially connect to the VM as this is public facing. This presents a security risk as it makes the VM vulnerable to possibly a brute force or password spray attack.

In order to reduce our attack surface, we need to enable a security feature called Just in Time access. You can read more about this at the following link.

Essentially what this feature does is only provide access to our Virtual Machine when necessary via time-based restrictions as well as implements the principle of least privilege by giving the option to restrict access to certain IP’s as well as RBAC roles.

Anyone who wants access to the VM will need to request and based on their IP and assigned role they would be granted or denied access. By default when creating your Azure Account you are a Global Administrator so upon request you will be granted access to the VM. To set this up we will perform the following steps:

Search Microsoft Defender for Cloud in the search bar at the top of the Azure Portal and select the service. You will see a page similar to this. On the left pane select “environment settings”

<img src="https://i.imgur.com/mKTsfaG.png"/>

Select your Azure Subscription from the list provided. Upon selection, the following services will be seen on the screen. By default, Enhanced security is off but you will want to select the Enable All Microsoft Defender for Cloud Plans. You will be given a 30 free trial so be sure to disable when finished with the lab to avoid any cost. You can select then “Enable all option” and hit save.

<img src="https://i.imgur.com/PImwzQf.png"/>

After enabling the plan, Navigate back to the homepage for Defender for Cloud and select Workload Protections on the left pane. That will result in the following screen.

<img src="https://i.imgur.com/xIJQwHP.png"/>

In the Advanced protection section select “Just in time VM access” and select “Enable JIT on VM”. Make sure to select VM you created from earlier steps.

<img src="https://i.imgur.com/MMTcngy.png"/>

Use the default Just in Time settings and click save. RDP is listed amongst the ports. With these settings, anyone who is connecting via RDP or another management port will be constricted to a three hour time block upon approval which we will later grant.

<img src="https://i.imgur.com/215NxS5.png"/>
          
Click save. Just in Time Access should now be enabled.

Go to your VM settings and click Connect on the left pane. Select “My IP” as Source IP Request Access. Select “Request access”.
          
<img src="https://i.imgur.com/YFjkKhd.png"/>        

If we go to the networking tab for our VM we can see our rules have been updated. Now RDP traffic is allowed for a certain amount of time only from the IP of your computer. Anyone else who attempts to establish and RDP connection will be blocked via our Just in Time Access rules.

<img src="https://i.imgur.com/wUjjiSy.png"/> 

Step 4: Create Log Analytics Workspace and Deploy Sentinel </h2>

When working with Log Data in Azure we need somewhere to store/operate that data. Log Analytics workspace is used to collect and store log data from Azure Resources.

To configure a Log Analytic workspace:

Search “Microsoft Sentinel” in Search Bar in Azure Portal. This will prompt you to create a Log Analytics Workspace.

Use the same resource group used for the Azure Virtual Machine you created in the previous step when filling out the necessary fields to create your Log Analytics workspace.

<img src="https://i.imgur.com/WICFiI1.png"/>


Click “review and create” to create the Log Analytics Workspace

After creating the Log Analytics Workspace search Sentinel in the search bar.

<img src="https://i.imgur.com/fzQ1BkC.png"/>
                                         
Scroll to the bottom of the page and select Add                                    


<img src="https://i.imgur.com/LAwcFdN.png"/>
                                          
 After clicking create, you should see this screen upon deployment :
                                          
<img src="https://i.imgur.com/O4NDZOG.png"/>
                                          
   <br />
<h2> Part 2: Getting Data into Sentinel </h2>   

After the Sentinel Deployment, if we go to the incidents tab on the left we see we don’t have any incidents currently as there is no data being fed into sentinel. We are next going to utilize data connectors and create a data collection rule to bring in data from our Windows 10 VM.                                        

<img src="https://i.imgur.com/PpkC1vj.png"/>

Go to the Data connectors Tab. This is where we can select the type of data that we want to bring into our SIEM

<img src="https://i.imgur.com/kc0MxJ1.png"/>

In the Search bar type in “Windows” and you will see Windows Security Events via AMA. Select that option and click “Open Connector Page”.

<img src="https://i.imgur.com/R9GjBw1.png"/>

Click “Add Data collection rule.”

<img src="https://i.imgur.com/3HYZUhw.png"/>

Give your rule a name and connect to it your resource group we have used for all resources thus far.

<img src="https://i.imgur.com/80XUS31.png"/>

Click “Add resources”.

<img src="https://i.imgur.com/C1EDusp.png"/>

Select the Virtual Machine created in Step 2.

<img src="https://i.imgur.com/GoWFkUu.png"/>

Your Virtual Machine should now be shown.

<img src="https://i.imgur.com/zUw6Mdr.png"/>

Select “All Security Events”

<img src="https://i.imgur.com/fOI7Ap0.png"/>

The data collection rule should have a “Validation Passed” screen.

<img src="https://i.imgur.com/Lw4Cv3G.png"/>

Refresh the page until the “Connected” status is shown.

<img src="https://i.imgur.com/67uq1QC.png"/>

<br />
<h2> Part 3: Generating Security Events </h2>  

Now that our VM is connected to Sentinel and our Log Analytics Workspace we need to transport data from our Logs. To do this we need to simply need to perform some action on the Windows 10 events that will generate security alerts.

Windows keeps a record of several types of security events. These events cover several potential scenarios such as privileged use, Logon events, processes, policy changes, and much more.

We will now observe some Windows security events on our Virtual Machine.

Utilize the Azure Portal to navigate to the VM created earlier in the lab.

Click “Start” at the top page to turn on the VM if its not on already . Enable Just in time Access if necessary.

<img src="https://i.imgur.com/vZQlzNX.png"/>

Under Networking, you are given a public IP. Use an RDP on your PC Client such as Remote Desktop Connection to access your VM by entering in the public IP address.

(you might need to refresh after starting the virtual machine to have the public IP show up)

<img src="https://i.imgur.com/bWleGc2.png"/>

<img src="https://i.imgur.com/kcZEhhW.png"/>

~ From here you will be prompted to enter the username and password created when you made the VM.

~ Once you successfully authenticate to the virtual machine and are logged in, search for Event Viewer and open the program.

~ As you can see there are several types of logs Windows Collects. Application logs, Security Logs, Setup, System, and Forwarded Events.

<img src="https://static.wixstatic.com/media/0f83c5_d04e201cb6634bb087ae1e259cdaefeb~mv2.png/v1/fill/w_740,h_176,al_c,q_90/0f83c5_d04e201cb6634bb087ae1e259cdaefeb~mv2.webp"/>

~ Our focus in this lab will be on Windows Security events.

Click “Security” and observe the events.

As you can see there are several security events in event viewer. Let’s drill into one of these events.

Use the find option and search for 4624

<img src="https://static.wixstatic.com/media/0f83c5_8566c014783d44c4a1e91265094bb974~mv2.png/v1/fill/w_740,h_400,al_c,q_90/0f83c5_8566c014783d44c4a1e91265094bb974~mv2.webp"/>

When we select event 4624 we see that 4624 ID is indicative of a successful logon. We can also examine more detailed information about the logon if need be.

<img src="https://static.wixstatic.com/media/0f83c5_e942ee26d1e1428cb381586a34199987~mv2.png/v1/fill/w_740,h_166,al_c,q_90/0f83c5_e942ee26d1e1428cb381586a34199987~mv2.webp"/>

<br />
<h2> Part 4: Kusto Query Language </h2> 
The purpose of a SIEM such as Azure Sentinel would be to bring data like this into a centralized location. In an enterprise, we would want data coming from all our endpoints and virtual machines to make it easier for an analyst to get the information that is needed quickly.

Let’s go back to Azure Sentinel and pull this event from our Sentinel Logs.

In the Sentinel, Main Page click “Logs”

<img src="https://i.imgur.com/pdGSHFA.png"/>

Logs should bring up this page.

<img src="https://i.imgur.com/uwJKAWP.png"/>

In the section where it says “Type your query” we are going to use the following KQL (Kusto Query Language) Logic:

SecurityEvent

| where EventID == 4624

Every SIEM has a search language that makes it simple to extract data from Logs. In Sentinel, that language is called KQL or Kusto Query Language. While there are many different syntax rules and ways to construct queries in KQL we will be using a few basic KQL commands to extract the data and write our analytics rule later in the lab.

Let’s break down the meaning of this query

Security Event refers to the event table we are pulling the data from. All the events we observed in the event viewer are stored there.

Where command filters on a specific category. In this case, we only want events that correspond to successful logins.

When the query is run we get this result.

<img src="https://i.imgur.com/LYg6nOc.png"/>

As you can see, we have a list of all the times we have had a successful login on our VM. However, as you can see the Account Name field is empty a sentinel is not automatically putting that data into that field. We will go over how to populate that field later in the lab when we create our analytic rule.

<br />
<h2> Part 5: Writing Analytic Rule and Generating Scheduled Task </h2> 

We can have the option to be alerted to certain events by setting up analytic rules. The Analytic rule will check our VM for the activity that matches the rule logic and generate an alert any time that activity is observed. There will be so some details provided in the alert that can help an analyst start their investigation into determining whether the event in the alert is a false positive or true positive.

<img src="https://i.imgur.com/fUs1ypj.png"/>

These are a list of alerts that we can enable our SIEM to monitor that come out of the box. If you wish, you can expand some and see what they are comprised of by clicking create a rule and following the onscreen tasks to see the rule logic and as well as enabling the rule. However, the rules will only fire if the logic is met by a security event on your VM.

<h2> Scheduled Task and Persistence Techniques </h2> 
The final part of this lab is to create our own custom rule to detect potentially malicious activity on our VM. In Windows Task scheduler you have the option to create a scheduled task. A scheduled task is essentially a way to automate certain activities on your machine .

For instance, you could set up a scheduled task that opens google chrome at a certain time every day. While many times scheduled task can be a harmless event this can also be used as a persistence technique for malicious actors

According to the MITRE Attack Framework, “Adversaries may abuse task scheduling functionality to facilitate initial or recurring execution of malicious code. Utilities exist within all major operating systems to schedule programs or scripts to be executed at a specified date and time”.

In this lab, our scheduled task will not be associated with any malicious activity as we will set up a scheduled task that opens Internet Explorer at a certain time but we will create an analytic rule to monitor for that specific action so we can be alerted to the to the activity in our SIEM in order to simulate the scenario.

The Windows Security Event ID that corresponds to scheduled task creation is 4698. However, these events are not logged by default in the Windows event viewer. To enable logging for this event we need to make some changes to the Windows Security policy in our VM.

Search for “Local Security Policy” in Windows 10 VM and expand “Advanced Audit Policy Configuration”

<img src="https://static.wixstatic.com/media/0f83c5_4f71034fbfd840a8b45074613c362efb~mv2.png/v1/fill/w_740,h_556,al_c,lg_1,q_90/0f83c5_4f71034fbfd840a8b45074613c362efb~mv2.webp"/>

Expand “System Audit Policies” and “Select Object Access”. Then select the “Audit Other Object Access”

<img src="https://static.wixstatic.com/media/0f83c5_4a2880473a71434294ca7f83714bd0c2~mv2.png/v1/fill/w_740,h_546,al_c,lg_1,q_90/0f83c5_4a2880473a71434294ca7f83714bd0c2~mv2.webp"/>

Enable “Success” and “Failure”

<img src="https://static.wixstatic.com/media/0f83c5_aeee7f7dc1f94a489ef3b3f52ee6c28e~mv2.png/v1/fill/w_546,h_669,al_c,q_90/0f83c5_aeee7f7dc1f94a489ef3b3f52ee6c28e~mv2.webp"/>

Logging is now enabled for the scheduled task event.

Creating our scheduled task

To detect a scheduled task creation, we need to generate some activity in our VM.

Open Windows Task Scheduler and navigate to “Create Task” . Add a name and change the “Configure For” Operating system to Windows 10.

<img src="https://static.wixstatic.com/media/0f83c5_64d55a997bbd44d3955fc35e7b269839~mv2.png/v1/fill/w_740,h_562,al_c,q_90/0f83c5_64d55a997bbd44d3955fc35e7b269839~mv2.webp"/>

Navigate to triggers and click “new” and schedule the task for a time close to your current time. Then select “OK”

Navigate to the action tap and select start a program.

Then open program or script and select a program to run every time this task runs. I will select Internet Explorer.

<img src="https://static.wixstatic.com/media/0f83c5_edde2aec658e4d4bb32e5cbecafaadc7~mv2.png/v1/fill/w_740,h_560,al_c,q_90/0f83c5_edde2aec658e4d4bb32e5cbecafaadc7~mv2.webp"/>

After this do not worry about the conditions and settings do not change any of the settings and click “OK”.

This will create your scheduled task you can now go to event viewer and search for that event id 4698 in the security logs. You can see here there are several instances of this event.

<img src="https://static.wixstatic.com/media/0f83c5_f4f958e0c95f4c0dbb6b01d277730581~mv2.png/v1/fill/w_740,h_76,al_c,q_90/0f83c5_f4f958e0c95f4c0dbb6b01d277730581~mv2.webp"/>

4C. Writing Analytic Rule

Lastly, we need to write some KQL logic to alert us when a scheduled task is created.

Go to the sentinel Home Page and click “Analytics Rules” and click create at the top of the page and select the scheduled query option.

<img src="https://static.wixstatic.com/media/0f83c5_2740b1c2416b48cfbaf738cc3b8e7ba8~mv2.png/v1/fill/w_740,h_608,al_c,lg_1,q_90/0f83c5_2740b1c2416b48cfbaf738cc3b8e7ba8~mv2.webpwebp"/>

Here we are simply providing some information about the alert to the analyst.

Next, we will come up with the alert logic that causes are our alert to fire.

Most of the logic will be like the KQL Query that we created earlier for logon event.

Security Event

| where EventID == 4698

This query will pull instances of scheduled task creation as shown here in our logs.

<img src="https://static.wixstatic.com/media/0f83c5_574aecde80934bb6839db87f49f2c877~mv2.png/v1/fill/w_740,h_355,al_c,q_90/0f83c5_574aecde80934bb6839db87f49f2c877~mv2.webp"/>

Expand the log for the scheduled task you created in the previous step and look at the Event Data category.

You should see something like this:

<img src="https://static.wixstatic.com/media/0f83c5_4ad40dd4849d496d9d097ad76b278dc4~mv2.png/v1/fill/w_740,h_263,al_c,lg_1,q_90/0f83c5_4ad40dd4849d496d9d097ad76b278dc4~mv2.webp"/>

There is a lot of useful data in here such as the name of the scheduled task, the Task Name field, the ClientProcessID, the username of the account that created the scheduled task amongst other info.

However, if you use the project command to display these data fields as columns when you run the query, we need to add this to our logic.

SecurityEvent  

 | where EventID == 4698
 
 | parse EventData with * '<Data Name="SubjectUserName">' User '</Data>' *
 
| parse EventData with * '<Data Name="TaskName">' NameofSceuduledTask '</Data>' *

 | parse EventData with * '<Data Name="ClientProcessId">' ClientProcessID '</Data>' *
 
 | project Computer, TimeGenerated, ClientProcessID, NameofSceuduledTask, User
 
 The Parse command will allow us to extract data from the Event Data Field that we find important.

This extracted the SubjectUserName , TaskName, ClientProcessID (Computer automatically displays) .

The above logic allows me to assign those to new categories such as User, NameofScheduledTask, and ClientProcessID respectively. When we project our new fields, the output is the following:

<img src="https://static.wixstatic.com/media/0f83c5_66a8f780cd1346048d99475437e91065~mv2.png/v1/fill/w_740,h_174,al_c,lg_1,q_90/0f83c5_66a8f780cd1346048d99475437e91065~mv2.webp"/>

As you can see we’re able to generate Event Data and place it into its own category for readability.

Copy and Paste the above query into the editor under the “Set Rule Logic” tab

<img src="https://static.wixstatic.com/media/0f83c5_613f15a1e5cd42e39a94f9c1610b1728~mv2.png/v1/fill/w_740,h_319,al_c,lg_1,q_90/0f83c5_613f15a1e5cd42e39a94f9c1610b1728~mv2.webp"/>

The Alert Enrichment section is below this. Enriching an alert simply is the process of adding context to the alert to make it easier for the analyst to investigate.

As opposed to having to query this data as we did in the analytic rule Alert Enrichment allows us to put the necessary data into the alert details as Entities so the analyst can begin investigating those specific components.

Use the following settings for entity mapping.

<img src="https://static.wixstatic.com/media/0f83c5_104d0f39bcef46eda3ecbe9dd44fbdb9~mv2.png/v1/fill/w_740,h_378,al_c,lg_1,q_90/0f83c5_104d0f39bcef46eda3ecbe9dd44fbdb9~mv2.webp"/>
 
 The only other setting that need to be changed for the purposes of this lab is how often the query is run and when to look up the data. Change defaults to the following:
 
 <img src="https://static.wixstatic.com/media/0f83c5_642e067ffa8345f5b699e334159214ac~mv2.png/v1/fill/w_740,h_134,al_c,lg_1,q_90/0f83c5_642e067ffa8345f5b699e334159214ac~mv2.webp"/>
 
 Incident Settings and Automated Response are not necessary to alter for this lab so you can go ahead to “review and create” to make the Analytic Rule.
 
  <img src="https://static.wixstatic.com/media/0f83c5_c03903ef3e8941ae8087471b4a5de360~mv2.png/v1/fill/w_740,h_585,al_c,q_90/0f83c5_c03903ef3e8941ae8087471b4a5de360~mv2.webp"/>
  
  Once you create your analytic rule the final step is to create another scheduled task in your Windows VM and wait for the alert to trigger in Sentinel

*Note this might take up to 10 minutes. Refresh the Incidents page periodically until the Alert triggers. When the alert fires this is what you should see

 <img src="https://static.wixstatic.com/media/0f83c5_f4b6062c49d047ee81bcd65bd111cb13~mv2.png/v1/fill/w_740,h_397,al_c,q_90/0f83c5_f4b6062c49d047ee81bcd65bd111cb13~mv2.webp"/>
 
 On the right pane we see all the necessary information we would need to begin investigating the alert such as the host machine, user account, process ID of the task, and the name of the scheduled task.

While in this case, the scheduled task is non-malicious, in the event it was, from here an analyst would investigate the entities such as the user account, the scheduled task, host, etc. with other tools such as an EDR solution and other security tools to decide if this is a false positive or true positive

<br />
<h2> Part 6: MITRE ATT&CK </h2> 

The observed MITRE ATT&CK tactic used in this lab is [TA0003 Persistence](https://attack.mitre.org/tactics/TA0003/) which essentially allows a malicious actor to maintain a foothold in an environment.
 
 <img src=" https://static.wixstatic.com/media/1f97f7_18710847426d42cda558b5d08402d6d8~mv2.png/v1/fill/w_740,h_223,al_c,q_90/1f97f7_18710847426d42cda558b5d08402d6d8~mv2.webp"/>
 
 We can dig further into this by showing the technique of using a scheduled Task/Job.
 
  <img src="https://static.wixstatic.com/media/1f97f7_0de881b56ed946019e9bec42783b070e~mv2.png/v1/fill/w_740,h_293,al_c,q_90/1f97f7_0de881b56ed946019e9bec42783b070e~mv2.webp"/>
  
Even further, we can dig further into the specific sub technique of [T1053.005](https://attack.mitre.org/techniques/T1053/005/)

<img src="https://static.wixstatic.com/media/1f97f7_3d9aeaf05cc844eea89468d2c694237b~mv2.png/v1/fill/w_740,h_291,al_c,q_90/1f97f7_3d9aeaf05cc844eea89468d2c694237b~mv2.webp"/>

Detection

As observed. monitoring and logging of specific windows event id was used to detect this activity. However, MITRE also has more recommendations for detection.

<img src="https://static.wixstatic.com/media/1f97f7_33d9e225925041468bb0f9f1b86cb91f~mv2.png/v1/fill/w_740,h_132,al_c,q_90/1f97f7_33d9e225925041468bb0f9f1b86cb91f~mv2.webp"/>

Mitigation

[MITRE ID M1019](https://attack.mitre.org/mitigations/M1018//) – User Account Management suggests that user account privileges should be limited to only authorize admins to create scheduled tasks on remote systems

<img src="https://static.wixstatic.com/media/1f97f7_4b1a1a7397f84a039b39ba1a9ffa30e3~mv2.png/v1/fill/w_740,h_275,al_c,q_90/1f97f7_4b1a1a7397f84a039b39ba1a9ffa30e3~mv2.webp"/> 
 
 <br />
