---
layout: post
title:  Executor Based Remoting
categories: XAP97
parent: space-based-remoting.html
weight: 100
---

{% compositionsetup %}
{% summary page %}Executor Remoting allows you to use remote invocations of POJO services, with the space as the transport layer using OpenSpaces Executors.{% endsummary %}

# Overview

Executor based remoting uses [Executors](./task-execution-over-the-space.html) to implement remoting capabilities on top of the space. Executor based remoting allows for direct invocation of services in an asynchronous manner as well as broadcast capabilities. Executor remoting works with services that are exposed within a processing unit that started a collocated space.

![Executor.jpg](/attachment_files/Executor.jpg)

<iframe width="640" height="360" src="//www.youtube.com/embed/-07-0PXUoeM?feature=player_detailpage" frameborder="0" allowfullscreen></iframe>
# The executor-proxy

The `executor-proxy` should be created at the client side to interact with the remote service. It can be created via Spring namespace , Sppring Bean or via API .

The `executor-proxy` include the following properties:

{: .table .table-bordered}
| property| Class Type | Required | NameSpace Bean | Description | Default |
|:--------|:-----------|:---------|:---------------|:------------|:--------|
|gigaSpace | GigaSpace | giga-space | Yes |Sets the GigaSpace interface that will be used to work with the space as the transport layer for executions of Space tasks.| |
|timeout | long |  Yes | timeout | Sets the timeout that will be used to wait for the remote invocation response. The timeout value is in milliseconds.  | 60000 (60 seconds).|
|remoteRoutingHandler | RemoteRoutingHandler | No | routing-handler |In case of remote invocation over a partitioned space the default partitioned routing index will be random (the hashCode of the newly created remote invocation). This RemoteRoutingHandler allows for custom routing computation (for example, based on one of the service method parameters).| |
|metaArgumentsHandler | MetaArgumentsHandler | No | meta-arguments-handler | Allows to set the meta arguemnts handler controlling the meta arguments passed with each remote invocation.| |
|asyncMethodPrefix | String |
|broadcast | boolean | No | broadcast| If set the true (defaults to false) causes the remote invocation to be called on all active (primary) cluster members.| |
|returnFirstResult | boolean |No | return-first-result | When set to true (defaults to true) will return the first result when using broadcast. If set to false, an array of results will be retuned. Note, this only applies when not setting a reducer.| |
|remoteResultReducer | RemoteResultReducer |No| result-reducer | When using broadcast set to true, allows to plug a custom reducer that can reduce the array of result objects into another response object.| |
| remoteInvocationAspect | RemoteInvocationAspect |No| Allows to set a aspect around each service execution.| |
|serviceProxy | Object |
| methodHashLookup | Map<Method, MethodHash> |
|interface | Class | Yes | | The interface (fully qualified class name) that this remoting proxy will proxy. Also controls which service will be invoked in the "server".| |

Example:
{% highlight xml %}
<bean id="dataProcessor" class="org.openspaces.remoting.ExecutorSpaceRemotingProxyFactoryBean">
    <property name="gigaSpace" ref="gigaSpace" />
    <property name="timeout" value="60000" />

    <property name="serviceInterface" value="org.openspaces.example.data.common.IDataProcessor" />
    <property name="broadcast" value="true" />

    <property name="remoteInvocationAspect">
    	<bean class="eg.MyRemoteInvocationAspect" />
    </property>

	<property name="metaArgumentsHandler">
    	<bean class="eg.MyMetaArgumentsHandler" />
    </property>

	<property name="remoteResultReducer">
    	<bean class="eg.MyRemoteResultReducer" />
    </property>

	<property name="remoteRoutingHandler">
        <bean class="org.openspaces.example.data.feeder.support.DataRemoteRoutingHandler"/>
    </property>
</bean>
{% endhighlight %}

# Defining the Contract

In order to support remoting, the first step is to define the contract between the client and the server. In our case, the contract is a simple Java interface. Here is an example:

{% highlight java %}
public interface IDataProcessor {

    /**
     * Process a given Data object, returning the processed Data object.
     */
    Data processData(Data data);
}
{% endhighlight %}

{% exclamation %} The `Data` object should be `Serializable`, or better yet, `Externalizable` (for better performance).

# Implementing the Contract

Next, an implementation of this contract needs to be provided. This implementation will "live" on the server side. Here is a sample implementation:

{% inittab os_simple_space|top %}
{% tabcontent Annotation %}

