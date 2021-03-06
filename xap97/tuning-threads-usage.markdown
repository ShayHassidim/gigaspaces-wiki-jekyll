---
layout: post
title:  Tuning Threads Usage
categories: XAP97
parent: performance-tuning-and-considerations.html
weight: 500
---

# Overview

GigaSpaces using Thread resources in an extensive manner to scale the different activities executed within the JVM process. The may happen at the client or server side. Some of the modules within the server or client side (proxy) require some tuning before moving into production. This section describes the important Thread (Thread pools) started and provides information when and how these should be tuned.

# Threads List

- # - Indicates the thread name includes a running number.
- SPACE_NAME - Indicates the thread name includes the associated space name using this thread.

{% tip %}
Starting with XAP 8.0, all GigaSpaces threads running within the JVM, using the same prefix, **GS-** as part of their name.
{% endtip %}

{: .table .table-bordered}
| Thread Name | Description | Configuration Parameters | Client/Server|
|:------------|:------------|:-------------------------|:-------------|
|LRMI Connection-pool#| See the [Communication Protocol](./communication-protocol.html) for details. |com.gs.transport\_protocol.lrmi.min-threads, com.gs.transport\_protocol.lrmi.max-threads| Server|
|LRMI Selector Accept |Responsible for accepting incoming socket connections. |Single Thread|Server|
|LRMI Selector Read and Write#| See the [Communication Protocol](./communication-protocol.html) for details. |com.gs.transport_protocol.lrmi.selector.threads |Server|
|LRMI async Selector#|Client side, handles async invocation, i.e executors, asyncRead/take. |4 Threads|Server|
|Pending Answers-pool#| Responsible for sending a callback to template based waiting  operations (read/take). |space-config.engine.min\_threads, space-config.engine.max\_threads|Server|
|Notifier-pool#| Responsible to dispath notification to client side. Used with the [Notify Container](./notify-container.html) and [Session Based Messaging API](./session-based-messaging-api.html). See the [Scaling Notification Delivery](./notify-container.html#Scaling Notification Delivery) for details.| space-config.engine.notify\_min\_threads , space-config.engine.notify\_max\_threads| Server|
|Processor-pool#|Pool for space operations post processing that can be done asynchronously to user operation. Transaction cleanup, notifications etc.  |space-config.engine.min\_threads , space-config.engine.max\_threads |Server|
|SG LeaseReaper  | Used by the Service Grid | Single Thread |Server|
|Template Expiration Manager Timer | The main thread of expiration manager. Wakes up on each expiration period to find the expired templates. | Single Thread per space | Server|
|TemplateExpirationManager-pool#|Responsible for sending response to waiting client when their template expires.| 16 threads max per space |Server|
|SyncReplicationChannel SPACE_NAME|Runs per sync replication channel |Dynamically adjusted|Server|
|CapabilityChannel  |  |  |Server|
|ClassLoaderCache-SelfCleaningTable |Used to clean up resources of class loaders once terminated (undeploy of processing unit) |Single Thread|Server|
|TransactionTableHolder-SelfCleaningTable | Responsible for cleaning zombie local transactions that were abandoned by the user application without committing. Single thread per GSC |Single Thread per space |Server|
|SLA Monitor Disk|Used by the Service Grid. SLA free disk space monitor|Single Thread|Server|
|Memory:writer|Used by the Service Grid. SLA Memory monitor|Single Thread|Server|
|CPU:writer|Used by the Service Grid. SLA CPU monitor |Single Thread|Server|
|pollingEventContainer#|Used with the [Polling Container](./polling-container.html) | See the [Concurrent Consumers](./polling-container.html#Concurrent Consumers)|Client|
|LeaseRenewalManager Task| Respisible to review resource lease.|One per resource|Client|
|JoinManager Task| Responsible to communicate with the lookup service|Single thread per client proxy| Client|
|Liveness-detector| See the [Proxy Connectivity](./proxy-connectivity.html) for details.|Single thread per client proxy|Client|
|Liveness-monitor| See the [Proxy Connectivity](./proxy-connectivity.html) for details.| Single thread per client proxy|Client|
|LocalTransactionManagerImpl$Reaper SPACE_NAME | A thread that reaps expired transactions entries and other objects| Single thread per space | Server|
|GSPingManager| Used by the Service Grid| |Server|
|LeaseManager$Reaper SPACE_NAME |See the [Lease Manager](./leases---automatic-expiration.html#Lease Manager) for details |Single Thread per space| Server|
|Cache PersistentGC|Responsible for backup activities (cleanup indexes)|Single Thread per space|Server|
|SPACE\_NAME BackgroundFifoThread3\_Notify=false#|Threads that use segmentation in order to handle background events like handling waiting templates in a fifo fashion | Thread pool per space|Server|
|SPACE\_NAME BackgroundFifoThread3\_Notify=true#|Threads that use segmentation in order to handle background notify events  in a fifo fashion|Thread pool per space|Server|
|Statistics-Task| | | Server|
|SPACE\_NAME Batch Notifier|Used when using batch notifications. See the [Batch Events](./notify-container.html#Batch Events) for details.|Single Thread per space|Server|
|ActiveFailureDetector| Responsible for the active election process. See the [Failure Detection](./failure-detection.html) for details.|Single Thread|Server|
|ProvisionLeaseManager|Used by the Service Grid|Single Thread|Server|
|Watchdog|See the [Communication Protocol](./communication-protocol.html) for details. |Single Thread per proxy|Client|
|Scheduled System Boot Thread|Used when the system starts|Single Thread| Server|
|SLAThresholdTaskPool#|Used by the Service Grid|Single Thread| Server|
|CapabilityChannel|Used by the Service Grid|Single Thread| Server|
|DynamicSmartStub-InvHandlerCache-SelfCleaningTable$Cleaner|Used by LRMI for backround cleanup|Single Thread|Server|
|ClassLoaderCache-SelfCleaningTable$Cleaner|Used by LRMI to cleanup unused classes|Single Thread|Server|
|ProvisionFailurePool#|Used by the Service Grid. Monitors the running services| |Server - GSM|
|ProvisionPool#|Used by the Service Grid. Monitors the running services| |Server - GSM|
|ProvisionMonitorEventPool#|Used by the Service Grid. Monitors the running services| |Server - GSM|
|Reggie Comm Task-pool#|Used by the Service Grid. Monitors the running services| |Server - GSM|
|SDM Cache Task|Used by the Service Grid.| |Server - GSM|
|unicast request|Used by the Service Grid.| |Server - GSM|
|service expire|Used by the Service Grid.| |Server - GSM|
|Webster Runner|Used by the Service Grid.| |Server - GSM|
|Event Lease Manager|Used by the Service Grid.| |Server - GSM|
|Fault Detection Handler|Used by the Service Grid.| |Server - GSM|
