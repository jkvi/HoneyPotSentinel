# Home SOC Project
This project provides a comprehensive guide to setting up a basic SOC in Azure, starting from scratch using a free Azure subscription. I went through the process of creating a virtual machine (VM) and turning it into a honeypot, intentionally opening it to the internet to capture real-world attack attempts. The security logs generated by the honeypot are then forwarded to a central repository for analysis. Finally, I integrated Microsoft Sentinel, Azure's powerful security information and event management (SIEM) service, to analyze the attack data, identify potential threats, and gain valuable experience in threat detection.

**Skills Learned**:

- **Cloud Computing Fundamentals**:
_Understanding of cloud infrastructure concepts (VMs, networking, storage).
Familiarity with the Azure platform and portal.
Resource management (creating, configuring, and managing Azure resources)._
- **Security Fundamentals**:
_Honeypot concepts and implementation.
Understanding of network security groups (NSGs) and their role in controlling network traffic.
Basic understanding of event logging and security event analysis.
Security event correlation and analysis._
Introduction to SIEM (Security Information and Event Management) systems.
- **Log Analysis and SIEM**:
_Log collection and forwarding.
Working with Log Analytics Workspaces (LAW).
Introduction to Azure Sentinel.
Data enrichment techniques (e.g., using watchlists for geolocation).
KQL (Kusto Query Language) proficiency for querying and analyzing security logs.
Threat visualization using workbooks and attack maps._
- **Practical Cybersecurity**:
_Hands-on experience with setting up a basic security monitoring solution.
Understanding of how to detect and respond to potential attacks.
Gaining experience with security tools and technologies used in real-world environments._
- **Troubleshooting**:
_Identifying and resolving issues related to VM configuration, network connectivity, and log forwarding.
Debugging KQL queries._

**Tools Used**:

* **Azure Portal**: _The web-based interface for managing Azure resources._
* **Azure Virtual Machines**: _For creating and managing virtual machines._
* **Network Security Groups (NSGs)**: _For controlling network traffic to and from the VM._
* **Windows Event Viewer**: _For viewing and analyzing Windows event logs._
* **Log Analytics Workspace (LAW)**: _For storing and querying log data._
* **Azure Sentinel**: _The cloud-native SIEM solution._
* **KQL (Kusto Query Language)**: _The query language used in Azure Log Analytics and Sentinel._
* **CSV Editor (e.g., Excel, Notepad++)**: _For viewing and editing the geoip-summarized.csv file._
* **Any text editor**: _for editing JSON files, and KQL queries._