{% highlight java %}
@RemotingService
public class DataProcessor implements IDataProcessor {

    public Data processData(Data data) {
    	data.setProcessor(true);
    	return data;
    }
}
{% endhighlight %}

{% endtabcontent %}
{% tabcontent XML %}

{% highlight java %}
public class DataProcessor implements IDataProcessor {

    public Data processData(Data data) {
    	data.setProcessor(true);
    	return data;
    }
}
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

The XML tab corresponds to exporting the service using an xml configuration (explained in the next section). The Annotation tab corresponds to exporting the service using annotations.

# Exporting the Service Over the Space

The next step is exporting the service over the space. Exporting the service is done on the server side. As with other Spring remoting support, exporting the service and the method of exporting the service is a configuration-time decision. Here is an example of a Spring XML configuration:

{% inittab os_simple_space|top %}
{% tabcontent Annotation %}

{% highlight xml %}
<!-- Support @RemotingService component scanning -->
<context:component-scan base-package="com.mycompany"/>

<!-- Support the @RemotingService annotation on a service-->
<os-remoting:annotation-support />

<os-core:space id="space" url="/./space" />

<os-core:giga-space id="gigaSpace" space="space"/>

<os-remoting:service-exporter id="serviceExporter" />
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Namespace %}

{% highlight xml %}
<os-core:space id="space" url="/./space" />

<os-core:giga-space id="gigaSpace" space="space"/>

<bean id="dataProcessor" class="DataProcessor" />

<os-remoting:service-exporter id="serviceExporter">
     <os-remoting:service ref="dataProcessor"/>
</os-remoting:service-exporter>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Plain XML %}

{% highlight xml %}
<bean id="space" class="org.openspaces.core.space.UrlSpaceFactoryBean">
    <property name="url" value="/./space" />
</bean>

<bean id="gigaSpace" class="org.openspaces.core.GigaSpaceFactoryBean">
	<property name="space" ref="space" />
</bean>

<bean id="dataProcessor" class="DataProcessor" />

<bean id="serviceExporter" class="org.openspaces.remoting.SpaceRemotingServiceExporter">
    <property name="services">
        <list>
            <ref bean="dataProcessor" />
        </list>
    </property>
</bean>
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

The name/id of the `SpaceRemotingServiceExporter` bean is important when using executor based remoting. By default, the `Task` representing the remote invocation searches for a service exporter named `serviceExporter`, otherwise, it uses the first bean that is of the type `SpaceRemotingServiceExporter`.

{% infosign %} Exporting services is done on a Processing Unit (or a Spring application context) that starts an embedded space.

# Using the Service on the Client Side

In order to use the exported `IDataProcessor` on the client side, beans should use the `IDataProcessor` interface directly:

{% highlight java %}
public class DataRemoting {

    private IDataProcessor dataProcessor;

    public void setDataProcessor(IDataProcessor dataProcessor) {
        this.dataProcessor = dataProcessor;

    }
}
{% endhighlight %}

Configuring the `IDataProcessor` proxy can done in the following manner:

{% inittab os_simple_space|top %}
{% tabcontent Namespace %}

{% highlight xml %}
<os-core:space id="space" url="jini://**/**/space" />

<os-core:giga-space id="gigaSpace" space="space"/>

<os-remoting:executor-proxy id="dataProcessor" giga-space="gigaSpace"
                   interface="org.openspaces.example.data.common.IDataProcessor">
</os-remoting:executor-proxy>

<bean id="dataRemoting" class="DataRemoting">
    <property name="dataProcessor" ref="dataProcessor" />
</bean>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Plain XML %}

{% highlight xml %}
<bean id="space" class="org.openspaces.core.space.UrlSpaceFactoryBean">
    <property name="url" value="jini://*/*/space" />
</bean>

<bean id="gigaSpace" class="org.openspaces.core.GigaSpaceFactoryBean">
	<property name="space" ref="space" />
</bean>

<bean id="dataProcessor" class="org.openspaces.remoting.ExecutorSpaceRemotingProxyFactoryBean">
    <property name="gigaSpace" ref="gigaSpace" />
    <property name="serviceInterface" value="org.openspaces.example.data.common.IDataProcessor" />
</bean>

<bean id="dataRemoting" class="DataRemoting">
    <property name="dataProcessor" ref="dataProcessor" />
</bean>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Code %}

{% highlight java %}
IJSpace space = new UrlSpaceConfigurer("jini://*/*/space").space();

GigaSpace gigaSpace = new GigaSpaceConfigurer(space).gigaSpace();

