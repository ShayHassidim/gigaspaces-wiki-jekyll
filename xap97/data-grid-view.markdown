---
layout: post
title:  Data Grid View
categories: XAP97
parent: web-management-console.html
weight: 500
---

{% compositionsetup %}{% summary %}Cluster wide information about your data grids including: data queries, metrics, metadata etc.{% endsummary %}

# Navigation

The Data Grid view provides Space and Space instance navigation, for querying data and viewing class metadata.
Select a "Space" or press the arrow to drill down into the Space instances of each Space (cluster).

![DataGrid_SpaceInstanceSelection.png](/attachment_files/DataGrid_SpaceInstanceSelection.png)

Navigate back to show all the Spaces, by clicking on the breadcrumb on the left.

![DataGrid_SpaceSelection.png](/attachment_files/DataGrid_SpaceSelection.png)

# Query Editor

The Query editor supports SQL queries. For example, to query a specific class:

{% highlight java %}
SELECT * FROM my.company.com.MyPojo WHERE rownum < 1000
{% endhighlight %}

In the screenshot below, we also provide the UID column of each object in the Space.

{% highlight java %}
SELECT uid,* FROM com.gigaspaces.sba.trading.model.TradePojo WHERE rownum < 7
{% endhighlight %}

Press the **"Play"** icon to execute the query. Query is executed against the selected Space (cluster) or Space instance.
If the query has too many results, use the paging at the bottom to move between them. Paging is static, meaning that these results are fetched once per execute request.
Use the arrows - **back** and **forward** to navigate between previously executed queries.

![query_9_6.png](/attachment_files/query_9_6.png)

# Space Types

The metadata of the Types in the Space are shown by clicking on the "Types" tab. This lists all the types registered with the Space.
Displayed for each type are: **instance count**, **notify template count**, **Space key** (index), **Space routing key**, and **indexed fields**.

![data_types_9_6.png](/attachment_files/data_types_9_6.png)

# Object inspection

Double click on a single result set in the query results table, to show the metadata and values of each result.
Object inspection shows the **field name**, **field type**, and **field value**. For compound fields, drill down using the arrow toggles.
For array types, the array length and toString is displayed.

![object_inspector_from_query_results_9_6.png](/attachment_files/object_inspector_from_query_results_9_6.png)

# Space Statistics

The Statistics view provides graphical representation of space/space instance operations performed and average throughput.

![space_statistics_9_6.jpg](/attachment_files/space_statistics_9_6.jpg)

# Gateways

The Gateways view provides information about replication over the WAN between the selected space and target spaces.
![gateways_all_9_6.jpg](/attachment_files/gateways_all_9_6.jpg)

![gateways_outbound_9_6.jpg](/attachment_files/gateways_outbound_9_6.jpg)

![gateways_inbound_9_6.jpg](/attachment_files/gateways_inbound_9_6.jpg)

![gateways_graphical_9_6.jpg](/attachment_files/gateways_graphical_9_6.jpg)

# Local Views

The Local Views panel provides information about client side views (Local Views) in sync with the selected space

![local_views1_9_6.jpg](/attachment_files/local_views1_9_6.jpg)

![local_views2_9_6.jpg](/attachment_files/local_views2_9_6.jpg)

![local_views3.jpg](/attachment_files/local_views3.jpg)

# Local Caches

![local_caches_9_6.jpg](/attachment_files/local_caches_9_6.jpg)

# Event Containers

![polling_event_containers.jpg](/attachment_files/polling_event_containers.jpg)

![async_polling_event_containers.jpg](/attachment_files/async_polling_event_containers.jpg)

![notify_event_containers.jpg](/attachment_files/notify_event_containers.jpg)

![event_container_templates.jpg](/attachment_files/event_container_templates.jpg)
