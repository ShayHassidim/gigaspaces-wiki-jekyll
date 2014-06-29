---
layout: post100
title:  Cassandra Archive Handler
categories: XAP100
parent: cassandra.html
weight: 200
---




{%section%}
{%column width=60% %}

The [Archive Container](./archive-container.html) can be configured to work against Cassandra (without writing any extra code). The [ArchiveOperationHandler interface](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/org/openspaces/archive/ArchiveOperationHandler.html) abstracts the Big-Data storage from the [Archive Container](./archive-container.html). The Cassandra Archive Operation Handler implements this interface by [serializing](#Property Value Serializer) space objects into Cassandra.

{%endcolumn%}
{%column width=35% %}
![archive-container-cassandra.jpg](/attachment_files/archive-container-cassandra.jpg)
{%endcolumn%}
{%endsection%}



#### Library dependencies

The Cassandra Archive Operation Handler uses the [Hector Library](http://hector-client.github.com/hector/build/html/index.html) for communicating with the Cassandra cluster.
Include the following in your `pom.xml`

{% inittab %}
{% tabcontent hector using log4j %}

{% highlight xml %}
<dependency>
    <groupId>org.apache.cassandra</groupId>
    <artifactId>cassandra-clientutil</artifactId>
    <version>1.1.6</version>
</dependency>

<dependency>
    <groupId>org.apache.cassandra</groupId>
    <artifactId>cassandra-thrift</artifactId>
    <version>1.1.6</version>
</dependency>

<dependency>
    <groupId>org.hectorclient</groupId>
    <artifactId>hector-core</artifactId>
    <version>1.1-2</version>
</dependency>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent hector using java.util.logging %}

{% highlight xml %}
<dependency>
    <groupId>org.apache.cassandra</groupId>
    <artifactId>cassandra-clientutil</artifactId>
    <version>1.1.6</version>
</dependency>

<dependency>
    <groupId>org.apache.cassandra</groupId>
    <artifactId>cassandra-thrift</artifactId>
    <version>1.1.6</version>
</dependency>

<dependency>
    <groupId>org.hectorclient</groupId>
    <artifactId>hector-core</artifactId>
    <version>1.1-2</version>
    <exclusions>
        <exclusion>
	    <groupId>org.slf4j</groupId>
	    <artifactId>slf4j-log4j12</artifactId>
	</exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.6.6</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-jdk14</artifactId>
    <version>1.6.6</version>
</dependency>
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

#### Setup

{% inittab os_simple_space|top %}
{% tabcontent Namespace %}

{% highlight xml %}

<os-archive:cassandra-archive-handler id="cassandraArchiveHandler"
  giga-space="gigaSpace"
  hosts="127.0.0.1"
  port="9160"
  keyspace="mykeyspace"
  write-consistency="QUORUM"
/>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Plain XML %}

{% highlight xml %}

<bean id="cassandraArchiveHandler" class="org.openspaces.persistency.cassandra.archive.CassandraArchiveOperationHandler">
	<property name="gigaSpace" ref="gigaSpace"/>
	<property name="hosts" value="127.0.0.1" />
	<property name="port" value="9160" />
	<property name="keyspace" value="mykeyspace" />
	<property name="writeConsistency" value="QUORUM" />
</bean>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Code %}

{% highlight java %}

ArchiveOperationHandler cassandraArchiveHandler =
    new CassandraArchiveOperationHandlerConfigurer()
    .gigaSpace(gigaSpace)
    .hosts("127.0.0.1")
    .port(9160)
    .keyspace("mykeyspace")
    .writeConsistency(CassandraConsistencyLevel.QUORUM)
    .create();

// To free the resources used by the archive container make sure you close it properly.
// A good life cycle event to place the destroy() call would be within the @PreDestroy or DisposableBean#destroy() method.

archiveContainer.destroy();
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

#### CassandraArchiveOperationHandler Properties

{: .table .table-bordered}
|Property|Description|
|:-------|:----------|
|gigaSpace| GigaSpace reference used for type descriptors. see [Archive Container#Configuration](./archive-container.html#Configuration)|
|hosts | Comma separated list of Cassandra host names or ip addresses|
|port | Cassandra port. By default uses 9160|
|keyspace | Cassandra keyspace|
|propertyValueSerializer|see [Property Value Serializer](#Property Value Serializer).|
|flattenedPropertiesFilter| see [Flattened Properties Filter](./cassandra-space-synchronization-endpoint.html#Flattened Properties Filter).|
|columnFamilyNameConverter| see [Column Family Name Converter](./cassandra-space-synchronization-endpoint.html#Column Family Name Converter).|

{%comment%}
#### XSD Schema

- <os-archive:cassandra-archive-handler> schema:

{% indent %}
![cassandra-archive-handler-schema-9-1-1.png](/attachment_files/cassandra-archive-handler-schema-9-1-1.png)
{% endindent %}

{%endcomment%}

#### Property Value Serializer

By default when serializing object/document properties to column values, the following serialization logic is applied:

- If the type of the value to be serialized matches a primitive type in Cassandra it will be serialized as defined by the Cassandra primitive type serialization protocol.
- Otherwise, the value will be serialized using standard java Object serialization mechanism.

It is possible to override this default behavior by providing a custom implementation of [PropertyValueSerializer](http://www.gigaspaces.com/docs/JavaDoc{% currentversion %}/index.html?org/openspaces/persistency/cassandra/meta/types/dynamic/PropertyValueSerializer.html).
This interface is defined by these 2 methods:

{% highlight java %}
ByteBuffer toByteBuffer(Object value);
Object fromByteBuffer(ByteBuffer byteBuffer);
{% endhighlight %}

Properties will only be serialized by the custom serializer if their type does not match a primitive type in Cassandra.

### Known Limitations

The CassandraArchiveHandler has the following known limitations:
 *  The archiver must not write two different entries with the same ID. This would corrupt the entry in Cassandra.
 *  Only Space Documents are supported. You can still write POJOs to the space, but the @EventTemplate used for taking objects from the space must be a SpaceDocument.
 *  The archiver is thread safe
 *  The archiver is idempotent as long as there are no two threads that are writing two different objects with the same space id.
 *  Both fixed and dynamic space properties are serialized with the same propertyValueSerializer.
