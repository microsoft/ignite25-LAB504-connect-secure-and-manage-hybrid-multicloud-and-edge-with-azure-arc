# Connect, secure and manage hybrid, multicloud and edge with Azure Arc

In this hands-on lab, you'll learn how to leverage Azure Arc and Azure management services to onboard, monitor, secure, and manage Windows virtual machines in hybrid environments.

## Getting Started

1\. Login to your Windows VM.


# Module 1 - Onboard a Windows VM

We're going to use the **Azure Arc Setup** wizard to connect this machine to Azure. A shortcut to this wizard is available on the desktop, but you can also find it by clicking the Azure Arc icon in the taskbar or by searching for "Azure Arc" in the start menu.

**Double click Azure Arc Setup** on the desktop to begin.

The setup wizard will check the system and network to ensure compatibility with Azure Arc. If all the checks pass, it will download and install the Azure Connected Machine agent. The Azure Connected Machine agent is what actually communicates between the server and Azure to enable remote management through Azure Arc.

Click **Next** to begin checking prerequisites and installing the Azure Connected Machine agent. This step may take 1-2 minutes to complete. After the agent is installed, click **Configure** to start connecting the machine to Azure.

Click **Next** to proceed.

On the *Sign in to Azure* page, leave the cloud set to **Azure Global** and click **Sign in to Azure**; you will then be directed to a sign-in webpage in Microsoft Edge. The Azure Connected Machine agent requires you to sign in with an account to discover which subscriptions and resource groups you have access to.

On the sign-in page, select the user that was used to log into Azure. 

After logging in, you'll see a message stating **Authentication complete.** Close Microsoft Edge and return to the configuration wizard.

Confirm that the wizard shows you've logged in, then click **Next**.

The *Resource Details* page asks where you'd like to represent this server in Azure. Like any other Azure resource, an Azure Arc-enabled server belongs to a specific tenant, subscription and resource group. The name of the resource defaults to the hostname of the machine.

We also need to select a region where metadata about the server will be stored. This doesn't need to match the physical location of the server.

For the tenant, subscription, and resource group, use the drop-down lists to select the available option. A resource group, <code spellcheck="false">azure-arc-rg</code>, was pre-provisioned for you. For the Azure region, choose <code spellcheck="false">West US 2</code>. Leave the network connectivity option set to <code spellcheck="false">Public endpoint</code>.

Click **Next** to apply these settings to the agent and create the resource representing this server in Azure. This step may take a minute or two to complete.

When you see the configuration successful message, the agent is fully configured and we can start to manage the server from Azure!

Click **Here** to navigate to your newly Arc-enabled machine.

To view all physical and virtual machines connected with Azure Arc in the portal, type ++Azure Arc++ in the search box **Search resource, services, and docs** and select **Machines - Azure Arc** from the list that appears.

The **Azure Arc resources**, **Machines** page shows all the physical and virtual machines connected with Azure Arc. Our newly onboarded server,**WS2022-VM**, should be the only server listed. Click its name to explore further.

Navigate to the overview page to see the list of capabilities available on the Azure Arc-enabled server. These capabilities represent some of the optional Azure services that you can enable on your servers using Azure Arc. If time permits, we will configure some of these services onto your machine.



# Module 2 - Enable monitoring with Azure Monitor Agent 

1. **Create a Log Analytics Workspace**. This is where monitoring data will be stored.
    1. Search for Log Analytics Workspaces in Azure Portal
    2. Select **+Create**
    3. Enter the Basic Information
        1. The Workspace **Name** must be: **ArcLogWorkspace**
        2. The Workpsace **Region** must be **East US**
    5. Select **Review & Create**
	6. Select **Create**

2. Navigate to your Azure **Arc-enabled Resource in Azure Portal** - and in the side menu, select **Extensions**.
    1. Select **Add +**
    2. Select the **Azure Monitor Agent for Windows (Recommended)**
    3. Select **Next**
    4. Select **Review & Create**
    5. Select **Create**

3. Create a **Data collection rule**. 
    1. In the Azure portal, search **Data Collection Rules**
    2. Select **+Create**
    3. In the Basics section, input the necessary information - and **leave data collection endpoint empty**
    4. In the Resources Section, select **Add Resources**
    5. Select your Arc-enabled Resource
    6. In the **Collect and Deliver** section, add a data source
    7. The first data source is **Performance Counters**. 
        1. Add A Destination. **The Destination Type is Azure Monitor Logs, and the Destination Details is your Log Analytics Workspace. **
    8. The second data source is **Windows Event Logs**. Select all Basic Application, Security, and System Logs. 
        1. Add A Destination. **The Destination Type is Azure Monitor Logs, and the Destination Details is your Log Analytics Workspace. **
    9. Create the Data Collection rule. 
    
4. Verify that traffic from the Azure Monitor Agent is reaching your Log Anlaytics Workspace. 
    1. Navigate to your Log Analytics Workspace in Azure Portal
    2. In the left pane, select **Logs**
    3. Exit out of the initial Queries Hub Pane
    4. Change the log view from Simple Mode to KQL Mode
    5. Run the following Query to show a timechart of the average CPU usage for the Arc-enabled machine, in 1-minute bins. 
    
    	If data is visible, it validates that Performance Counter data is flowing correctly. If data is not yet visible, please wait 2-3 min and try again, as  it may take a few mins for logs to be available.  
    
        _Perf 
        | where ObjectName == "Processor Information" and CounterName == "% Processor Time" 
        | summarize AvgCPU = avg(CounterValue) by bin(TimeGenerated, 1m), Computer 
        | render timechart_

    6. Run the following query to display the top Windows Event Log error messages and how often they occur across your systems. 
    
    	This provides a clear, descriptive view of the most common problems appearing in your logs. If data is not yet visible, please wait 2-3 min and try again, as  it may take a few mins for logs to be available.  
        
        If results are visible, it confirms that Windows Event Log data is flowing correctly into your workspace.
    
        _Event
        | where EventLevelName in ("Information", "Error", "Critical")
        | summarize ErrorCount = count() by RenderedDescription
        | top 10 by ErrorCount desc
        | render barchart_


