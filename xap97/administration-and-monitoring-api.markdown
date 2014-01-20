---
layout: post
title:  Administration and Monitoring
categories: XAP97
parent: programmers-guide.html
weight: 1700
---


{% summary%}Simple API to monitor and administer GigaSpaces services and components.{% endsummary %}

# Overview

{%section%}
{%column width=50% %}
The Admin API provides a way to administer and monitor all of GigaSpaces services and components using a simple API. The API provides information and the ability to operate on the currently running [GigaSpaces Agent](./service-grid.html#gsa), [GigaSpaces Manager](./service-grid.html#gsm), [GigaSpaces Container](./service-grid.html#gsc), [Lookup Service](./service-grid.html#lus), [Processing Unit](./packaging-and-deployment.html) and Spaces.
{%endcolumn%}
{%column width=45% %}
![archi_manag.jpg](/attachment_files/archi_manag.jpg)
{%endcolumn%}
{%endsection%}

<iframe width="640" height="360" src="//www.youtube.com/embed/j3FfHBqtj1o?feature=player_embedded" frameborder="0" allowfullscreen></iframe>

{% tip %}
You can use the [GigaSpaces Universal Deployer](/sbp/universal-deployer.html) to deploy complex multi-processing unit applications. Its is also a good example how to use the Admin API.
{% endtip %}

Before diving into the Admin API, here are some code examples showing how the Admin API can be used to display information on the of currently deployed services / components:

{% inittab admin_test|top %}
{% tabcontent GSA %}

{% highlight java %}

Admin admin = new AdminFactory().addGroup("myGroup").createAdmin();
// wait till things get discovered (you can also use specific waitFor)
for (GridServiceAgent gsa : admin.getGridServiceAgents()) {
    System.out.println("GSA [" + gsa.getUid() + "] running on Machine [" + gsa.getMachine().getHostAddress());
    for (AgentProcessDetails processDetails : gsa.getProcessesDetails()) {
        System.out.println("   -> Process [" + Arrays.toString(processDetails.getCommand()) + "]");
    }
}
{% endhighlight %}

{% endtabcontent %}
{% tabcontent GSM %}

{% highlight java %}

Admin admin = new AdminFactory().addGroup("myGroup").createAdmin();
// wait till things get discovered (you can also use specific waitFor)
for (GridServiceManager gsm : admin.getGridServiceManagers()) {
    System.out.println("GSM [" + gsm.getUid() + "] running on Machine " + gsm.getMachine().getHostAddress());
}
{% endhighlight %}

{% endtabcontent %}
{% tabcontent GSC %}

{% highlight java %}

Admin admin = new AdminFactory().addGroup("myGroup").createAdmin();
// wait till things get discovered (you can also use specific waitFor)
for (GridServiceContainer gsc : admin.getGridServiceContainers()) {
    System.out.println("GSC [" + gsc.getUid() + "] running on Machine " + gsc.getMachine().getHostAddress());
    for (ProcessingUnitInstance puInstance : gsc.getProcessingUnitInstances()) {
        System.out.println("   -> PU [" + puInstance.getName() + "][" +
        puInstance.getInstanceId() + "][" + puInstance.getBackupId() + "]");
    }
}
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Processing Unit %}

{% highlight java %}
Admin admin = new AdminFactory().addGroup("myGroup").createAdmin();
// wait till things get discovered (you can also use specific waitFor)
for (ProcessingUnit processingUnit : admin.getProcessingUnits()) {
    System.out.println("Processing Unit: " + processingUnit.getName() + " status: " + processingUnit.getStatus());
    if (processingUnit.isManaged()) {
        System.out.println("   -> Managing GSM: " + processingUnit.getManagingGridServiceManager().getUid());
    } else {
        System.out.println("   -> Managing GSM: NA");
    }
    for (GridServiceManager backupGSM : processingUnit.getBackupGridServiceManagers()) {
        System.out.println("   -> Backup GSM: " + backupGSM.getUid());
    }
    for (ProcessingUnitInstance processingUnitInstance : processingUnit) {
        System.out.println("   [" + processingUnitInstance.getClusterInfo() + "] on GSC [" +
         processingUnitInstance.getGridServiceContainer().getUid() + "]");
        if (processingUnitInstance.isEmbeddedSpaces()) {
            System.out.println("      -> Embedded Space [" + processingUnitInstance.getSpaceInstance().getUid() + "]");
        }
        for (ServiceDetails details : processingUnitInstance) {
            System.out.println("      -> Service " + details);
        }
    }
}
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Space %}

