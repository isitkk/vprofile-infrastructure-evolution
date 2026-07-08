# Phase 1.2: Cloud-Native Rearchitecting & PaaS Refactoring Log

This document chronicles the production optimization of the infrastructure stack. To eliminate single-points-of-failure and administrative system overhead, all stateful data layers were migrated from standalone EC2 virtual machines into fully managed AWS Platform-as-a-Service (PaaS) architectures.

## 🏗️ Managed Cloud Platform Architecture

Instead of manually maintaining OS patches, backups, and replication streams across raw EC2 instances, the target architecture leverages managed AWS ecosystems decoupled by purpose:

* **Platform-as-a-Service Runtime:** AWS Elastic Beanstalk (Tomcat Stack) managing the Application Load Balancer (ALB) and Auto Scaling Group (ASG) automatically.
* **Relational Database Tier:** Amazon RDS (MySQL Engine) configured with automated snapshots and multi-AZ capability.
* **Distributed Caching Layer:** Amazon ElastiCache (Memcached Engine) clusters handling session caching at sub-millisecond latencies.
* **Asynchronous Messaging Broker:** Amazon MQ (RabbitMQ Platform) handling decoupled transaction lines.
* **Domain Name System:** AWS Route53 routing inbound client requests to the Elastic Beanstalk endpoint.

---

## 🔒 Security Group & Subnet Isolation Matrix

To enforce strict isolation, the PaaS backend components utilize custom security group parameters nested to trust endpoints within the VPC:

1. **Amazon RDS Security Layer:** Attached to a private database subnet group. Its firewall accepts traffic exclusively on Port `3306` originating from the active Elastic Beanstalk security layer.
2. **Endpoint Decoupling:** Instead of routing traffic to static local `/etc/hosts` configurations, the Java application context files (`application.properties`) were refactored to consume dynamic AWS DNS connection strings:
   ```properties
   jdbc.url=jdbc:mysql://vprofile-rds-rearch.cqn4ao8gunha.us-east-1.rds.amazonaws.com 
   memcached.active.host=vprofile-rearch-cache.osoyyc.cfg.use1.cache.amazonaws.com
   rabbitmq.address=b-e3ac5f53-1827-4a21-a0ca-fa59a9ee4215.mq.us-east-1.on.aws