# Module 3 - Use Azure Policy / Machine Configuration to Govern your Machine

Navigate to the Azure Portal Policy Page.

Azure policy provides a catalogue of built-in policies that can be applied seamlessly through the Azure portal. They also provide an extensibility framework to deploy custom policies. Feel free to browse through the available definitions.
For this lab, we will be focusing on policies that configure settings in-guest through Machine configuration.

Navigate to the Definitions blade and select the category: Guest Configuration.

From here, you can see the built-in policies that audit and configure operating system settings.

We are going to apply three policies that are both designed to improve the security posture of your Arc-enabled servers as well as your Azure VMs.

***
**Policy 1: Local Authentication Methods Should Be Disabled on Windows Servers**

This policy ensures that local authentication methods (like local user accounts or passwords stored only on the machine) are turned off. Instead, it enforces centralized identity management through Microsoft Entra or Microsoft Entra-based authentication.

Why it's useful:
* Reduces the attack surface by eliminating local credentials that can be stolen or brute-forced.
* Ensures consistent access control across your organization using centralized policies (e.g., MFA, conditional access).
* Simplifies auditing and incident response since all authentication logs come from one place.

Search for specific policy names through the search bar on the definitions page.

To assign the policy, click in the definition and navigate to the assign policy flow.

Policies are assigned to scopes including management groups, subscriptions, and resource groups. This means that not only will your current server be governed, but as you add additional Arc enabled servers to your environment, they will also be governed by these policies by default.
To assign the policy to your machine, select **your Subscription** as the scope.

Next, on Parameters tab, **change the value of the parameter "Inclue Arc connected servers" from false to true**.

After you've selected the correct scope, select review and create for the policy to take effect.



***
**Policy 2: Configure Secure Communication Protocols (TLS 1.1 or TLS 1.2) on Windows Machines**

This policy enforces the use of secure encryption protocols - typically TLS 1.2 or higher - for data transmitted between clients and servers. It disables outdated and vulnerable protocols such as SSL, TLS 1.0, and sometimes TLS 1.1.

Why it's useful:
* Protects against common attacks like man-in-the-middle (MITM) or downgrade attacks.
* Ensures encrypted communication between systems, safeguarding sensitive data in transit.
* Helps organizations comply with security standards (e.g., PCI-DSS, ISO 27001, HIPAA).

Following the same steps as the previous policy, assign the policy to the ignite scope for enforcement to begin on your server. Ensure you, **change the value of the parameter "Inclue Arc connected servers" from false to true** on the Parameters tab, and **check Create a remediation task** to ensure the policy applies to your existing Arc-enabled resource.


***

**Policy 3: Enable Microsoft Defender for Cloud on your subscription**

This policy makes sure Microsoft Defender for Cloud (formerly Azure Security Center) is turned on for your subscription. Defender for Cloud provides unified visibility and protection across Azure resources, hybrid environments, and workloads.

Why it's useful:
* Continuously monitors the security posture of your environment.
* Detects vulnerabilities, misconfigurations, and potential threats using Microsoft threat intelligence.
* Provides actionable recommendations and automated hardening guidance to improve overall cloud security posture.
* Enables advanced threat protection features for VMs, storage, databases, and more.

To enable it, begin by expanding the **Authoring** tab to access the **Assignments** page.

Click on **Assign policy**.

Click on the elipses by **Policy Definition**. Here is where you can search through the list of available policies. Search for **Enable Microsoft Defender for Cloud on your subscription** and click **Add**.

On the Remediation tab, check **Create a remediation task** to ensure the policy applies to your existing Arc-enabled resource.

Navigate to the **Review + Create** tab and hit **Create**.

You should see a notification in the top-right corner of the Azure Portal to indicate that your policy assignment has succeeded.

Refresh the Policy Assignments page. You should see the Microsoft Defender for Cloud policy is now in the list. Click on the assignment name to assess the compliance. Note: The compliance state will likely be **Not Started** at first. Wait a few minutes for the policy assessment to complete.

*Note, if you receive errors arelated to Role Assignment Creation failing, don't worry - the policy should still work. 
*


# Module 4 - Use Microsoft Defender for Cloud to Manage your VM Security

Navigate to the Microsoft Defender for Cloud page in the Azure portal.

Click on the **Inventory** tab to see the machines in your subscription that have Microsoft Defender for Cloud enabled. Explore the **Recommendations** tab to view any security recommendations that Defender has for your resources. Note, it may take up to 20 minutes for your Arc-enabled Resource to show up in yout Microsoft Defender for Cloud Inventory.



# Module 5 - Enable Automatic Agent Upgrade  

Starting with version 1.48 of the Azure Connected Machine agent, you can configure the agent to automatically upgrade itself to the latest version. This feature is currently in public preview and only available in the Azure public cloud.

When enabled, the agent will upgrade within one version of the latest release, using a batched rollout to maintain stability across regions.

1. Navigate to the Arc Machine in Azure portal

2. Select **Properties** under **Settings**
    1. In the Agent status section, you'll see fields such as: Agent version, and **Agent auto upgrade (preview)** 

3. Click **Not Enabled** and enable the capability. 
    1. You might need to refresh the resource page to see the change.

Now your Arc agent will always be up to date!


