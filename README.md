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

The ELK Stack is deployed within a Vultr Virtual Private Cloud (VPC) and serves as the central logging and monitoring platform. Windows and Ubuntu endpoints forward telemetry through Elastic Agent, managed centrally using Fleet. A dedicated Kali Linux attack machine and Mythic C2 server are used to simulate realistic adversary activity, while alerts generated within Elastic can be tracked using osTicket as part of a basic incident response workflow. The SOC analyst connects remotely to Kibana to monitor, investigate, and respond to security events.

<img width="804" height="949" alt="SOC Architecture" src="https://github.com/user-attachments/assets/c644fd2e-a910-4057-949f-4097933a43d0" />

---

## 2. Deploying the ELK Stack

Provisioned an Ubuntu cloud instance in Vultr and secured remote access by creating a firewall group that permits SSH connections only from my public IP address. After establishing a secure connection via SSH, I installed Elasticsearch, enabled the service to start automatically at boot, and confirmed the deployment by verifying that the Elasticsearch service was running successfully.

<img width="2522" height="506" alt="image" src="https://github.com/user-attachments/assets/44478cdf-045e-48cd-ad56-eaf26c34a698" />

Installed Kibana on the Ubuntu server. After confirming that Elasticsearch was running successfully, I generated a Kibana enrolment token from the `elasticsearch/bin` directory to securely authenticate Kibana with the Elasticsearch cluster.

**Command**

```bash
./elasticsearch-create-enrollment-token --scope kibana
```

The generated token was then used during Kibana's initial setup to establish a trusted connection with the Elasticsearch deployment, allowing Kibana to communicate securely with the cluster.
<img width="2544" height="536" alt="image" src="https://github.com/user-attachments/assets/1a31ce20-911b-4fd9-bd56-b9e23d9000b7" />

Connected to Kibana using the enrolment token generated during the Elasticsearch setup. After logging in, configured the required three encryption keys in the kibana.yml configuration file via the SSH session. Restarted the Kibana service to apply the changes, then logged back in to verify the configuration. With the encryption keys in place, the Alerts dashboard and other security features became available.

<img width="2549" height="1268" alt="image" src="https://github.com/user-attachments/assets/8bffe90d-2f15-47cf-8c78-76b558740850" />

---

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