IDataProcessor dataProcessor = new ExecutorRemotingProxyConfigurer<IDataProcessor>(gigaSpace, IDataProcessor.class)
                               .proxy();

DataRemoting dataRemoting = new DataRemoting();

dataRemoting.setDataProcessor(dataProcessor);
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

The above example uses the `executor-proxy` bean in order to create the remoting proxy which can then be injected into the `DataRemoting` object. OpenSpaces remoting also allows you to inject the remoting proxy directly on the remoting service property using annotations. Here is an example using the `DataRemoting` class:

{% highlight java %}
public class DataRemoting {

    @ExecutorProxy
    private IDataProcessor dataProcessor;

    // ...
}
{% endhighlight %}

If there are more than one `GigaSpace` beans defined within the application context, the ID of the `giga-space` bean needs to be defined on the annotations.

In order to enable this feature, the following element needs to be added to the application context XML:

{% inittab os_simple_space|top %}
{% tabcontent Namespace %}

{% highlight xml %}
<os-remoting:annotation-support />
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Plain XML %}

{% highlight xml %}
<bean id="space" class="org.openspaces.remoting.RemotingAnnotationBeanPostProcessor" />
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

## Remote Routing Handler

Many times, space remoting is done by exporting services in a space with a partitioned cluster topology. The service is exported when working directly with a cluster member (and not against the whole space cluster). When working with such a topology, the client side remoting automatically generates a random routing index value. In order to control the routing index, the following interface can be implemented:

{% highlight java %}
public interface RemoteRoutingHandler<T> {

    /**
     * Returns the routing field value based on the remoting invocation. If <code>null</code>
     * is returned, will use internal calcualtion of the routing index.
     */
    T computeRouting(SpaceRemotingInvocation remotingEntry);
}
{% endhighlight %}

Here is a sample implementation which uses the first parameter `Data` object type as the routing index.

{% highlight java %}
public class DataRemoteRoutingHandler implements RemoteRoutingHandler<Long> {

    public Long computeRouting(SpaceRemotingInvocation remotingEntry) {
        if (remotingEntry.getMethodName().equals("processData")) {
            Data data = (Data) remotingEntry.getArguments()[0];
            return data.getType();
        }
        return null;
    }
}
{% endhighlight %}

Finally, the wiring is done in the following manner:

{% inittab os_simple_space|top %}
{% tabcontent Namespace %}

{% highlight xml %}
<os-core:space id="space" url="jini://*/*/space" />

<os-core:giga-space id="gigaSpace" space="space"/>

<os-remoting:executor-proxy id="dataProcessor" giga-space="gigaSpace"
                   interface="org.openspaces.example.data.common.IDataProcessor">
    <os-remoting:routing-handler>
        <bean class="org.openspaces.example.data.feeder.support.DataRemoteRoutingHandler"/>
    </os-remoting:routing-handler>
</os-remoting:executor-proxy>

<bean id="dataRemoting" class="DataRemoting">
    <property name="dataProcessor" ref="dataProcessor" />
</bean>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Plain XML %}

{% highlight xml %}
<bean id="space" class="org.openspaces.core.space.UrlSpaceFactoryBean">
    <property name="url" value="jini://*/*/space" />
</bean>

<bean id="gigaSpace" class="org.openspaces.core.GigaSpaceFactoryBean">
	<property name="space" ref="space" />
</bean>

<bean id="dataProcessor" class="org.openspaces.remoting.ExecutorSpaceRemotingProxyFactoryBean">
    <property name="gigaSpace" ref="gigaSpace" />
    <property name="serviceInterface" value="org.openspaces.example.data.common.IDataProcessor" />
    <property name="remoteRoutingHandler">
        <bean class="org.openspaces.example.data.feeder.support.DataRemoteRoutingHandler"/>
    </property>
</bean>

<bean id="dataRemoting" class="DataRemoting">
    <property name="dataProcessor" ref="dataProcessor" />
</bean>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Code %}

{% highlight java %}
IJSpace space = new UrlSpaceConfigurer("jini://*/*/space").space();

GigaSpace gigaSpace = new GigaSpaceConfigurer(space).gigaSpace();

IDataProcessor dataProcessor = new ExecutorRemotingProxyConfigurer<IDataProcessor>(gigaSpace, IDataProcessor.class)
                               .remoteRoutingHandler(new DataRemoteRoutingHandler())
                               .proxy();

DataRemoting dataRemoting = new DataRemoting();

