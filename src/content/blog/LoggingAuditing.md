---
title: 'Trino Troubleshooting: Logging and Auditing '
description: 'In this article, I describe how Logging and Auditing work in Starburst'
pubDate: 'August 1, 2025'
category: 'DevOps and Cloud Infrastructure'
heroImage: '../../assets/images/2 - FA6Yb.jpg'
tags: ['ML']
---

How does Logging and auditing work in Starburst?

Logging and auditioning provided visibility into changes, data access, and API interactions for security, compliance, troubleshooting and operational insights. 


Starburst Core Log mechanisms Background

We’re going start by detailing the different types of logging available within the Starburst Trino Backend service database. Change logs track config changes, schema modifications and internal admin actions. Change logs are typical stored in the biac_change_log table in the backend service database. In juxtaposition to this, access logs are designed to capture records of user and client interactions with the Starburst coordinator. These logs are stored in the bias_access_log table in the backend service database. Access logs capture client requests like client submissions of jobs to the coordinator and requests originating from the Trino Python client or PyStarburst.

Best Practices 

The recommended data collection method is fetching protocol. This means that audit logs of REST API call history should be fetched using the methods detailed in the official Starburst documentation. 

Why is Separate Storage is critical

The reason for why separate storage is crucial is that a single query execution in Trino can generate hundreds of individual API calls. There are severe performance implications of writing such a high volume of ops to the primary backend db. This would result in a significant slow down to other critical Starburst insight-related db operations that rely on that backend db being performant. So separating the storage ensures that the audit trail doesn’t allow for degrading the performance of ops analytics and troubleshooting that depends on the core backend DB


