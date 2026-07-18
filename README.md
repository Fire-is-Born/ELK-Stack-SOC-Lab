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