dataRemoting.setDataProcessor(dataProcessor);
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

## Routing Annotation

The above option of using the remote routing handler is very handy when not using annotations. OpenSpaces remoting supports the `@Routing` annotation in order to define which of the parameters control the routing. Here is an example:

{% highlight java %}
public interface MyService {

    void doSomething(@Routing int param1, int param2);
}
{% endhighlight %}

In the above example, the routing is done using the param1 value. When using complex objects, a method name can be specified to be invoked on the parameter, in order to get the actual routing index. Here is an example:

{% highlight java %}
public interface MyService {

    void doSomething(@Routing("getProperty") Value param1, int param2);
}
{% endhighlight %}

In the above example, the `getProperty` method is called on the `Value` object, and its return value is used to extract the routing index.

{% exclamation %} **Note:** Using a [SpaceDocument](./document-api.html) as the annotated routing argument will cause an exception since the `SpaceDocument` class does not define a getter method for each property, but rather a generic getter method to get a property value by its name. The solution is either to extend the `SpaceDocument` class as described in [Extending Space Documents](./extending-space-documents.html), and define the relevant `getProperty` method in the extension class, or use the `RemoteRoutingHandler` mentioned above.

# Asynchronous Execution

OpenSpaces executor remoting supports asynchronous execution of remote services using the `AsyncFuture` interface (which extends the `java.util.concurrent.Future` interface). Since the remoting implementation is asynchronous by nature, a `Future` can be used to execute remote services asynchronously.

Since the definition of the interface acts as the contract between the client and the server, asynchronous invocation requires the interface to also support the same business method, returning a `Future` or `AsyncFuture` instead of its actual return value. The asynchronous nature is only relevant on the client side, provided by the asynchronous nature of OpenSpaces remoting. There are two options to implement such behavior:

## Different Service Interfaces

The client and the server can have a different service interface definition under the same package and under the same name. The server includes the actual implementation:

{% highlight java %}
public interface SimpleService {

    String say(String message);

    int calc(int value1, int value2);
}
{% endhighlight %}

The client has the same method except it returns a future:

{% highlight java %}
public interface SimpleService {

    AsyncFuture<String> say(String message);

    int calc(int value1, int value2);
}
{% endhighlight %}

In the above example, the `say()` method uses a `Future` in order to receive the result, while the `calc` method remains synchronous.

{% infosign %} When using different services, it is important to include the interface definition in the processing unit `lib` directory (or the compiled classes root directory), so that both client and server use their own definitions and don't share a single one.

## Single Service Interface

Both the client and server share the same interface, with the interface holding both synchronous and asynchronous services. Here is an example:

{% highlight java %}
public interface SimpleService {

    String say(String message);

    AsyncFuture<String> asyncSay(String message);
}
{% endhighlight %}

