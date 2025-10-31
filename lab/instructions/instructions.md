# Lab Instructions

# Connect, secure and manage hybrid, multicloud and edge with Azure Arc

In this lab, you'll learn how to leverage Azure Arc, Arc Gateway, and Azure Management services to secure and manage machines in your hybrid environments.

## Getting Started

1\. Login to your Windows VM\. 

# Module 1 - Create an Arc Gateway Resource

1. In the Search resources, services, and docs text box at the top, search for and select +++**Arc gateways**+++.

2. On the **Azure Arc | Azure Arc gateway** page, select **Create**.

3. Select the pre-existing subscription and resource group where you want the Arc gateway resource to be managed within Azure. 

4. For Name, input the name you'd like for the Arc gateway resource to have.

5. For Location, input the region where the Arc gateway resource should live. An Arc gateway resource can be used by any Arc-enabled Resource in the same Azure tenant, regardless of region.

6. Select **Review + create**.

7. Review your input details, and then once the final validation completes, select Create. The gateway creation process can take up to 10 minutes to complete.

# Module 2 - Onboard a Windows VM

We're going to use the **Azure Arc Setup** wizard to connect this machine to Azure. A shortcut to this wizard is available on the desktop, but you can also find it by clicking the Azure Arc icon in the taskbar or by searching for "Azure Arc" in the start menu.

1. **Double click Azure Arc Setup** on the desktop to begin. The setup wizard will check the system and network to ensure compatibility with Azure Arc. If all the checks pass, it will download and install the Azure Connected Machine agent. The Azure Connected Machine agent is what actually communicates between the server and Azure to enable remote management through Azure Arc.

2. Click **Next** to begin checking prerequisites and installing the Azure Connected Machine agent. This step may take 1-2 minutes to complete. After the agent is installed, click **Configure** to start connecting the machine to Azure.

3. Click **Next** to proceed.

4. On the *Sign in to Azure* page, leave the cloud set to **Azure Global** and click **Sign in to Azure**; you will then be directed to a sign-in webpage in Microsoft Edge. The Azure Connected Machine agent requires you to sign in with an account to discover which subscriptions and resource groups you have access to.

5. On the sign-in page, select the user that was used to log into Azure. If this user is not shown up, use the following credentials to log into your lab subscription. Note that this is not the same credential used to log into the virtual machine.

6. After logging in, you'll see a message stating **Authentication complete.** Close Microsoft Edge and return to the configuration wizard.

7. Confirm that the wizard shows you've logged in, then click **Next**. The *Resource Details* page asks where you'd like to represent this server in Azure. Like any other Azure resource, an Azure Arc-enabled server belongs to a specific tenant, subscription and resource group. The name of the resource defaults to the hostname of the machine.

8. We also need to select a region where metadata about the server will be stored. This doesn't need to match the physical location of the server.

9. For the tenant, subscription, and resource group, use the drop-down lists to select the available option. A resource group, <code spellcheck="false">azure-arc-rg</code>, was pre-provisioned for you. For the Azure region, choose <code spellcheck="false">West US 2</code>. Leave the network connectivity option set to <code spellcheck="false">Public endpoint</code>.

10. Click **Next** to apply these settings to the agent and create the resource representing this server in Azure. This step may take up to a minute to complete. When you see the configuration successful message, the agent is fully configured and we can start to manage the server from Azure!

11. Click **Here** to navigate to your newly Arc-enabled machine.

12. To view all physical and virtual machines connected with Azure Arc in the portal, type ++Azure Arc++ in the search box **Search resource, services, and docs** and select **Azure Arc** from the list that appears.

13. The **Azure Arc resources**, **Machines** page shows all the physical and virtual machines connected with Azure Arc. Our newly onboarded server,**WS2022-VM**, should be the only server listed. Click its name to explore further.

14. Navigate to the overview page to see the list of capabilities available on the Azure Arc-enabled server. These capabilities represent some of the optional Azure services that you can enable on your servers using Azure Arc. If time permits, we will configure some of these services onto your machine.

# Module 3 - Connect your Arc Enabled Machine with your Arc Gateway Resource

1. Navigate to your **Arc gateways** page in Portal 

2. Select the Arc gateway Resource you would like to associate with your Arc-enabled Server

3. Navigate to the **Associated Resources** pane for your gateway Resource

4. Select **Add**

5. Select your Arc enabled Resource.

6. Select **Select**

7. Open **Powershell** on your machine. 