{% highlight java %}

for (Space space : admin.getSpaces()) {
    System.out.println("Space [" + space.getUid() + "] numberOfInstances [" +
     space.getNumberOfInstances() + "] numberOfbackups [" +
     space.getNumberOfBackups() + "]");
    System.out.println("  Stats: Write [" + space.getStatistics().getWriteCount() + "/" +
    space.getStatistics().getWritePerSecond() + "]");
    for (SpaceInstance spaceInstance : space) {
        System.out.println("   -> INSTANCE [" + spaceInstance.getUid() + "] instanceId [" + spaceInstance.getInstanceId() +
        "] backupId [" + spaceInstance.getBackupId() + "] Mode [" + spaceInstance.getMode() + "]");
        System.out.println("         -> Host: " + spaceInstance.getMachine().getHostAddress());
        System.out.println("         -> Stats: Write [" + spaceInstance.getStatistics().getWriteCount() + "/" +
        spaceInstance.getStatistics().getWritePerSecond() + "]");
    }
    for (SpacePartition spacePartition : space.getPartitions()) {
        System.out.println("   -> Partition [" + spacePartition.getPartitiondId() + "]");
        for (SpaceInstance spaceInstance : spacePartition) {
            System.out.println("      -> INSTANCE [" + spaceInstance.getUid() + "]");
        }
    }
}
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Virtual Machine %}

{% highlight java %}

Admin admin = new AdminFactory().addGroup("myGroup").createAdmin();
// wait till things get discovered (you can also use specific waitFor)
System.out.println("VM TOTAL STATS: Heap Committed [" +
	admin.getVirtualMachines().getStatistics().getMemoryHeapCommittedInGB() +"GB]");
	System.out.println("VM TOTAL STATS: GC PERC [" +
	admin.getVirtualMachines().getStatistics().getGcCollectionPerc() + "], Heap Used [" +
 	admin.getVirtualMachines().getStatistics().getMemoryHeapPerc() + "%]");
for (VirtualMachine virtualMachine : admin.getVirtualMachines()) {
    System.out.println("VM [" + virtualMachine.getUid() + "] " +
			"Host [" + virtualMachine.getMachine().getHostAddress() + "] " +
            "GC Perc [" + virtualMachine.getStatistics().getGcCollectionPerc() + "], " +
            "Heap Usage [" + virtualMachine.getStatistics().getMemoryHeapPerc() + "%]");

    for (ProcessingUnitInstance processingUnitInstance : virtualMachine.getProcessingUnitInstances()) {
        System.out.println("   -> PU [" + processingUnitInstance.getUid() + "]");
    }
    for (SpaceInstance spaceInstance : virtualMachine.getSpaceInstances()) {
        System.out.println("   -> Space [" + spaceInstance.getUid() + "]");
    }
}
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Machine %}

{% highlight java %}

Admin admin = new AdminFactory().addGroup("myGroup").createAdmin();
// wait till things get discovered (you can also use specific waitFor)
for (Machine machine : admin.getMachines()) {
    System.out.println("Machine [" + machine.getUid() + "], " +
            "TotalPhysicalMem [" + machine.getOperatingSystem().getDetails().getTotalPhysicalMemorySizeInGB() + "GB], " +
            "FreePhysicalMem [" + machine.getOperatingSystem().getStatistics().getFreePhysicalMemorySizeInGB() + "GB]]");
    for (SpaceInstance spaceInstance : machine.getSpaceInstances()) {
        System.out.println("   -> Space [" + spaceInstance.getUid() + "]");
    }
    for (ProcessingUnitInstance processingUnitInstance : machine.getProcessingUnitInstances()) {
        System.out.println("   -> PU [" + processingUnitInstance.getUid() + "]");
    }
}
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

