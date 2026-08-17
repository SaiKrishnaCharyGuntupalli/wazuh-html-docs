# Non-Wazuh SOC Client Onboarding & Pipeline Implementation Guide

*GitHub → Nginx → Logstash → OpenSearch (ityug-raw-logs)*

Consolidated reference covering client onboarding, secure webhook ingestion, and index lifecycle management.

Prepared for internal SOC / MSSP operations use
Version 1.0

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope and Audience](#2-scope-and-audience)
3. [Part A — SOC Client Onboarding Plan](#part-a--soc-client-onboarding-plan)
<!-- 4. [Part B — Secure Webhook Ingestion: Nginx Reverse Proxy (Replacing ngrok)](#part-b--secure-webhook-ingestion-nginx-reverse-proxy-replacing-ngrok)
5. [Part C — OpenSearch Index Lifecycle: ISM Rollover](#part-c--opensearch-index-lifecycle-ism-rollover)
6. [Part D — Consolidated End-to-End Architecture](#part-d--consolidated-end-to-end-architecture)
7. [Appendix](#appendix) -->

---

## 1. Executive Summary

This document consolidates three working artifacts produced during the setup of a non-Wazuh SOC client pipeline into a single, structured reference: (1) the client onboarding plan, (2) the Nginx reverse-proxy implementation that replaced a temporary ngrok tunnel for GitHub webhook delivery, and (3) the OpenSearch Index State Management (ISM) rollover configuration that keeps the resulting index (`ityug-raw-logs`) healthy over time.

Together these three pieces describe one continuous data path: a client's source events (GitHub, GitLab, network telemetry, Kubernetes, etc.) are collected, forwarded securely over the public internet, parsed by Logstash, written into a rolling OpenSearch index, and evaluated by alert monitors — all wrapped in a repeatable onboarding process so the same pattern can be applied to future non-Wazuh clients.

The goal of this guide is to give any engineer picking up the project a single source of truth: what information must be collected from a client before work starts, how the ingress layer is built and secured, and how the storage layer is configured to roll over and retain data without manual intervention.

## 2. Scope and Audience

This guide is intended for SOC engineers, MSSP platform administrators, and DevOps/backend engineers involved in onboarding a new client that does not use a native Wazuh agent. It assumes familiarity with Linux system administration, basic networking (DNS, TLS, reverse proxies), and the Elastic/OpenSearch ecosystem (Logstash, indices, aliases, ISM).

- **Part A** defines the business and technical information that must be gathered from the client and the steps to provision their tenant.
- **Part B** is a step-by-step implementation of the secure webhook ingress (Nginx reverse proxy + Let's Encrypt TLS), replacing the ngrok tunnel used during initial testing.
- **Part C** is a step-by-step implementation of automatic index rollover using OpenSearch ISM, so a single alias always points at the current write index.
- **Part D** ties the three parts together into one end-to-end architecture diagram.
- The **Appendix** provides a glossary, a security checklist, and a consolidated command reference.

---

## Part A — SOC Client Onboarding Plan

### A.1 Purpose

Define a standardized onboarding process for clients that will be monitored without a native Wazuh agent, including how the appropriate ingestion pipeline is chosen for each client based on their environment and monitoring requirements.

> **Note:** The pilot rollout referenced in Parts B and C uses a GitHub test repository as the source. The original planning notes referenced GitLab as the "current selection" — confirm with the client which source system(s) are in scope before repeating this process for a new tenant, since the pipeline choice (Part B/C) depends on it.

### A.2 Pre-Requisites Checklist

Collect the following information from the client before provisioning begins. Items marked **TBD** are still being standardized internally and should be confirmed with the CISO / platform owner at the time of onboarding.

#### A.2.1 Environment and Data Profile

- Number of endpoints to be monitored
- Type of endpoints — e.g. GitLab, GitHub, network telemetry (NetFlow, Suricata, Zeek, etc.), Kubernetes, and other log sources
- Expected data volume (e.g. 30 GB/day or more)
- Networking requirements for log forwarding from the endpoint to the Logstash server — confirm required ports and protocols are reachable
- Subscription / licensing limits (**TBD**)

#### A.2.2 Platform and Index Configuration

- Wazuh Indexer indices to be created
- Index rollover strategy (see Part C)
- ISM policy — thresholds to be agreed with the customer
- RBAC roles — default or custom
- Dashboards required
- Wazuh / Logstash routing preference
- Monitor logic, triggers, and actions — defined per client requirement
- Retention policy — standard (CISO-defined) or custom per client
- Tenant name
- Agent ID — default sequence unless the client requires otherwise
- Region
- Compliance requirements (e.g. data residency, industry regulations)
- Backup strategy

#### A.2.3 Platform Access and Security

- Dedicated SOC platform vs. shared/multi-tenant platform
- If shared: communicate platform guidelines and limitations, including the isolation scope between tenants
- Confirm whether SSO is required for platform access
- If SSO is requested, provide RBAC role mapping details
- Access privileges for each RBAC role
- Collect user/member details required to provision platform access
- Per-user role assignment

#### A.2.4 Integrations (TBD)

- SIEM integration
- SOAR integration
- Ticketing system integration

#### A.2.5 Commercial and Operational

- Point of contact / contact details
- Agreement start date and total agreement period / tenure
- Alert reporting style (webhook)
- Report selection — on-demand or frequency-based
- Onboarding confirmation and access handover
- Maintenance and support model (**TBD**)

### A.3 Onboarding Workflow

Once the pre-requisites above have been collected, onboarding proceeds in two stages: tenant registration, followed by platform (MSSP) setup.

#### A.3.1 Stage 1 — Tenant Registration / Organization

1. Application: register the tenant with all details collected during the pre-requisites phase.
2. Enroll the new tenant using the information gathered from the client.
3. Prepare a backend engineering checklist, or raise a ticket, capturing the tenant's configuration.
4. Send the setup request / ticket to the backend engineering team.

#### A.3.2 Stage 2 — Begin Platform (MSSP) Setup

1. Template creation.
2. Tenant-specific index creation.
3. Update Filebeat/Logstash configuration to accommodate alert routing for the new tenant.
4. Deploy the endpoint/agent, or configure agentless collection where no agent is used.
5. Verify connectivity from the source to the ingestion layer.
6. Confirm agent/source health.
7. Configure RBAC.
8. Create index patterns.
9. Run platform validation checks before handover.

#### A.3.3 Reference Pipeline — GitHub Test Repository

The current pilot pipeline collects regular/default Git repository events (pushes, webhook deliveries) and converts them into alerts. It is used as the reference implementation for Parts B and C of this guide, and as the template for onboarding future clients with a similar source profile.

### A.4 Onboarding Sign-off Checklist

Before handing the tenant over to steady-state operations, confirm the following:

- [ ] All pre-requisite information in A.2 has been collected and recorded against the tenant record
- [ ] Ingestion pipeline (Part B) is deployed, TLS-secured, and verified end to end
- [ ] Index and rollover policy (Part C) are created and verified
- [ ] RBAC roles and (if applicable) SSO are configured and tested with a sample user
- [ ] Dashboards and index patterns are visible to the client's assigned users
- [ ] Retention and backup policy is documented and agreed with the client
- [ ] Point of contact, escalation path, and support/maintenance model are documented
- [ ] Client has received access credentials and onboarding confirmation