**Part 1: Azure Subscription Setup**
![AzureLOGIN](https://github.com/user-attachments/assets/af2a39a4-b763-4f4f-9651-51c3462b088f)
[Image: Azure portal login screen]


Create a Free or Paid Azure Subscription: Choose a free account (if available) or a paid subscription, keeping cost-effectiveness in mind.
Azure Portal Login: Access the Azure portal at https://portal.azure.com.

**Part 2: Creating the Honeypot VM**
![VM_Creation_Screen](https://github.com/user-attachments/assets/db04e549-63a7-43ff-9e63-c5d24a4e1b35)
[Image: Azure Virtual Machines creation screen]

Create a Windows 10 VM: In the Azure portal, search for and create a new Windows 10 virtual machine. Choose an appropriate VM size, considering cost implications. Note the username and password.
Configure Network Security Group: Allow all inbound traffic in the VM's Network Security Group (NSG) for this educational honeypot. This is for testing purposes only and should not be done in production environments.
Disable Windows Firewall: Log into the VM and disable the Windows Firewall for testing purposes only. ![WINDOWS_FIREWALL_DISABLED](https://github.com/user-attachments/assets/726497d9-ef52-44e7-955f-c4f750da8e88)
[Image: Windows Firewall settings showing it's disabled]

**Part 3: Simulating Attacks and Inspecting Logs**
Simulate Login Attempts: Attempt four failed logins using a test username (e.g., "employee").
Inspect Security Logs: Open Event Viewer on the VM and examine the security logs for the failed login attempts (Event ID 4625). Event ID 4625 specifically represents failed logon attempts in Windows security logs. ![EventViewer_FailedLogin](https://github.com/user-attachments/assets/4254585a-cdba-4921-b1fc-266f92884fdd)
[Image: Screenshot of Event Viewer showing Event ID 4625 entries]

**Part 4: Log Forwarding and KQL**
![LAW-Logs1](https://github.com/user-attachments/assets/9a72f895-0a2c-484c-9467-dd4288a793f5)
[Image: Diagram illustrating Log Analytics Workspace]
![image](https://github.com/user-attachments/assets/fceb713f-c359-413f-9185-3043366d0504)
[Image: Diagram illustrating Microsoft Sentinel connected to the Log Analytics Workspace]
![DCR_Sentinel](https://github.com/user-attachments/assets/8d35534e-27be-4ba7-9144-432e564efe71)
[Image: Diagram illustrating Data Connector in Windows Security Events via AMA]

Create Log Analytics Workspace: Create a Log Analytics Workspace to centralize your logs.
Create and Connect Sentinel: Create a Microsoft Sentinel instance and connect it to the Log Analytics Workspace.
Configure Data Connector: Configure the "Windows Security Events via AMA" connector in Sentinel to forward logs from the VM.
Query Logs: Use KQL queries (e.g., SecurityEvent | where EventId == 4625). This KQL query filters the SecurityEvent table to include only events where the EventId is 4625 (failed logon attempts).![LAW-LogsWHERE-PROJECT](https://github.com/user-attachments/assets/747b76a6-bc80-4576-bc10-6fa25b7e23b5)
 [Image: Example KQL query in Sentinel]

**Part 5: Log Enrichment and Geolocation**
![image](https://github.com/user-attachments/assets/86d1dcb2-763c-414b-a13c-8dc7655180d0)
[Image: Screenshot of the geoip-summarized.csv file]

Import GeoIP Data: Import the geoip-summarized.csv https://drive.google.com/file/d/13EfjM_4BohrmaxqXZLB5VUBIz2sv9Siz/view file as a Sentinel Watchlist named "geoip" to add geolocation data to your logs.
Enriched KQL Query: Use KQL with ipv4_lookup to enhance your query with geolocation information from the watchlist. The ipv4_lookup function joins the SecurityEvent logs with the geoip watchlist based on IP addresses, adding geolocation data to each log entry. 
- let GeoIPDB_FULL = _GetWatchlist("geoip");
- let WindowsEvents = SecurityEvent
-    | where IpAddress == <attacker IP address>
-    | where EventID == 4625
-    | order by TimeGenerated desc
-    | evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network);
- WindowsEvents


**Part 6: Creating the Attack Map**
![MAPSOC](https://github.com/user-attachments/assets/e983ee1a-f4c4-4b0d-ba4f-a14fc8f60e39)
[Image: Screenshot of the Sentinel workbook with the attack map]

Create Sentinel Workbook: Create a new workbook in Sentinel.
Add Query and Map: Add a "Query" element and paste the provided JSON https://drive.google.com/file/d/1ErlVEK5cQjpGyOcu4T02xYy7F31dWuir/view?usp=drive_link for the attack map. Configure the map settings to display attack locations.
Observe the Attack Map: Visualize the attack locations on the map.

**Results and Observations:**
The project successfully demonstrated the ability to monitor a honey pot VM in Azure, collect and analyze security logs, and enrich the logs with geographic location data. The KQL queries provided valuable insights into the simulated attacks, including the attacker's IP address and geographic location. This showcases the effectiveness of using Azure Sentinel for security monitoring and incident response.

**Future Enhancements:**
- Implement automated threat detection using Azure Sentinel's analytics capabilities.
- Integrate additional security tools and data sources for more comprehensive monitoring.
- Develop more sophisticated KQL queries to detect and analyze various types of cyberattacks.
- This project provides a practical example of how to build and analyze a honey pot environment within Azure, highlighting the importance of effective log management and security information and event management (SIEM) systems.
