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

The ELK Stack is deployed within a Vultr Virtual Private Cloud (VPC) and serves as the central logging and monitoring platform. Windows and Ubuntu endpoints forward telemetry using Elastic Agent, managed centrally through Fleet. A dedicated Kali Linux attack machine and Mythic C2 server are used to simulate realistic adversary activity, while alerts generated within Elastic are tracked through osTicket as part of a basic incident response workflow. The SOC analyst connects remotely to Kibana to monitor, investigate, and respond to security events.

<img width="804" height="949" alt="SOC Architecture" src="https://github.com/user-attachments/assets/c644fd2e-a910-4057-949f-4097933a43d0" />

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

## 2.4 Verification

Verified that Elasticsearch and Kibana were communicating successfully by accessing the Kibana web interface and confirming that the deployment completed without errors. At this stage, the ELK Stack was fully operational and ready for Fleet Server deployment and endpoint onboarding.

## 3. Configuring Fleet and Elastic Agents

*To be completed...*

---

## 4. Onboarding Windows and Linux Endpoints

*To be completed...*

---

## 5. Deploying Mythic C2

*To be completed...*

---

## 6. Simulating Attacker Activity

*To be completed...*

---

## 7. Creating Detection Rules and Dashboards

*To be completed...*

---

## 8. Investigating Security Alerts

*To be completed...*

---

## 9. Lessons Learned

*To be completed...*