8. After 2-3 minutes, run _azcmagent show_ in PowerShell. In the response, your "Gateway URL" should match the URL shown in your Arc gateway's Resource details in Azure Portal.

    When a machine is Arc-enabled without an Arc gateway Resource, it communicates directly with Azure's public endpoints to access Arc-enabled services. In restricted or secured environments, this requires allowing numerous public endpoints through enterprise proxies or firewalls - a process that is often complex and difficult to maintain.

   By connecting the machine to an Arc Gateway resource, this complexity is significantly reduced. The Arc Gateway securely proxies all communication between the machine and Azure, so only the gateway requires outbound access to Azure's public endpoints. This approach minimizes the number of endpoints that must be allowed through the network perimeter and simplifies overall connectivity management.


# Module 4 - Use Azure Policy / Machine Configuration to Govern your Machine

1. Navigate to the Azure Portal Policy Page.

Azure policy provides a catalogue of built-in policies that can be applied seamlessly through the Azure portal. They also provide an extensibility framework to deploy custom policies. Feel free to browse through the available definitions.
For this lab, we will be focusing on policies that configure settings in-guest through Machine configuration.

2. Navigate to the Definitions blade and select the category: Guest Configuration.

3. From here, you can see the built-in policies that audit and configure operating system settings.

We are going to apply three policies that are both designed to improve the security posture of your Arc-enabled servers as well as your Azure VMs.

**Policy 1: Local Authentication Methods Should Be Disabled on Windows Servers**

This policy ensures that local authentication methods (like local user accounts or passwords stored only on the machine) are turned off. Instead, it enforces centralized identity management through Azure Active Directory or Active Directory-based authentication.

Why it's useful:
* Reduces the attack surface by eliminating local credentials that can be stolen or brute-forced.
* Ensures consistent access control across your organization using centralized policies (e.g., MFA, conditional access).
* Simplifies auditing and incident response since all authentication logs come from one place.

1. Search for specific policy names through the search bar on the definitions page.
2. To assign the policy, click in the definition and navigate to the assign policy flow. Policies are assigned to scopes including management groups, subscriptions, and resource groups. This means that not only will your current server be governed, but as you add additional Arc enabled servers to your environment, they will also be governed by these policies by default.
3. To assign the policy to your machine, select the "Ignite..." scope.
4. After you've selected the correct scope, select review and create for the policy to take effect.

**Policy 2: Configure Secure Communication Protocols (TLS 1.1 or TLS 1.2) on Windows Machines**

This policy enforces the use of secure encryption protocols - typically TLS 1.2 or higher - for data transmitted between clients and servers. It disables outdated and vulnerable protocols such as SSL, TLS 1.0, and sometimes TLS 1.1.

Why it's useful:
* Protects against common attacks like man-in-the-middle (MITM) or downgrade attacks.
* Ensures encrypted communication between systems, safeguarding sensitive data in transit.
* Helps organizations comply with security standards (e.g., PCI-DSS, ISO 27001, HIPAA).

1. Following the same steps as the previous policy, assign the policy to the ignite scope for enforcement to begin on your server.

**Policy 3: Enable Microsoft Defender for Cloud on your subscription**

This policy makes sure Microsoft Defender for Cloud (formerly Azure Security Center) is turned on for your subscription. Defender for Cloud provides unified visibility and protection across Azure resources, hybrid environments, and workloads.

Why it's useful:
* Continuously monitors the security posture of your environment.
* Detects vulnerabilities, misconfigurations, and potential threats using Microsoft threat intelligence.
* Provides actionable recommendations and automated hardening guidance to improve overall cloud security posture.
* Enables advanced threat protection features for VMs, storage, databases, and more.

1. To enable it, begin by expanding the **Authoring** tab to access the **Assignments** page.
2. Click on **Assign policy**.
3. Click on the elipses by **Policy Definition**. Here is where you can search through the list of available policies. Search for **Enable Microsoft Defender for Cloud on your subscription** and click **Add**.
4. Navigate to the **Review + Create** tab and hit **Create**.
5. You should see a notification in the top-right corner of the Azure Portal to indicate that your policy assignment has succeeded.
6. Refresh the Policy Assignments page. You should see the Microsoft Defender for Cloud policy is now in the list. Click on the assignment name to assess the compliance. Note: The compliance state will likely be **Not Started** at first. Wait a few minutes for the policy assessment to complete.

# Module 5 - Use Microsoft Defender for Cloud to Manage your VM Security

1. If time permits, navigate to the Microsoft Defender for Cloud page in the Azure portal.
2. Click on the **Inventory** tab to see the machines in your subscription that have Microsoft Defender for Cloud enabled. Explore the **Recommendations** tab to view any security recommendations that Defender has for your resources.
