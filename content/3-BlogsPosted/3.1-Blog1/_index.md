---
title: "Blog 1"
date: 2026-07-03
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---


# BUILDING A HYBRID MULTI-TENANT SAAS ARCHITECTURE FOR STATEFUL SERVICES ON AWS

Designing a multi-tenant architecture for stateful services (such as Game Servers and real-time Chat) is highly complex. Businesses are often stuck between the Isolation model (Silo – secure but expensive) and the Shared model (Pool – cost-effective but prone to performance bottlenecks). The Hybrid Architecture solution combines both models on a single AWS system to optimize both cost and performance.

Key points to grasp:

* Flexible customer tiering: Group small customers into a shared resource pool (Pool) to save costs, and separate VIP/Enterprise customers into independent partitions (Silo) to guarantee performance.
* Distributed state management: Session states are continuously synchronized, ensuring users do not lose data or experience service interruptions if a server encounters an issue.
* Intelligent Ingestion & Routing Layer: Utilize Amazon CloudFront and AWS WAF to filter malicious traffic at the edge. Then, API Gateway or ALB checks the accompanying Tenant ID and looks up configuration dynamically in Amazon DynamoDB to route accurately to the underlying resource partition.
* Compute & Stateful Storage Layer: The heart of the system is the Amazon EKS cluster, leveraging Namespaces and Node Affinities to segregate boundaries between Tenants. The system uses Amazon ElastiCache (Redis) as a high-speed cache to synchronize real-time states between Containers (Pods) with millisecond latency.
* Cost optimization and centralized operations: Maximize the elimination of idle resource costs thanks to the Pool model, protect VIP customers from the "Noisy Neighbor" effect (being impacted by overloads from other accounts), and allow DevOps to manage all Tenants flexibly on a single EKS cluster.
* Auto-scaling challenges: Because connections maintain state (Sticky Sessions), reducing the number of Pods (Scale-down) when traffic is low can easily disconnect users. Developers must implement additional logic for safe state data migration (State Migration) before terminating Pods.

The mechanism of using DynamoDB as a dynamic "Routing Table" allows upgrading customers from a standard tier to a VIP tier simply by changing the data configuration, without needing to modify code or infrastructure. Design your Routing Layer to be loosely coupled from the very beginning to freely customize the underlying infrastructure without impacting customers.

![Hybrid Multi-Tenant Architecture for Stateful Services](/images/3.1-Blog1/Picture1.png)

### REFERENCE LINKS
* [AWS Architecture Blog - Building a Hybrid Multi-Tenant Architecture for Stateful Services on AWS](https://aws.amazon.com/blogs/architecture/building-hybrid-multi-tenant-architecture-for-stateful-services-on-aws/)

### IMPLEMENTATION GUIDE AND DETAILED ANALYSIS

Based on the system architecture diagram, the process of designing and operating the routing layer along with the stateful storage layer should focus on the following core components:

1. **Building a Unified Routing Endpoint (Unified Endpoint):**
   * Use **Amazon Route 53** combined with flexible routing configurations (**Headers-based routing** or **Weighted Routing**) to ingest traffic from various Tenants (Tenant 1, Tenant 2, Tenant 3) into the system.
   * After passing through DNS Management, traffic will be directly routed to the corresponding infrastructure groups (**Infra group 1**, **Infra group 2**) located within the designated VPC partitions (Tier 1-Cell-1).

2. **Configuring Routing at the Application Load Balancer (ALB):**
   * At each Infra group, configure the ALB to analyze HTTP Headers or identification parameters to recognize the Tenant ID.
   * Based on the identification result, the ALB will direct connections of customers belonging to the shared group (Pool) into shared Containers, or route connections of VIP/Enterprise customers into isolated Pods (Silo) that have been pre-configured.

3. **Centralized Management and Monitoring with CloudWatch:**
   * All operational logs and metrics from the Infra groups must be configured to export (**Export Metrics**) and centralize at a management AWS Account (Tier 1) via **Amazon CloudWatch Dashboards**.
   * This helps DevOps monitor performance details and proactively detect resource bottlenecks caused by the "Noisy Neighbor" effect in the Pool group.

4. **Secure Connectivity and Session State Storage (Session State):**
   * Establish a secure, isolated network connection between the compute cluster (EKS Pods) and the caching layer (Cache) using **AWS PrivateLink**.
   * This connection leads directly to a dedicated Cache account (**AWS Account Tier 1 Cache**) - which operates the **Amazon ElastiCache (Redis)** cluster. Here, the session states (Sticky Sessions) of users in the game or real-time chat application will be continuously synchronized and maintained with ultra-low latency of under 10ms.