In the above case, the server implementation does not need to implement the `asyncSay` method (simply return `null` for the `asyncSay` method). The client side can choose which service to invoke.
You should note the naming convention and the signature of the asynchronous method in this case.
If the regular method looks as follows:
`T doSomething(String arg)`, T being the return value type, then the asynchronous method should have the following signature:
`Future<T> asyncDoSomething(String arg)`.
In other words, the regular method should be prefixed with `async` and the return value should be a `Future` of the original return value (unless it's `void`, in which case it stays `void`).

# Server Side Services Injection

Each parameter of a remote service can be dynamically injected as if it were a bean defined within the server side (where the service is actually executed) context. This allows you to utilize server side resources and state, within the actual invocation. In order to enable such injection the service interface should either be annotated with `AutowireArguments` annotation, or implement the marker interface `AutowireArgumentsMarker`. Let's see an example:

{% highlight java %}
@AutowireArguments
public interface MyService {

    void doSomething(Value param1);
}
{% endhighlight %}

The above service exposes a service which accepts a `Value` as a parameter. The `Value` parameter, if needed, can be injected dynamically with server side resources and state. For example, if the server side execution needs to be aware of the `ClusterInfo`, the `Value` parameter can implement the `ClusterInfoAware` interface. Here is what it looks like:

{% highlight java %}
public class Value implements ClusterInfoAware {

    private transient ClusterInfo clusterInfo;

    public void setClusterInfo(ClusterInfo clusterInfo) {
        this.clusterInfo = clusterInfo;
    }
}
{% endhighlight %}

{% infosign %} Note the fact that `clusterInfo` is transient, so it won't be marshalled after injection back to the client.

Another example is injecting another service that is defined in the server side directly into the parameter, for example, using Spring `@Autowired` annotation.

{% highlight java %}
public class Value implements ClusterInfoAware {

    @Autowired
    private transient AnotherService anotherService;
}
{% endhighlight %}

# Transactional Execution

Executor remoting supports transactional execution of services. On the client side, if there is an ongoing declarative transaction during the service invocation (a Space based transaction), the service will be executed under the same transaction. The transaction itself is passed to the server and any Space related operations (performed using `GigaSpace`) will be executed under the same transaction. The transaction lifecycle itself is controlled on the client side (declaratively) and is committed / rolled back only on the client side. (Note, exceptions on the server side will simply propagate to the client side, and will cause a rollback only if the client "decides" so.)

When using broadcast with executor remoting, a declarative distributed transaction must be used and not a local (or JTA one). Also note, the `GigaSpace` instance that is used internally by the executor remoting invocation must be configured with a transaction manager (as is the case for enabling transaction execution with any other `GigaSpace` based operation).

Here is a simple example how the Client and the Service should be configured:

{% highlight java %}
<context:component-scan base-package="com.demo"/>
<os-core:space id="space" url="jini://*/*/space" />

<os-core:distributed-tx-manager id="myTransactionManager" />
<os-core:giga-space id="gigaspace" space="space"  tx-manager="myTransactionManager"/>
<tx:annotation-driven transaction-manager="myTransactionManager" />

<os-remoting:executor-proxy id="myService" giga-space="gigaspace" interface="com.demo.IMyService" broadcast="true">
<os-remoting:result-reducer>
	<bean class="com.demo.MyRemoteResultReducer" />
	</os-remoting:result-reducer>
	</os-remoting:executor-proxy>

<bean id="callingBean" class="com.demo.ClientWarrper">
           <property name="client" ref="client"/>
</bean>

<bean id="client" class="com.demo.Client">
	   <property name="myService" ref="myService" />
	   <property name="gigaspace" ref="gigaspace" />
</bean>
{% endhighlight %}

{% highlight java %}
@Transactional(readOnly = true , value="myTransactionManager")
public class Client{
	GigaSpace gigaspace ;
	IMyService myService ;

	public static void main(String[] args) {
	}

	@Transactional(propagation=Propagation.REQUIRES_NEW , value="myTransactionManager")
	public void doTX() throws Exception
	{
		MyData myObj = new MyData(1);
		gigaspace.write(myObj);

		System.out.println("Client - TX:" + gigaspace.getCurrentTransaction());
		boolean ret = myService.doTX();
		System.out.println("Service Call Done! - Result:"+ret);
	}
}
{% endhighlight %}

{% highlight java %}
<context:component-scan base-package="com.demo"/>
<os-remoting:annotation-support />
<os-core:space id="space" url="/./space" />
<os-core:giga-space id="gigaSpace" space="space"/>
<os-remoting:service-exporter id="serviceExporter" />
{% endhighlight %}

{% highlight java %}
@RemotingService
@Transactional(readOnly = true)
public class MyService implements IMyService {
	public MyService()	{}
	@Autowired
	GigaSpace gigaSpace;

	@Transactional(propagation=Propagation.SUPPORTS)
	public boolean doTX()
	{
		MyData myObj = new MyData(2);
		gigaSpace.write(myObj);
		System.out.println("Service - TX:" + gigaSpace.getCurrentTransaction());
	}
}
{% endhighlight %}

# Execution Aspects

Space based remoting allows you to inject "aspects" that can wrap the invocation of a remote method on the client side, as well as wrapping the execution of an invocation on the server side.

## The Client Invocation Aspect

The client invocation aspect interface is shown below. You should implement this interface and wire and instance of the implementing class to the client side remote proxy, as explained below:

{% highlight java %}
public interface RemoteInvocationAspect<T> {

    /**
     * The aspect is called instead of the actual remote invocation. The methodInvocation can
     * be used to access any method related information. The remoting invoker passed can be used
     * to actually invoke the remote invocation.
     */
    T invoke(MethodInvocation methodInvocation, RemotingInvoker remotingInvoker) throws Throwable;
}
{% endhighlight %}

An implementation of such an aspect can be configured as follows:

{% inittab os_simple_space|top %}
{% tabcontent Namespace %}

{% highlight xml %}
<os-core:space id="space" url="jini://*/*/space" />

<os-core:giga-space id="gigaSpace" space="space"/>

<os-remoting:executor-proxy id="dataProcessor" giga-space="gigaSpace"
                   interface="org.openspaces.example.data.common.IDataProcessor">
    <os-remoting:aspect>
    	<bean class="eg.MyRemoteInvocationAspect" />
    </os-remoting:aspect>
</os-remoting:executor-proxy>

<bean id="dataRemoting" class="DataRemoting">
    <property name="dataProcessor" ref="dataProcessor" />
</bean>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Plain XML %}

