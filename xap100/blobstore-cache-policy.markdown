---
layout: post100
title:  Flash drive IMDG Storage
categories: XAP100
parent: memory-management-overview.html
weight: 400
---

{% summary %}  {% endsummary %}


XAP 10 introduces a new Storage interface allowing Blob/Block/external type storage to act as a natural memory space for the IMDG. This storage model allows the IMDG to interact with the Storage interface as an integral data space together with the RAM/Heap based memory space. 

This storage model allows you to leverage high capacity storage such as Enterprise flash drives (SSD) as the IMDG memory storage area instead of the IMDG container Process (Heap). This storage model leveraging the RAM (Heap) to store indexes and SSD for the raw data in a serialized form.  


![blobstore1.jpg](https://dl.dropboxusercontent.com/u/7390820/blobstore1.jpg)


Heap (RAM) used also as a first level cache for frequently used data. Read operations (ById, Template or SQL based) for the same data will be first loaded from the SSD and later will be served from the RAM based cache.

## How the Storage Model is different than the Persistence Model?
Unlike the traditional Persistence model where the IMDG backend store is usually a database with relatively slow response time located in a central location , the Storage interface assumes very fast access time for write and read operations where each IMDG instance accessing a local dedicated store via key/value interface. Each IMDG primary and backup instance across the grid is interacting with its dedicated storage drive independently in an atomic manner. 

When running a traditional persistence configuration, the IMDG front-end the database, acting as a transactional cache storing a subset of the data. In this configuration data is loaded in a lazy manner to the IMDG as the assumption is RAM capacity is limited.

With the Storage Model the entire data is kept on SSD. The assumption is the SSD capacity across the grid is large enough to accommodate the entire application data set.

## BlobStore Target Applications

Real time analytics applications , Big Data applications , Customer 360 degree information applications would find the XAP Solid-State Drive flash-based IMDG Storage very useful. As these applications consume relatively large data capacity and have relatively fast data access requirements, reducing database dependency and leveraging distributed In-memory storage fabric to improve their performance, scalability, reliability and overall cost is important.

## Would Garbage Collection Activity will be affected?

Yes. Since the IMDG raw data is stored outside the Heap (SSD) the Garbage collector would not need to manage this memory. This is reducing pauses related to garbage collection. The SSD BlobStore increasing the amount of data an IMDG node can manage to the local SSD capacity (several TB) rather the maximum heap size (hundreds of GB). This allows you to manage hundreds of TB just with few IMDG nodes.

## Are all SSD Vendors Supported?

Yes. All Enterprise flash drives are supported. SanDisk, Fusion-IO, Intel® SSD , etc are supported with the IMDG Storage technology. Central SSD (RAID) devices such as Tegile, Cisco Whiptail , DSSD , Violin Memory are also supported.

## How it Works?
XAP is using [SanDisk Flash Data Fabric (FDF)](http://www.sandisk.com/) which enables direct flash access for In-Memory performance. It eliminate any other storage interface, when writing an object to the space , its indexed data maintained on Heap where the Storage interface implementing using the FDF libraries to interact with the underlying flash drive. 


![blobstore2.jpg](https://dl.dropboxusercontent.com/u/7390820/blobstore2.jpg)


The indexes maintain in RAM allowing the XAP query engine to evaluate the query without accessing the raw data stored on the flash device. This allows XAP to execute SQL based queries extremely efficiently even across large number of nodes. All XAP Data Grid APIs are supported including distributed transactions, leasing (Time To live) , FIFO , batch operations ,etc. All clustering topologies supported. All client side cache options are supported.

# The BlobStore Configuration

The BlobStore settings includes the following options:

{: .table .table-bordered}
| Property               | Description                                               | Default | Use |
|:-----------------------|:----------------------------------------------------------|:--------|:--------|
| devices | Flash devices. Comma separated available devices. The list used as a search path from left to right. The first one exists will be used. |  | required |
| volume-dir | Directory path which contains a link to the the SSD device. | | required |
| blob-store-capacity-GB | Flash device allocation size in Gigabytes. | 200 | optional |
| blob-store-capacity-MB | Flash device allocation size in Megabytes. | 204800 | optional |
| blob-store-capacity-MB | FDF internal cache size in Megabytes. | 100 | optional |
| blob-store-reformat | Whether to clear all SSD data in the space initialization phase or not. | false | optional |
| enable-admin | Whether to start an FDF admin agent or not. FDF admin provides a simple command line interface (CLI) through a TCP port. The FDF CLI uses port 51350 by default. This port can be changed through the configuration parameter FDF_ADMIN_PORT. | true |
| statistics-interval | Applications can optionally enable periodic dumping of statistics to a specified file (XAP_HOME/logs). This is disabled by default. | | optional |
| durability-level | PERIODIC: sync storage every 1024	writes. Up to 1024 objects can be lost in the event of application crash.{%wbr%}SW_CRASH_SAFE: This policy guarantees no data loss in the event of software crashes. But some data might be lost in the event of hardware failure.{%wbr%}HW_CRASH_SAFE: This policy guarantees no data loss if the hardware crashes.Since there are performance implication it is recommended to work with NVRAM device and configure log-flash-dir to a folder on this device. | PERIODIC | optional |
| log-flush-dir | In case of HW_CRASH_SAFE, directory in a file system on top of NVRAM backed disk. This directory must be unique per space, you can add ${clusterInfo.runningNumber} as suffix | /tmp | optional |

The IMDG BlobStore settings includes the following options:{%wbr%}

{: .table .table-bordered}
| Property | Description | Default | Use |
|:---------|:------------|:--------|:--------|
| blob-store-handler | BlobStore implementation |  | required |
| cache-entries-percentage | Cache percentage of the JVM max memory(-Xmx). In case of missing `-Xmx` configuration the cache size will `10000` objects. This is an LRU based data cache which store the entire IMDG object.| 20% | optional |
| avg-object-size-KB |  Average object size. | 5KB | optional |

# Installation
The SSD Storage library can be download from:
To apply the library simply copy it to :

{%note title=Supported OS%}XAP Flash Storage library is supported only CentOS 5.8-5.10{%endnote%}

# Configuration
Configuring an IMDG (Space) with BlobStore should be done via the `SanDiskBlobStoreDataPolicyFactoryBean`, or the `SanDiskBlobStoreConfigurer`. For example:

## PU XML Configuration
{% inittab os_simple_space|top %}
{% tabcontent Namespace %}

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:os-core="http://www.openspaces.org/schema/core"
       xmlns:blob-store="http://www.openspaces.org/schema/blob-store"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
       http://www.openspaces.org/schema/core http://www.openspaces.org/schema/10.0/core/openspaces-core.xsd
       http://www.openspaces.org/schema/blob-store http://www.openspaces.org/schema/10.0/blob-store/openspaces-blobstore.xsd">

    <bean id="propertiesConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer"/>

    <blob-store:sandisk-blob-store id="blobstoreid" volume-dir="/tmp" devices="/dev/xvdb,/dev/xvdc"
            blob-store-capacity-GB="200" blob-store-cache-size-MB="50" blob-store-reformat="true" durability-level="SW_CRASH_SAFE">
    </blob-store:sandisk-blob-store>

    <os-core:space id="space" url="/./mySpace" >
        <os-core:blob-store-data-policy blob-store-handler="blobstoreid" cache-entries-percentage="10" avg-object-size-KB="5"/>
    </os-core:space>

    <os-core:giga-space id="gigaSpace" space="space"/>
</beans>
{% endhighlight %}
{% endtabcontent %}

{% tabcontent Plain XML %}

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:os-core="http://www.openspaces.org/schema/core"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
       http://www.openspaces.org/schema/core http://www.openspaces.org/schema/10.0/core/openspaces-core.xsd">

    <bean id="propertiesConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer"/>

    <bean id="blobstoreid" class="com.gigaspaces.blobstore.storagehandler.SanDiskBlobStoreHandler">
        <property name="blobStoreCapacityGB" value="200"/>
        <property name="blobStoreCacheSizeMB" value="50"/>
        <property name="blobStoreDevices" value="/dev/xvdb,/dev/xvdc"/>
        <property name="blobStoreVolumeDir" value="/tmp"/>
        <property name="blobStoreDurabilityLevel" value="SW_CRASH_SAFE"/>
    </bean>

    <os-core:space id="space" url="/./mySpace" >
        <os-core:blob-store-data-policy blob-store-handler="blobstoreid" cache-entries-percentage="10"
            avg-object-size-KB="5"/>
    </os-core:space>

    <os-core:giga-space id="gigaSpace" space="space"/>
</beans>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Code %}


## Programmatic API
Programmatic approach to start a space using the `SanDiskBlobStoreConfigurer`:
{% highlight java %}
SanDiskBlobStoreConfigurer configurer = new SanDiskBlobStoreConfigurer();
configurer.addDevices("/dev/xvdb,/dev/xvdc");
configurer.addVolumeDir("/tmp");
configurer.setBlobStoreCapacityGB(200);
configurer.setBlobStoreCacheSizeMB(50);
configurer.setDurabilityLevel(DurabilityLevel.SW_CRASH_SAFE);

SanDiskBlobStoreHandler blobStoreHandler = configurer.create();
BlobStoreDataCachePolicy cachePolicy = new BlobStoreDataCachePolicy();
cachePolicy.setAvgObjectSizeKB(5l);
cachePolicy.setCacheEntriesPercentage(10);
cachePolicy.setBlobStoreHandler(blobStoreHandler);

UrlSpaceConfigurer urlConfig = new UrlSpaceConfigurer(spaceURL);
urlConfig.cachePolicy(cachePolicy);

gigaSpace = new GigaSpaceConfigurer(urlConfig).gigaSpace();
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

The above example:
- Configures the SanDisk BlobStore bean.{%wbr%}
- Configures the Space to use the above blobStore implementation also configure the cache size, in this LRU cache the space store also the data objects and not just the object indexes in RAM.

# Controlling BlobStore at the Space Class Level
When writing POJO to the Space, you may specify if the space class instances will be stored as the `BlobStore` or only in Memory.

Here is a sample POJO class disabling `BlobStore` for the Class `Person`:

{% highlight java %}
@SpaceClass(BlobStore = false)
public class Person {
    private Integer id;
    private String name;
    private String lastName;
    private int age;

    ...
    public Person() {}

    @SpaceId(autoGenerate=false)
    @SpaceRouting
    public Integer getId() { return id;}

    public void setId(Integer id) {  this.id = id; }

    @SpaceProperty(index=BASIC)
    public Long getLastName() { return lastName; }

    public void setLastName(String type) { this.lastName = lastName; }

    @SpaceProperty(nullValue="-1")
    public int getAge(){ return age; }

    .......
}
{% endhighlight %}


# The BlobStoreStorageHandler Interface

The `BlobStoreStorageHandler` Interface provides the following methods. You may customize it to have your specific functionality:  
{% highlight java %}
abstract class BlobStoreStorageHandler 
{
  public void initialize(String spaceName, Properties properties, boolean warmStart){};
  public abstract Object add(java.io.Serializable id,java.io.Serializable data, BlobStoreObjectType objectType);
  public abstract  java.io.Serializable get(java.io.Serializable id, Object position,  BlobStoreObjectType objectType);
  public abstract  Object replace(java.io.Serializable id,java.io.Serializable data,  Object position,  BlobStoreObjectType objectType);
  public abstract void  remove(java.io.Serializable id, Object position, BlobStoreObjectType objectType);
  public List<BlobStoreBulkOperationResult> executeBulk(List<BlobStoreBulkOperationRequest> operations, BlobStoreObjectType objectType, boolean transactional)
  public  DataIterator<BlobStoreGetBulkOperationResult> iterator(BlobStoreObjectType objectType)
  public Properties getProperties(){};
  public void close(){};
}
{% endhighlight %}

If you wish to add Async Persistency - Mirror to your IMDG please refer to [Async Persistency with Mirror](./asynchronous-persistency-with-the-mirror.html).
