---
layout: post97
title:  XAP.NET in 5 Minutes
categories: XAP97NET
parent: tutorials.html
weight: 100
---

{% summary  %}{% endsummary %}


{%comment%}
# Overview

In this tutorial we will:

1. Download and install **XAP.NET**.
2. Deploy a Data Grid.
3. Write code that connects to the Data Grid and interacts with it.
{%endcomment%}

This tutorial explains how to deploy and use an XAP [Data Grid](/product_overview/the-in-memory-data-grid.html) from a .NET client application.


{%comment%}
{% tip %} This tutorial contains code snippets in [C#](http://msdn.microsoft.com/en-us/library/kx37x362(v=vs.100).aspx), but naturally any other .NET language ([Visual Basic](http://msdn.microsoft.com/en-us/library/2x7h1hfk(v=vs.100).aspx), [C++/CLI](http://msdn.microsoft.com/en-us/library/xey702bw(v=vs.100).aspx), etc.) is supported as well.{%endtip%}

# Download and Install XAP

Getting XAP.NET is simple: download it from the [Current Releases](http://www.gigaspaces.com/LatestProductVersion) page.

{%info%} There are two XAP.NET packages, one for `x86` and another for `x64`. This tutorial refers to the `x86` package names and locations, but it fits the `x64` as well.{%endinfo%}
 
To install, simply double-click the `{{ site.latest_msi_filename }}` you've downloaded and follow the wizard to complete the installation. By default, the product will be installed into `C:\GigaSpaces\{{ site.latest_gshome_net_dirname }}`. 

# Deploy a Data Grid
{%endcomment%}

{%vbar title=Download and Install XAP%}
GigaSpaces XAP.NET is packaged as a standard Windows Installer package (.msi file). After you have downloaded the latest version of XAP from the [downloads page](http://www.gigaspaces.com/xap-download), start the installation by double-click the msi file, and the installation wizard will pop up and guide you through the installation process.

Once you accept the licence agreement, you will be asked to choose a setup type. Select 'Complete' to install all the features in the default path (C:\GigaSpaces\XAP.NET X.x.x). Selecting 'Custom' will allow you to customize the installation path, which features will be installed, and more.
{%endvbar%}

# Starting a Service Grid

A Data Grid requires a [Service Grid](/product_overview/service-grid.html) to host it. A service grid is composed of one or more machines (service grid nodes) running a [Service Grid Agent](/product_overview/service-grid.html#gsa) (or `GSA`), and provides a framework to deploy and monitor applications on those machines, in our case the Data Grid.

In this tutorial you'll launch a single node service grid on your machine. To start the service grid, simply run the `Gs-agent.exe` from the product's `bin` folder.

{% tip title=Optional - The Web Console %}
XAP provides a web-based tool for monitoring and management. From the `bin` folder start `Gs-webui.exe`, then browse to `http://localhost:8099`. Click the 'Login' button and take a look at the *Dashboard* and *Hosts* tabs - you'll see the service grid components created on your machine.
{% endtip %}

# Deploying the Data Grid

The Data grid can be deployed from command line, from the web management tool or via an Administration API. In this tutorial we'll use the command line.

Start a command line, navigate to the product's `bin` folder and run the following command:

{% highlight bash %}
gs-cli deploy-space -cluster total_members=2,1 myDataGrid
{% endhighlight %}
  
This command deploys a Data Grid (aka space) called **myDataGrid** with 2 partitions and 1 backup per partition (hence the `2,1` syntax). 

If you're using the web console mentioned above to see what's going on, you'll see the data grid has been deployed.
 
{%info%} Note that the Lite edition is limited to a single partition - if you're using it type `total_members=1,1` instead.{%endinfo%}

# Interacting with the Data Grid


### Setting up your IDE

Launch Visual Studio, create a new C# *Console Application* and add a reference to **GigaSpaces.Core.dll** from `C:\GigaSpaces\{{ site.latest_gshome_net_dirname }}\NET v4.0.30319\Bin`. If you're new to Visual Studio and .NET, follow these instructions:

{% togglecloak id=0 %}How to create a XAP.NET Project in Visual Studio{% endtogglecloak %}
{% gcloak 0 %}
1. Open Microsoft Visual Studio. From the **File** menu select **New > Project**. The **New Project** dialog appears.
2. In the **Project types** tree, select **Visual C#**, then select **Console Application** in the **Templates** list.
3. In the **Name** test box, type **XapDemo**. If you wish, change the default location to a path you prefer.
4. Select **OK** to continue. Visual Studio creates the project and opens the automatically generated `program.cs` file.
5. From the **Project** menu, select **Add Reference**. The **Add Reference** dialog appears.
6. Select the **Browse** tab, navigate to the XAP.NET installation folder (e.g. **C:\GigaSpaces\{{ site.latest_gshome_net_dirname }}\NET v4.0.30319**). Go into the **Bin** folder, select **GigaSpaces.Core.dll**, and click **OK**.
7. In the **Solution Explorer**, make sure you see **GigaSpaces.Core** in the project references. There's no need to reference any other assembly.

{% endgcloak %}

{%warning%} If you're using XAP.NET x64, note that the [default platform for Console Applications is x86](http://connect.microsoft.com/VisualStudio/feedback/details/455103/new-c-console-application-targets-x86-by-default), and you must [change it to x64](http://msdn.microsoft.com/en-us/library/ms185328.aspx).{%endwarning%}


### Connecting to the Grid

Since the Data grid is not located in our client process, we need some sort of address to find it. Data grids are searched using a `Space URL`, for example: `jini://*/*/myDataGrid?groups=$(XapNet.Groups)`. This roughly translates to: Find a remote space called `myDataGrid` (for more information see [SpaceURL](./the-space-configuration.html)).

Now that we have an address, we can connect to the grid:       

{% highlight csharp %}
ISpaceProxy spaceProxy = GigaSpacesFactory.FindSpace("jini://*/*/myDataGrid?groups=$(XapNet.Groups)");
{% endhighlight %}

The result is a `ISpaceProxy` instance, which is a proxy to the `myDataGrid` data grid. 

### Implementing a PONO

We now have a `ISpaceProxy` instance connected to our grid, which we can use to store and retrieve entries. But what shall we write? Actually, any PONO can be stored in the space, so long as it has a default constructor and an ID property. For this tutorial let's define a `Person` class with the following properties:

{% highlight csharp %}
using GigaSpaces.Core.Metadata;

public class Person
{
    [SpaceID]
    public int? Ssn {get; set;}
    public String FirstName {get; set;}
    public String LastName  {get; set;}
}
{% endhighlight %}

Note that we've annotated the `Ssn` property with a custom XAP.NET attribute (`[SpaceID]`) to mark it as the entries ID.

### Interacting with the grid

Now that we have a `ISpaceProxy` instance connected to our grid and a PONO which can be stored, we can store entries in the grid using the `Write()` method and read them using various `Read()` methods:

{% highlight csharp %}
Console.WriteLine("Write (store) a couple of entries in the data grid:");
spaceProxy.Write(new Person {Ssn=1, FirstName="Vincent", LastName="Chase"});
spaceProxy.Write(new Person {Ssn=2, FirstName="Johnny", LastName="Drama"});

Console.WriteLine("Read (retrieve) an entry from the grid by its id:");
Person result1 = spaceProxy.ReadById<Person>(1);

Console.WriteLine("Read an entry from the grid using LINQ:");
var query = from p in spaceProxy.Query<Person>()
            where p.FirstName == "Johnny"
            select p;
Person result2 = spaceProxy.Read<Person>(query.ToSpaceQuery());

Console.WriteLine("Read all entries of type Person from the grid:");
Person[] results = spaceProxy.ReadMultiple(new Person());
{% endhighlight %}

If you're using the web console mentioned above to see what's going on, you'll see two entries stored in the grid, one in each partition.

### {% anchor source %} Full Source Code

{% togglecloak id=1 %}`Program.cs`{% endtogglecloak %}
{% gcloak 1 %}
{% highlight csharp %}
using System;
using System.Linq;
using GigaSpaces.Core;
using GigaSpaces.Core.Linq;

namespace XapDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            String url = "jini://*/*/myDataGrid?groups=$(XapNet.Groups)";
            Console.WriteLine("Connecting to data grid " + url);
            ISpaceProxy spaceProxy = GigaSpacesFactory.FindSpace(url);

            Console.WriteLine("Write (store) a couple of entries in the data grid:");
            spaceProxy.Write(new Person { Ssn = 1, FirstName = "Vincent", LastName = "Chase" });
            spaceProxy.Write(new Person { Ssn = 2, FirstName = "Johnny", LastName = "Drama" });

            Console.WriteLine("Read (retrieve) an entry from the grid by its id:");
            Person result1 = spaceProxy.ReadById<Person>(1);
            Console.WriteLine("Result: " + result1);

            Console.WriteLine("Read an entry from the grid using LINQ:");
            var query = from p in spaceProxy.Query<Person>()
                        where p.FirstName == "Johnny"
                        select p;
            Person result2 = spaceProxy.Read<Person>(query.ToSpaceQuery());
            Console.WriteLine("Result: " + result2);

            Console.WriteLine("Read all entries of type Person from the grid:");
            Person[] results = spaceProxy.ReadMultiple(new Person());
            Console.WriteLine("Result: " + String.Join<Person>(",", results));

            Console.WriteLine("Demo completed - press ENTER to exit");
            Console.ReadLine();
        }
    }
}
{% endhighlight %}
{% endgcloak %}

{% togglecloak id=2 %}`Person.cs`{% endtogglecloak %}
{% gcloak 2 %}

{% highlight csharp %}
using System;
using GigaSpaces.Core.Metadata;

namespace XapDemo
{
    public class Person
    {
        [SpaceID(AutoGenerate=false)]
        public int? Ssn { get; set; }
        public String FirstName { get; set; }
        public String LastName { get; set; }

        public override string ToString()
        {
            return "Person #" + Ssn + ": " + FirstName + " " + LastName;
        }
    }
}
{% endhighlight %}
{% endgcloak %}

{%comment%}
# What's Next?

[The Full XAP .NET Tutorial](./net-home.html) will introduce you to the basic concepts and functionalities of XAP. Many ready to run examples are provided.

Read more about the GigaSpaces runtime environment, how to model your data in a clustered environment, and how to leverage the power capabilities of the Space.

- [Elastic Processing Unit]({%currentneturl%}/elastic-processing-unit.html)
- [Modeling and Accessing Your Data](/sbp/modeling-and-accessing-your-data.html)
- [Deploying and Interacting with the Space]({%currentjavaurl%}/deploying-and-interacting-with-the-space.html)
- [The GigaSpaces Runtime Environment]({%currentadmurl%}/the-runtime-environment.html)
{%endcomment%}