{% tip %}
See a fully running example of a [Scaling Agent](/sbp/scaling-agent.html) as part of the Solutions & Best Practices section.
{% endtip %}

# Admin Construction

The Admin API uses the `AdminFactory` in order to create `Admin` instances. Once working with the `Admin` is done, its `Admin#close()` method should be called.

The Admin discovers all the advertised services from the [Lookup Service](./service-grid.html#lus). In order to define which lookup groups the `AdminFactory#addGroup` can be used. The lookup locators can also be used for non multicast enabled environment using `AdminFactory#addLocator` can be used. If the services started are secured, the username and password can be set on the Admin API as well.


# Discovery Process

Once the `Admin` is created, it will start to receive discovery events of all the advertised services / components within its lookup groups / lookup locators. Note, the events occur asynchronously and the data model within the Admin gets initialized in the background with services coming and going.

This means that just creating the `Admin` and calling a specific "getter" for a data structure might not return what is currently deployed, and one should wait till the structures are filled. Some components has a waitFor method that allow to wait for specific number of services to be up. When navigating the data model, the Admin API will provide its most up to date state of the system it is monitoring.

# Domain Model

The Admin Domain Model has representation to all GigaSpaces level main actors. They include:
|[GridServiceAgent](#GridServiceAgentLink) | [GridServiceAgents](#GridServiceAgentsLink) | [GridServiceManager](#GridServiceManagerLink) |
|[GridServiceManagers](#GridServiceManagersLink) | [GridServiceContainer](#GridServiceContainerLink) | [GridServiceContainers](#GridServiceContainersLink) |
|[LookupService](#LookupServiceLink) | [LookupServices](#LookupServicesLink) | [ProcessingUnit](#ProcessingUnitLink) | [ProcessingUnitInstance](#ProcessingUnitInstanceLink) | [ProcessingUnits](#ProcessingUnitsLink) |
|[Space](#SpaceLink) | [SpaceInstance](#SpaceInstanceLink) | [Spaces](#SpacesLink) | [VirtualMachine](#VirtualMachineLink) | [VirtualMachines](#VirtualMachinesLink) |
|[Machine](#MachineLink) | [Machines](#MachinesLink) | [OperatingSystem](#OperatingSystemLink) | [OperatingSystems](#OperatingSystemsLink) | [Transport](#TransportLink) | [Transports](#TransportsLink)

![admin_DomainModel.jpg](/attachment_files/admin_DomainModel.jpg)

{: .table .table-bordered}
|Name|Description|Main Operations|Runtime Events|
|:---|:----------|:--------------|:-------------|
|[{% anchor GridServiceAgentLink %}GridServiceAgent](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/gsa/GridServiceAgent.html)|A process manager that manages Service Grid processes such as GSM, GSC and LUS. More info [here](./service-grid.html#gsa).|* Allows to list all the currently managed processes.{% wbr %}- Start processes (GSM, GSC, LUS).| |
|{% anchor GridServiceAgentsLink %}[GridServiceAgents](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/gsa/GridServiceAgents.html)|Holds all the currently discovered Grid Service Agents.|* Get all the currently discovered Grid Service Agents.{% wbr %}- Wait for X number of Grid Service Agents to be up.|* Register for Grid Service Agent addition (discovery) and removals events.|
|{% anchor GridServiceManagerLink %}[GridServiceManager](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/gsm/GridServiceManager.html)|Managing Processing Unit deployments and Grid Service Containers. More info [here](./service-grid.html#gsm).|* Deploy Processing Units.{% wbr %}- Deploy pure Space Processing Units.{% wbr %}- Get the Grid Service Agent Managing it.{% wbr %}- Restart itself (if managed by a Grid Service Agent).{% wbr %}- Kill itself (if managed by a Grid Service Agent).| |
|{% anchor GridServiceManagersLink %}[GridServiceManagers](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/gsm/GridServiceManagers.html)|Holds all the currently discovered Grid Service Managers.|* Deploy Processing Units on a random Grid Service Manager.{% wbr %}- Deploy pure Space Processing Units on a random Grid Service Manager.{% wbr %}- Get all the currently discovered Grid Service Managers.{% wbr %}- Wait for X number of Grid Service Managers to be up.|* Register for Grid Service Manager addition (discovery) and removals events.|
|{% anchor GridServiceContainerLink %}[GridServiceContainer](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/gsc/GridServiceContainer.html)|Container hosting Processing Unit Instances deployed from the GSM. More info [here](./service-grid.html#gsc).|\* List currently running Processing Units Instances.|\* Register for Processing Unit Instance additions and removals events.|
|{% anchor GridServiceContainersLink %}[GridServiceContainers](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/gsc/GridServiceContainers.html)|Holds all the currently discovered Grid Service Containers.|* Get all the currently discovered Grid Service Containers.{% wbr %}- Wait for X number of Grid Service Containers to be up.|* Register for Grid Service Container addition (discovery) and removals events.|
|{% anchor LookupServiceLink %}[LookupService](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/lus/LookupService.html)|A registry of services (GSM, GSC, Space Instances, Processing Unit Instances) that can be lookup up using it. More info [here](./the-lookup-service.html#lus).|* Get the Lookup Groups and Locator it was started with.| |
|{% anchor LookupServicesLink %}[LookupServices](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/lus/LookupServices.html)|Holds all the currently discovered Lookup Services.|* Get all the currently discovered Lookup Services.{% wbr %}- Wait for X number of Lookup Services to be up.|* Register for Lookup Service addition (discovery) and removals events.|
|{% anchor ProcessingUnitLink %}[ProcessingUnit](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/pu/ProcessingUnit.html)|A deployable processing unit running one or more Processing Unit Instances. Managed by the Grid Service Manager.| * Undeploy the Processing Unit{% wbr %}- Increase the number of Processing Units Instances (if allowed).{% wbr %}- Decrease the number of Processing Unit Instances (if allowed).{% wbr %}- Get the deployment status of the Processing Unit.{% wbr %}- Get the managing Grid Service Manager.{% wbr %}- Get the list of backup Grid Service Managers.{% wbr %}- List all the currently running Processing Unit Instances.{% wbr %}- Wait for X number of Processing Unit Instances or be up.{% wbr %}- Get an embedded Space that the Processing Unit has.{% wbr %}- Wait for an embedded Space to be correlated (discovered) with the Processing Unit.|* Register for Processing Unit Instances additions and removals events.{% wbr %}- Register for Processing Unit Instance provision attempts, failures, success and pending events.{% wbr %}- Register for Managing Grid Service Manager change events.{% wbr %}- Register for Space correlation events.{% wbr %}- Register for deployment status change events.{% wbr %}- Register for backup Grid Service Manager change events.|
|{% anchor ProcessingUnitInstanceLink %}[ProcessingUnitInstance](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/pu/ProcessingUnitInstance.html)|An actual instance of a Processing Unit running within a Grid Service Container.|* Destroy itself (if SLA is breached, will be instantiated again).{% wbr %}- Decrease itself (and destroying itself in the process). Will not attempt to create it again.{% wbr %}- Relocate itself to a different Grid Service Container.{% wbr %}- List all its inner services (such as event containers).{% wbr %}- Get the embedded Space Instance running within it (if there is one).{% wbr %}- Get the JEE container details if it is a web processing unit.| |
|{% anchor ProcessingUnitsLink %}[ProcessingUnits](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/pu/ProcessingUnits.html)|Holds all the currently deployed Processing Units|* Get all the currently deployed Processing Units.{% wbr %}- Wait for (and return) a Processing by a specific name.|* Register for Processing Unit deployments and undeployment events.{% wbr %}- Register for all Processing Unit Instance addition and removal events (across all Processing Units).{% wbr %}- Register for all Processing Unit Instance provision attempts, failures, success and pending events (across all Processing Units).{% wbr %}- Register for Managing Grid Service Manager change events on all Processing Units.{% wbr %}- Register for backup Grid Service Manager change events on all Processing Units.|
|{% anchor SpaceLink %}[Space](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/space/Space.html)|Composed of one or more Space Instances to form a Space topology (cluster)|* Get all the currently running Space Instance that are part of the Space.{% wbr %}- Wait for X number of Space Instances to be up.{% wbr %}- Get aggregated Space statistics.{% wbr %}- Get a clustered [GigaSpace](./the-gigaspace-interface.html) to perform Space operations.|* Register for Space Instance additions and removals events.{% wbr %}- Register for Space Instance change mode events (for all Space Instances that are part of the Space).{% wbr %}- Register for Space Instance replication status change events (for all Space Instances that are part of the Space).{% wbr %}- Register for aggregated Space statistics events (if monitoring).|
|{% anchor SpaceInstanceLink %}[SpaceInstance](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/space/SpaceInstance.html)|An actual instance of a Space that is part of a topology (cluster), usually running within a Processing Unit Instance|* Get its Space Mode (primary or backup).{% wbr %}- Get its replication targets.{% wbr %}- Get a direct [GigaSpace](./the-gigaspace-interface.html) to perform Space operations.{% wbr %}- Get Space Instance statistics.|* Register for replication status change events.{% wbr %}- Register for Space Mode change events{% wbr %}- Register for Space Instance statistics (if monitoring).|
|{% anchor SpacesLink %}[Spaces](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/space/Spaces.html)|Holds all the currently running Spaces|* Get all the currently running Spaces.{% wbr %}- Wait for (and return) a specific Space by name.|* Register for Space additions and removal events.{% wbr %}- Register for Space Instance additions and removal events (across all Spaces).{% wbr %}- Register for Space Instance Mode change events (across all Space Instances).{% wbr %}- Register for Space Instance replication change events (across all Space Instances).{% wbr %}- Register for aggregated Space level statistics change events (across all Spaces, if monitoring).{% wbr %}- Register for Space Instance statistics change events (across all Space Instances, if monitoring).|
|{% anchor VirtualMachineLink %}[VirtualMachine](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/vm/VirtualMachine.html)|A virtual machine (JVM) that is currently running at least one GigaSpaces component / service.|* Get the Grid Service Agent (if exists).{% wbr %}- Get the Grid Service Manager (if exists).{% wbr %}- Get the Grid Service Container (if exists).{% wbr %}- Get all the Processing Unit Instances that are running within the Virtual Machine.{% wbr %}- Get all the Space Instances that are running within the Virtual Machine.{% wbr %}- Get the details of the Virtual Machine (min/max memory, and so on).{% wbr %}- Get the statistics of the Virtual Machine (heap used, and so on).|* Register for Processing Unit Instance additions and removals events.{% wbr %}- Register for Space Instance additions and removals events.{% wbr %}- Register for statistics change events (if monitoring).|
|{% anchor VirtualMachinesLink %}[VirtualMachines](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/vm/VirtualMachines.html)|Holds all the currently discovered Virtual Machines|* Get all the currently discovered Virtual Machines.{% wbr %}- Get aggregated Virtual Machines details.{% wbr %}- Get aggregated Virtual Machines statistics.|* Register for Virtual Machines additions and removals events.{% wbr %}- Register for aggregated Virtual Machines statistics events (if monitoring).{% wbr %}- Register for Virtual Machine level statistics change events (across all Virtual Machines, if monitoring).|
|{% anchor MachineLink %}[Machine](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/machine/Machine.html)|An actual Machine (identified by its host address) running one or more GigaSpaces components / services in one or more Virtual Machines. Associated with one Operating System|* Get all the Grid Service Agents running on the Machine.{% wbr %}- Get all the Grid Service Containers running on the Machine.{% wbr %}- Get all the Grid Service Managers running on the Machine.{% wbr %}- Get all the Virtual Machines running on the Machine.{% wbr %}- Get all the Processing Unit Instances running on the Machine.{% wbr %}- Get all the Space Instances running on the Machine.{% wbr %}- Get the Operating System the Machine is running on.|* Register for Space Instances additions and removals events from the Machine.{% wbr %}- Register for Processing Unit Instance additions and removals events from the Machine.|
|{% anchor MachinesLink %}[Machines](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/machine/Machines.html)|Holds all the currently discovered Machines|* Get all the currently running Machines.{% wbr %}- Wait for X number of Machines or be up.|* Register for Machine additions and removals events.|
|{% anchor OperatingSystemLink %}[OperatingSystem](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/os/OperatingSystem.html)|The Operating System GigaSpaces components / services are running on. Associated with one Machine.|* Get the details of the Operating System.{% wbr %}- Get the operating system statistics.|* Register for statistics change events (if monitoring).|
|{% anchor OperatingSystemsLink %}[OperatingSystems](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/os/OperatingSystems.html)|Holds all the currently discovered Operating Systems|* Get all the current Operating Systems.{% wbr %}- Get the aggregated Operating Systems details.{% wbr %}- Get the aggregated Operating Systems statistics.|* Register for aggregated Operating Systems statistics change events (if monitoring).{% wbr %}- Register for Operating System level statistics change events (across all Operating Systems, if monitoring).|
|{% anchor TransportLink %}[Transport](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/transport/Transport.html)|The communication layer each GigaSpaces component / service uses|* Get the Transport details (host, port).{% wbr %}- Get the Transport statistics.|* Register for Transport statistics change events (if monitoring).|
|{% anchor TransportsLink %}[Transports](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/admin/transport/Transports.html)|Holds all the currently discovered Transports|* Get all the current Transports.{% wbr %}- Get the aggregated Transports details.{% wbr %}- Get the aggregated Transports statistics.|* Register for aggregated Transports statistics change events (if monitoring).{% wbr %}- Register for Transport level statistics change events (across all Transports, if monitoring).|

# Accessing the Domain Model

There are two ways the Admin API can be used to access information the Admin API can provide.

- Call specific "getters" for the data and iterate over them (as shown in the example at the top of the page).
- Register for specific events using the Admin API. Events are handled by different components of the Admin API in similar manner. We will take one of them and use it as a reference example.

If we want to register, for example, for Grid Service Container additions, we can use the following code (note, removing the event listener is not shown here for clarity):

{% highlight java %}
admin.getGridServiceContainers().getGridServiceContainerAdded().add(new GridServiceContainerAddedEventListener() {
    public void gridServiceContainerAdded(GridServiceContainer gridServiceContainer) {
        // do something here
    }
});
{% endhighlight %}

Removals are done in similar manner:

{% highlight java %}
admin.getGridServiceContainers().getGridServiceContainerRemoved().add(new GridServiceContainerRemovedEventListener() {
    public void gridServiceContainerRemoved(GridServiceContainer gridServiceContainer) {
        // do something here
    }
});
{% endhighlight %}

Since both removals and additions are common events that we would like to register for in one go, we can use:

{% highlight java %}
admin.getGridServiceContainers().addLifecycleListener(new GridServiceContainerLifecycleEventListener() {
    public void gridServiceContainerAdded(GridServiceContainer gridServiceContainer) {
        // do something here
    }

    public void gridServiceContainerRemoved(GridServiceContainer gridServiceContainer) {
        // do something here
    }
});
{% endhighlight %}

All other data structures use similar API to register for events. Some might have specific events that goes beyond just additions and removals, but they still use the same model. For example, here is how we can register for Space Mode change events across all currently running Space topologies and Space Instances:

{% highlight java %}
admin.getSpaces().getSpaceModeChanged().add(new SpaceModeChangedEventListener() {
    public void spaceModeChanged(SpaceModeChangedEvent event) {
        System.out.println("Space [" + event.getSpaceInstance().getSpace().getName() + "] " +
		"Instance [" + event.getSpaceInstance().getInstanceId() + "/" +
                event.getSpaceInstance().getBackupId() + "] " +
		"changed mode from [" + event.getPreviousMode() + "] to [" + event.getNewMode() + "]");
    }
});
{% endhighlight %}

Of course, we can register the same listener on a specific `Space` topology or event on a specific `SpaceInstance`.

Last, the `Admin` interface provides a one stop method called `addEventListener` that accepts an `AdminListener`. Most events listener implement this interface. One can create a class that implements several chosen listener interfaces, call the `addEventListener` method, and they will automatically be added to their respective components. For example, if our listener implements `GridServiceContainerAddedEventListener` and `GridServiceManagerAddedEventListener`, the listener will automatically be added to the `GridServiceManagers` and `GridServiceContainers`.

# Details and Statistics

- Some components in the Admin API can provide statistics. For example, a `SpaceInstance` can provide statistics on how many times the read API was called on it. Statistics change over time, and in order to get them either the "getter" for the Statistics can be used, or a statistics listener can be registered for statistics change events.

- Details of a specific component provide information that does not change over time, but can be used to provide more information regarding the component, or to compute statistics. For example, the VirtualMachine provides in its details the minimum and maximum heap memory size, which the VirtualMachine statistics provide the currently used heap memory size. The detailed information is used to provide the percentage used in the Virtual Machine statistics.

- The Admin API also provide aggregated details and statistics. For example, the `Space` provides `SpaceStatistics` allowing to get the aggregated statistics of all the different Space Instances that belong to it.

- Each component in the Admin API that can provide statistics (either direct statistics, or aggregated statistics) implements the `StatisticsMonitor` interface. The statistics monitor allows to start to monitor statistics and stop to monitor statistics. Monitoring for statistics is required if one wishes to register for statistics change events. The interval that statistics will be polled is controlled using the statistics interval.

- The statistics interval is important event when the Admin API is not actively polling for statistics. Each call to a "getter" of statistics will only perform a remote call to the component if the last statistics fetch happened **before** the statistics interval. This behavior allows for users of the Admin API to not worry about "hammering" different components for statistics since the Admin will make sure that statistics calls are cached internally for the statistics interval period.

- A `SpaceInstance` implements the `StatisticsMonitor` interface. Calling `startMonitor` and `stopMonitor` on it will cause monitoring of statistics to be enabled or disabled on it.

- `Space` also implements the `StatisticsMonitor` interface. Calling `startMonitor` on it will cause it to start monitoring all its `SpaceInstance` s. If a `SpaceInstance` is discovered after the the call to `startMonitor` occurred, it will start monitoring itself automatically. This means that if the a `SpaceInstanceStatisticsChangedEventListener` was registered on the `Space`, it will automatically start to get Space Instance statistics change events for the newly discovered `SpaceInstance`.

- `Spaces` also implements the `StatisticsMonitor` interface. Calling `startMonitor` on it will cause it to start monitoring all the `Space` s it has (and as a result, also `SpaceInstance` s, see the paragraph above). A `SpaceInstanceStatisticsChangedEventListener` can also be registered on the `Spaces` level as well.

- The above Space level statistics behavior works in much the way with other components. For example, the `VirutalMachine` and `VirtualMachines`, `Transport` and `Transports`, `OperatingSystem` and `OperatingSystems`.

- The `Admin` interface also implements the `StatisticsMonitor` interface. Calling `startMonitor` on it will cause all holders to start monitoring. These include: `Spaces`, `VirtualMachines`, `Transports`, and `OperatingSystems`.

# Space Runtime Statistics

The space maintains statistics information about all the data types (e.ge. space class types) it stores and the amount of space objects stored in memory for each data type. Below example how to retrieve this data. This approach avoiding the need to use the `GigaSpace.Count()` that is relatively expensive with spaces that store large number of objects.

{% highlight java %}
public static void printAllClassInstanceCountForAllPartitions (String spaceName) throws Exception
{
	Admin admin = new AdminFactory().addLocator("127.0.0.1").createAdmin();
	Space space = admin.getSpaces().waitFor(spaceName, 10 , TimeUnit.SECONDS);
	SpacePartition spacePartitions[] = space.getPartitions();
	System.out.println(spaceName + " have " +spacePartitions.length + " Partitions , Total " + space.getTotalNumberOfInstances() + " Instances");
	Arrays.sort(spacePartitions, new SpacePartitionsComperator ());

	for (int i = 0; i < spacePartitions.length; i++) {
		SpacePartition partition = spacePartitions [i];
		while (partition.getPrimary()==null)
		{
			Thread.sleep(1000);
		}

		SpaceInstance primaryInstance = partition.getPrimary();
		System.out.println ("Partition " + partition.getPartitionId() + " Primary:");
		Map<String, Integer> classInstanceCountsPrimary = primaryInstance.getRuntimeDetails().getCountPerClassName();
		Iterator<String> keys = classInstanceCountsPrimary.keySet().iterator();
		while (keys.hasNext())
		{
			String className = keys.next();
			System.out.println ("Class:" + className +" Instance count:" + classInstanceCountsPrimary.get(className));
		}

		SpaceInstance backupInstance = partition.getBackup();
		System.out.println ("Partition " + partition.getPartitionId() + " Backup:");

		Map<String, Integer> classInstanceCountsBackup = backupInstance .getRuntimeDetails().getCountPerClassName();
		keys = classInstanceCountsPrimary.keySet().iterator();
		while (keys.hasNext())
		{
			String className = keys.next();
			System.out.println ("Class:" + className +" Instance count:" + classInstanceCountsBackup.get(className));
		}

		admin.close();
	}
}

static class SpacePartitionsComperator implements Comparator<SpacePartition>{
	public int compare(SpacePartition o1,SpacePartition o2) {
		if (o2.getPartitionId() > o1.getPartitionId())
			return 0;
		else
			return 1;
	}
}
{% endhighlight %}

# Monitoring the Mirror Service

You are now able to monitor various aspects of the mirror service using the administration and monitoring API.
The mirror statistics are available using the `SpaceInstance` statistics. They can be used to monitor the state of the mirror space and whether or not it is functioning properly. These statistics are relevant only for a mirror space instance, and are not available for ordinary space instances. The code below traverses all the space instances and finds the mirror space by retreiving the mirror statistics object (if it isn't null this means it's a mirror space). It then prints out some of the available statistics.

{% highlight java %}
for (Space space : admin.getSpaces()) {
    System.out.println("Space [" + space.getUid() + "] numberOfInstances [" +
     space.getNumberOfInstances() + "] numberOfbackups [" +
     space.getNumberOfBackups() + "]");

    for (SpaceInstance spaceInstance : space) {
        System.out.println("   -> INSTANCE [" + spaceInstance.getUid() + "] instanceId [" + spaceInstance.getInstanceId() +
        "] backupId [" + spaceInstance.getBackupId() + "] Mode [" + spaceInstance.getMode() + "]");
        System.out.println("         -> Host: " + spaceInstance.getMachine().getHostAddress());

         MirrorStatistics mirrorStat = spaceInstance.getStatistics().getMirrorStatistics();
         // check if this instance is mirror
         if(mirrorStat != null)
         {

            System.out.println("Mirror Stats:");
            System.out.println("total operation count:" + mirrorStat.getOperationCount());
            System.out.println("successful operation count:" + mirrorStat.getSuccessfulOperationCount());
            System.out.println("failed operation count:" + mirrorStat.getFailedOperationCount());
         }
    }

}

{% endhighlight %}

For more information please refer to the API documentation: **[MirrorStatistics](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/com/gigaspaces/cluster/replication/async/mirror/MirrorStatistics.html)**


# Monitoring the Remote Transport Activity

You may monitor the remote communication activity via the Administration and Monitoring API. You may receive information in real-time about every aspect of the communication and transport activity. See the [Monitoring LRMI via the Administration API](./communication-protocol.html#Monitoring LRMI via the Administration API) for details.