{% highlight xml %}
<bean id="space" class="org.openspaces.core.space.UrlSpaceFactoryBean">
    <property name="url" value="jini://*/*/space" />
</bean>

<bean id="gigaSpace" class="org.openspaces.core.GigaSpaceFactoryBean">
	<property name="space" ref="space" />
</bean>

<bean id="dataProcessor" class="org.openspaces.remoting.ExecutorSpaceRemotingProxyFactoryBean">
    <property name="gigaSpace" ref="gigaSpace" />
    <property name="serviceInterface" value="org.openspaces.example.data.common.IDataProcessor" />
    <property name="remoteInvocationAspect">
    	<bean class="eg.MyRemoteInvocationAspect" />
    </property>
</bean>

<bean id="dataRemoting" class="DataRemoting">
    <property name="dataProcessor" ref="dataProcessor" />
</bean>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Code %}

{% highlight java %}
IJSpace space = new UrlSpaceConfigurer("jini://*/*/space").space();

GigaSpace gigaSpace = new GigaSpaceConfigurer(space).gigaSpace();

IDataProcessor dataProcessor = new ExecutorRemotingProxyConfigurer<IDataProcessor>(gigaSpace, IDataProcessor.class)
                               .remoteInvocationAspect(new MyRemoteInvocationAspect())
                               .proxy();

DataRemoting dataRemoting = new DataRemoting();

dataRemoting.setDataProcessor(dataProcessor);
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

The instance of the class that implements the `RemoteInvocationAspect` interface will be called by the framework prior to the invocation on the client side. Note that it is up to the aspect implementation to decide whether or not the call should proceed.
If the aspect implementation decides to proceed with the call it should call `method.invoke()` on the provided `method` argument. Otherwise the call to the server will not be performed.

{% anchor serverExecutionApect %}

## The Server Invocation Aspect

The server side invocation aspect interface is shown below. You should implement this interface and wire and instance of the implementing class to the server side service exporter (this is the component that is responsible for exposing your service bean to remote clients):

{% highlight java %}
public interface ServiceExecutionAspect {

    /**
     * A service execution callback allows you to wrap the execution of "server side" service. If
     * actual execution of the service is needed, the <code>invoke</code> method will need to
     * be called on the passed <code>Method</code> using the service as the actual service to
     * invoke it on, and {@link SpaceRemotingInvocation#getArguments()} as the method arguments.
     *
     * <p>As an example: <code>method.invoke(service, invocation.getArguments())</code>.
     */
    Object invoke(SpaceRemotingInvocation invocation, Method method, Object service)
                                  throws InvocationTargetException, IllegalAccessException;
}
{% endhighlight %}

An implementation of such an aspect can be configured as follows:

{% inittab os_simple_space|top %}
{% tabcontent Namespace %}

{% highlight xml %}
<os-core:space id="space" url="/./space" />

<os-core:giga-space id="gigaSpace" space="space"/>

<bean id="dataProcessor" class="DataProcessor" />

<os-remoting:service-exporter id="serviceExporter">
     <os-remoting:aspect>
         <bean class="eg.MyServiceExecutionAspect" />
     </os-remoting:aspect>
     <os-remoting:service ref="dataProcessor"/>
</os-remoting:service-exporter>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Plain XML %}

{% highlight xml %}
<bean id="space" class="org.openspaces.core.space.UrlSpaceFactoryBean">
    <property name="url" value="/./space" />
</bean>

<bean id="gigaSpace" class="org.openspaces.core.GigaSpaceFactoryBean">
	<property name="space" ref="space" />
</bean>

<bean id="dataProcessor" class="DataProcessor" />

<bean id="serviceExporter" class="org.openspaces.remoting.SpaceRemotingServiceExporter">
    <property name="serviceExecutionAspect">
         <bean class="eg.MyServiceExecutionAspect" />
    </property>
    <property name="services">
        <list>
            <ref bean="dataProcessor" />
        </list>
    </property>
</bean>
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

