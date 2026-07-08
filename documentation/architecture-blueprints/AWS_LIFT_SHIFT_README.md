# Phase 1.1: Enterprise AWS Lift-and-Shift Infrastructure Architecture

This document logs the strategic migration of our on-premise multi-tier software stack into an isolated, multi-node cloud compute matrix on AWS. This stage establishes foundational virtual server infrastructure, explicit firewall segmentation, and private application bindings.

## 🗺️ AWS Virtual Infrastructure Mapping

Instead of a localized hypervisor network, the 5-tier architecture is mapped to distinct, highly configured Amazon Elastic Compute Cloud (EC2) instances running enterprise Linux distributions.

| Component Service | Assigned OS Platform | Network Boundary & IAM Security Strategy |
| :--- | :--- | :--- |
| **`vprofile-db`** | RedHat Enterprise Linux (RHEL) / MariaDB | **Private Isolated Tier:** inbound access restricted exclusively to Port 3306 from the app security group. |
| **`vprofile-mc`** | RedHat Enterprise Linux (RHEL) / Memcached | **Private Caching Tier:** Inbound traffic restricted to Port 11211 solely from the app tier. |
| **`vprofile-rmq`**| RedHat Enterprise Linux (RHEL) / RabbitMQ | **Private Broker Tier:** Messaging lanes restricted to Port 5672 from the application layer. |
| **`vprofile-app`**| Ubuntu Server / Apache Tomcat | **Internal App Tier:** Application layer listening on Port 8080, accessible exclusively from the Nginx proxy layer. |
| **`vprofile-web`**| Ubuntu Server / Nginx | **Public Boundary Tier:** Exposed to public traffic via HTTP (80) / HTTPS (443). Connects upstream to the Tomcat container. |

---

## 🔒 Security Group Firewall Architecture

To prevent cross-tier network compromises, an enterprise security segmentation matrix was codified across all instances. Traffic is strictly governed using specific stateful security boundaries:

```text
[Internet Client]
       │
       ▼ (Port 80/443)
 ┌───────────┐
 │ vprof-web │  (Nginx Proxy Security Group)
 └─────┬─────┘
       │
       ▼ (Port 8080 upstream)
 ┌───────────┐
 │ vprof-app │  (Tomcat Server Security Group)
 └─────┬─────┘
       ├───► (Port 3306) ───► [ vprof-db ] (MariaDB Security Group)
       ├───► (Port 11211) ───► [ vprof-mc ] (Memcached Security Group)
       └───► (Port 5672) ───► [ vprof-rmq ](RabbitMQ Security Group)
