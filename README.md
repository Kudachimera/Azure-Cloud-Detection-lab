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



 <br />