The instance of the class that implements the `ServiceExecutionAspect` interface will be called by the framework prior to the invocation of the service bean on the server side. Note that it is up to the aspect implementation to decide whether or not the call should proceed.
If the aspect implementation decides to proceed with the call it should call `method.invoke()` on the provided `method` argument. Otherwise the call to the service bean will not be performed.

# The Metadata Handler

When executing a service using Space based remoting, a set of one or more metadata arguments (in the form of a `java.lang.Object` array) can be passed to the server with the remote invocation. This feature is very handy when you want to pass certain metadata with every remote call that is not part of the arguments of the method you call. A prime example for using meta arguments is security -- i.e. passing security credentials as meta arguments, and using a server side aspect to authorize the execution or to log who actually called the method.

To create the meta arguments on the client side you should implement the following interface and inject an instance of the implementing class to the client side proxy:

{% highlight java %}
public interface MetaArgumentsHandler {

    /**
     * Meta argument handler can control the meta data objects that will be used
     * for the remote invocation.
     */
    Object[] obtainMetaArguments(SpaceRemotingInvocation remotingEntry);
}
{% endhighlight %}

The following snippets show how to plug a custom meta arguments handler to the client side remote proxy. The `Object` array returned by the implementation of the `MetaArgumentsHandler` interface will be sent along with the invocation to server side.

{% inittab os_simple_space|top %}
{% tabcontent Namespace %}

{% highlight xml %}

<os-core:space id="space" url="jini://*/*/space" />

<os-core:giga-space id="gigaSpace" space="space"/>

<os-remoting:executor-proxy id="dataProcessor" giga-space="gigaSpace"
                   interface="org.openspaces.example.data.common.IDataProcessor">
    <os-remoting:meta-arguments-handler>
    	<bean class="eg.MyMetaArgumentsHandler" />
    </os-remoting:meta-arguments-handler>
</os-remoting:executor-proxy>

<bean id="dataRemoting" class="DataRemoting">
    <property name="dataProcessor" ref="dataProcessor" />
</bean>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Plain XML %}

{% highlight xml %}

<bean id="space" class="org.openspaces.core.space.UrlSpaceFactoryBean">
    <property name="url" value="jini://*/*/space" />
</bean>

<bean id="gigaSpace" class="org.openspaces.core.GigaSpaceFactoryBean">
	<property name="space" ref="space" />
</bean>

<bean id="dataProcessor" class="org.openspaces.remoting.ExecutorSpaceRemotingProxyFactoryBean">
    <property name="gigaSpace" ref="gigaSpace" />
    <property name="serviceInterface" value="org.openspaces.example.data.common.IDataProcessor" />
    <property name="metaArgumentsHandler">
    	<bean class="eg.MyMetaArgumentsHandler" />
    </property>
</bean>

<bean id="dataRemoting" class="DataRemoting">
    <property name="dataProcessor" ref="dataProcessor" />
</bean>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Code %}

{% highlight java %}
IJSpace space = new UrlSpaceConfigurer("jini://*/*/space").space();

GigaSpace gigaSpace = new GigaSpaceConfigurer(space).gigaSpace();

IDataProcessor dataProcessor = new ExecutorRemotingProxyConfigurer<IDataProcessor>(gigaSpace, IDataProcessor.class)
                               .metaArgumentsHandler(new MyMetaArgumentsHandler())
                               .proxy();

DataRemoting dataRemoting = new DataRemoting();

dataRemoting.setDataProcessor(dataProcessor);
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

The way to access the meta arguments on the server side is to configure a [server side execution aspect](#serverExecutionApect) by implementing the `ServiceExecutionAspect` and wiring it on the server side as shown [above](#serverExecutionApect). To access the meta arguments, you should call `SpaceRemotingInvocation.getMetaArguments()` on the `invocation` argument provided to the server side aspect.

# Broadcast Remoting

When using executor remoting, a remote invocation can be broadcasted to all active (primary) cluster members. Each Service instance is invoked and return a result to its called which in turn reduce these and pass the final result to the application.

The First phase involves the Service invocation:
![Executor1.jpg](/attachment_files/Executor1.jpg)

The Second phase involves reducing the results retrieved from the Services:
![Executor2.jpg](/attachment_files/Executor2.jpg)

The configuration of enabling broadcasting is done on the client level, by setting the broadcast flag to `true`:

{% inittab os_simple_space|top %}
{% tabcontent Namespace %}

{% highlight xml %}
<os-core:space id="space" url="jini://*/*/space" />

<os-core:giga-space id="gigaSpace" space="space"/>

<os-remoting:executor-proxy id="dataProcessor" giga-space="gigaSpace"
                   interface="org.openspaces.example.data.common.IDataProcessor" broadcast="true">
</os-remoting:executor-proxy>

<bean id="dataRemoting" class="DataRemoting">
    <property name="dataProcessor" ref="dataProcessor" />
</bean>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Plain XML %}

