---
layout: post
title:  Troubleshooting Tools
categories: XAP97
parent: troubleshooting-for-administrators.html
weight: 300
---

{% summary page|60 %}Troubleshooting tools for testing a running product and environmental issues. {% endsummary %}

# CPU and Memory Dumper

## CPU Dumper Java Utility for Windows

The application starts a space and collects the memory and CPU utilization and dumps the threads when the CPU britches the configured CPU level.

- Extract the *[zip](/download_files/winJVMDumper.zip)* under `Gigaspaces root dir/bin`.
- Modify the config.dat and set the proper classpath, the CPU (in % beyond it the dump will be created) and the TIME (in msc of the time which lasts the application to breech beyond the configured CPU value)
- Run the GetJvmDump.exe

## A Script Utility for Linux

The [attached gsInstance.sh script](/download_files/gsInstanceDumper.zip) script starts a space using the regular gsInstance.sh equivalent and dumps the threads once the application breeches a certain CPU level for a certain period, set by the TIME value. The CPU and TIME can be passed as additional arguments to the script.
You can also simply copy the following section into any script:

{% highlight java %}
CPUUSAGE=$1
TIME=$2
if [ $# -lt  2 ] ; then
  echo  ""
  echo  "USAGE:    ./gsInstance.sh <CPU> <TIME>"
  echo  ""
  echo  "             <CPU> - CPU Utilization in percents (%)"
  echo  "             <TIME> - Time in seconds"
  echo  ""
  echo  "Example: ./gsInstance.sh 80 20 \"/./TestSpace\" "
  exit
fi

myPid=$$
chk=0
cpu=0
while [ 1 ]
do
   str=`ps --ppid $myPid --no-headers -o pcpu,rss,pid,cmd| grep java `
   cpu=`echo $str | awk '{print $1}'|cut -d. -f1`
   mem=`echo $str | awk '{print $2}'`

   echo "*********************************************"
   echo "***  CPU= $cpu %   MEMORY= $mem K"
   echo "*********************************************"
   if [ $cpu -gt  $CPUUSAGE ]; then
         chk=`expr $chk + 1`
   else
         chk=0
   fi
   if [ $chk -ge $TIME ];then
       pid=`ps --ppid $myPid  -o pid,pcpu,cmd | awk '{print $1}'`
       kill -3 $pid
   fi
   sleep 1
done
{% endhighlight %}

## Windows Debugging Tools

1. **userdump** - (correct version should be chosen - 32 or 64-bit according to the application) to get a dump
1. **[Debugging Tools for Windows](http://www.microsoft.com/whdc/DevTools/Debugging/default.mspx)** - to navigate in the dump. While userdump is by default non-intrusive and should not kill the application, it causes the application freeze which could be for a few seconds and it, eventually, could crash the application. So, it is NOT 100% safe.
Debugging Tools for Windows is GUI wrapper for debugging tools that could be used to simplify process identification, debugging, dump reading, it shows stack per each thread, etc.
Both tools are well tested and common in Windows environment.

1. **jstack Utility** - (part of JDK 1.6, including Windows) which shows thread stack and locking state per thread. It is non-disruptive and is very light with almost immediate execution (subsecond; tested), so it should not be a problem to run it even in pretty loaded environment
1. **Performance Monitor** - As trigger to shoot jstack when CPU goes up, built-in Windows XP Performance Monitor could be used (Control Panel/Administrative Tools/Performance). It has alerts on any of resource events, including CPU. Alert could start any program when chosen metric comes to the predefined value. It should start jstack-based script producing each time the file with the unique name (timestamp+). To avoid overloading the system with jstack runs, sampling interval could be set to 1min in alert definition.
1. **JRockit Mission Control** - When we'll get stacks of the process, likely, it will be not that easy to interpret it, as we should know what are the threads with most of activity. It could be done with no impact on performance with BEA JRockit "Mission Control" profiler, for example.

# Packet Sniffer/Network Analyzer Tool

{% infosign %} See [Packet Sniffer/Network Analyzer Tool](./how-to-configure-multicast.html#Packet) Sniffer/Network Analyzer Tool) section for more details.

# Threads Dumper

## SendSignal Utility

Java has this great feature where if you hit Ctrl-break on the console, it dumps a list of threads and all their held locks to stdout. If your app is stuck or overwhelmed with too many threads, you can figure out what it is doing. If it is deadlocked/overwhelmed, sometimes the JVM can even tell you exactly which threads are involved. The problem is that if you can't get to the console, you can't hit ctrl-break. This commonly happens to us when we are running under an IDE. The IDE captures stdout and there is no console. If the app deadlocks, there is no way to get the crucial debugging info. Another common usage scenario is once we do not have access to the console itself.

A very useful utility can be used for sending a Windows Ctrl+break signal to any process (or Java Process) and which will dump the threads.
The nice thing is that only the target process is affected, and any process (even a windowed process) can be targeted.

{% infosign %} Download the Send.Signal EXE file from the [project webpage](http://www.latenighthacking.com/projects/2003/sendSignal/)

Usage:

    SendSignal <pid>
      <pid> - send ctrl-break to process <pid> (hex ok)

# Log Correlation Tool

In many scenarios, mainly in production or large deployments, you many times face issues occurred in the same time over several components and cross several distributed machines. In order to track down what was the root cause of the event and which were just symptoms you need to review many log files and correlate what happened in that specific time.

In order to correlate events from different log files and visualize it you have to create a Log Correlation which is where the **[Eclipse Log and Trace Analyzer component \(TPTP LTA\)](/presentation_files/Eclipse%20TPTP%20-%20Log%20And%20Trace%20Analyzer.doc)** shines. This component is an extensive and extendable framework that includes built in probes mechanism.
