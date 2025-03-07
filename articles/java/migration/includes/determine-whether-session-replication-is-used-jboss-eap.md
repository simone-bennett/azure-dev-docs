---
ms.author: jafreebe
ms.date: 02/10/2022
---

### Determine whether session replication is used

If your application relies on session replication, with or without Oracle Coherence*Web, you have two options:

* Refactor your application to use a database for session management.
* Refactor your application to externalize the session to Azure Redis Service. For more information, see [Azure Cache for Redis](/azure/azure-cache-for-redis/cache-overview).

For all of these options, it's a good idea to master how WebLogic does HTTP Session State Replication. For more information, see [HTTP Session State Replication](https://docs.oracle.com/en/middleware/fusion-middleware/weblogic-server/12.2.1.4/clust/failover.html#GUID-E13D8142-66BA-46A1-854F-4FC6F82992DD) in the Oracle documentation.