{% highlight xml %}
<bean id="space" class="org.openspaces.core.space.UrlSpaceFactoryBean">
    <property name="url" value="jini://*/*/space" />
</bean>

<bean id="gigaSpace" class="org.openspaces.core.GigaSpaceFactoryBean">
	<property name="space" ref="space" />
</bean>

<bean id="dataProcessor" class="org.openspaces.remoting.ExecutorSpaceRemotingProxyFactoryBean">
    <property name="gigaSpace" ref="gigaSpace" />
    <property name="serviceInterface" value="org.openspaces.example.data.common.IDataProcessor" />
    <property name="broadcast" value="true" />
</bean>

<bean id="dataRemoting" class="DataRemoting">
    <property name="dataProcessor" ref="dataProcessor" />
</bean>
{% endhighlight %}

{% endtabcontent %}
{% tabcontent Code %}

{% highlight java %}
IJSpace space = new UrlSpaceConfigurer("jini://*/*/space").space();

GigaSpace gigaSpace = new GigaSpaceConfigurer(space).gigaSpace();

IDataProcessor dataProcessor = new ExecutorRemotingProxyConfigurer<IDataProcessor>(gigaSpace, IDataProcessor.class)
                               .broadcast(true)
                               .proxy();

DataRemoting dataRemoting = new DataRemoting();

dataRemoting.setDataProcessor(dataProcessor);
{% endhighlight %}

{% endtabcontent %}
{% endinittab %}

## The Remote Result Reducer

When broadcasting remote invocations to all active cluster members, and the remote method returns a result, on the client side an array of all remote results needs to be processed. By default, the first result is returned, and it can be overriden using the `return-first-result` flag (which will then return the array of responses). The executor remoting proxy allows for a pluggable remote result reducer that can reduce a collection of remoting results into a single one. Here is the defined interface:

{% highlight java %}
public interface RemoteResultReducer<T, Y> {

    /**
     * Reduces a list of Space remoting invocation results to an Object value. Can use
     * the provided remote invocation to perform different reduce operations, for example
     * based on the {@link SpaceRemotingInvocation#getMethodName()}.
     *
     * <p>An exception thrown from the reduce operation will be propagated to the client.
     *
     * @param results            A list of results from (usually) broadcast sync remote invocation.
     * @param remotingInvocation The remote invocation
     * @return A reduced return value (to the calling client)
     * @throws Exception An exception that will be propagated to the client
     */
    T reduce(SpaceRemotingResult<Y>[] results, SpaceRemotingInvocation remotingInvocation) throws Exception;
}
{% endhighlight %}

# Failover

In case of a Primary Backup clusters and Partitioned Sync2Backup clusters, when there is failure of primary space partition, remoting request is transparently forwarded to the backup.

Executor proxy is aware of each partition in the cluster and connection with the failed primary partition will get interrupted/disconnected in case of a failure. Proxy recognizes failure and redirects the request to the backup partition. When the backup becomes ready (any warmup code that needs to run and is ready to accept operations), it will accept this request and process the request. All of this failover behavior is done within the GigaSpaces proxy code and client does not need any extra logic. For more information refer to [Proxy Connectivity](./proxy-connectivity.html).

If the backup space does not become ready to accept requests within the configured number of retries, execute request will fail with an Exception. In case of a broadcast request, `SpaceRemotingResult` for failed partition will have an exception. Following example shows how to inspect for exceptions in Reducer,

{% highlight java %}
public class DataProcessorServiceReducer implements RemoteResultReducer<Integer, Integer>{

	public Integer reduce(SpaceRemotingResult<Integer>[] results,
                               SpaceRemotingInvocation sri) throws Exception {
		int total_result =0;
		for (SpaceRemotingResult<Integer> result : results)
		{
			if (result.getException() == null) {
				int temp_result = result.getResult().intValue();
				total_result +=  temp_result ;
			} else {
				System.out.println("Exception on the Server: "
                                                   + result.getException().getMessage());
				result.getException().printStackTrace();
			}

		}
		return total_result/results.length  ;
	}
}
{% endhighlight %}
