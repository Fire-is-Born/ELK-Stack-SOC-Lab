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


### RDP Failed Avtivity

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






