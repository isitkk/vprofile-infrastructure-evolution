# Local Multi-Node Environment Provisioning via Vagrant

This directory contains the automated configurations required to provision a local 5-tier enterprise software sandbox environment mimicking an on-premise physical data center layout.

## 📐 Local Architecture & Network Topography
The system leverages Oracle VirtualBox managed via Vagrant to spin up localized, private host-only virtual machine nodes. To keep data flow secure, strict IP mappings and internal software packages are allocated across private host-only subnet domains (`192.168.56.0/24`).

| Node Hostname | Private IP Mapping | Role & Software Component Stack |
| :--- | :--- | :--- |
| `vprodb01` | `192.168.56.15` | **Database Tier:** Hardened MariaDB/MySQL storage engine persisting user profile data schemas. |
| `vpromc01` | `192.168.56.16` | **Caching Tier:** Memcached cluster saving heavy read loads from hitting database records repeatedly. |
| `vprormq01` | `192.168.56.17` | **Message Broker Tier:** RabbitMQ handling backend streaming tokens and asynchronous user operations. |
| `vproapp01` | `192.168.56.12` | **Application Tier:** Java Tomcat servlet container hosting the primary application logic binaries. |
| `vproweb01` | `192.168.56.11` | **Routing/Proxy Tier:** Nginx reverse proxy serving static front-end assets and balancing upstream client requests. |

## 🧪 Validation & Component Inter-connectivity
1.  **Component Resolution:** Systems communicate using localized host mappings via internal `/etc/hosts` resolution files rather than public external DNS paths.
2.  **Firewall Boundaries:** All backend nodes are configured to accept traffic exclusively from within the private subnet space, completely isolating the data, messaging, and caching tiers from direct machine access.
