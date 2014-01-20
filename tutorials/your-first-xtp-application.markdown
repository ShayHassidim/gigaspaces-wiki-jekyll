---
layout: post
title:  Your First XTP Application
categories: TUTORIALS
weight: 500
parent: xap-tutorials.html
---

{% trainingbox %}

{% compositionsetup %}

{% comment %}
=============================================================
         Summary
=============================================================
{% endcomment %}

**Summary:** {% excerpt %}This tutorial explains how to build your first GigaSpaces Application in 4 easy steps, from basic API usage to scaling your application and making it highly available{% endexcerpt %}

<iframe width="640" height="360" src="//www.youtube.com/embed/Q1zCFxqbqrQ?feature=player_embedded" frameborder="0" allowfullscreen></iframe>

{% comment %}
=============================================================
         Set Up Your Environment
=============================================================
{% endcomment %}

{% anchor Step1 %}

{% section %}

## Before You Begin - Set Up Your Environment

If you would like to run the tutorial sample application, [download GigaSpaces and set up your development environment](./setting-up-your-ide-to-work-with-gigaspaces.html).
The sample application used in all steps is located in ![folder.jpg](/attachment_files/folder.jpg) <GigaSpaces root>\examples\helloworld

{% comment %}
=============================================================
         Quick Start Guide Steps
=============================================================
{% endcomment %}

{% anchor Step2 %}

## Quick Start Guide Steps

We recommend that you follow these 4 basic tutorials in the specified order:
{% endsection %}

{% panel borderStyle=solid|borderColor=#3c78b5|bgColor=white %}

{% section %}

{% comment %}
---------------------------------------------------------------
         Using Processing Units for Scaling
---------------------------------------------------------------
{% endcomment %}

{% column width=7 %}
{% align center %}

![Step1.jpg](/attachment_files/Step1.jpg)
{% endalign %}

{% endcolumn %}
{% column width=35% %}
[**Using Processing Units for Scaling**](./step-one---using-processing-units-for-scaling.html)
{% color grey %}~5 minutes{% endcolor %}
A short introduction that shows what a Processing Unit is, and how it is used for scaling your applications

{% endcolumn %}
{% column width=50% %}
{% align center %}

[![What is a Processing Unit.jpg](/attachment_files/What is a Processing Unit.jpg)](./step-one---using-processing-units-for-scaling.html)
{% endalign %}

{% endcolumn %}
{% endsection %}

{% endpanel %}
{% panel borderStyle=solid|borderColor=#3c78b5|bgColor=white %}

{% section %}

{% comment %}
---------------------------------------------------------------
         Helloworld Tutorial
---------------------------------------------------------------
{% endcomment %}

{% column width=7 %}
{% align center %}

![Step2.jpg](/attachment_files/Step2.jpg)
{% endalign %}

{% endcolumn %}
{% column width=35% %}
[**Creating the Hello World Application**](./step-two---creating-the-hello-world-application.html)
{% color grey %}~10 minutes{% endcolor %}
How to create, deploy, run and monitor your Processing Unit

{% togglecloak id=2 %}  {% sub %}**In this tutorial you will learn...**{% endsub %}{% endtogglecloak %}
{% gcloak 2 %}

**.** {% sub %}How to Create a scalable application using Processing Units{% endsub %}
**.** {% sub %}How to run a Processing Unit within your IDE{% endsub %}

{% endgcloak %}
{% endcolumn %}
{% column width=50% %}
{% align center %}

![qsg_helloworld_processing_unit.gif](/attachment_files/qsg_helloworld_processing_unit.gif)
{% endalign %}

{% endcolumn %}
{% endsection %}

{% endpanel %}

{% panel borderStyle=solid|borderColor=#3c78b5|bgColor=white %}

{% section %}

{% comment %}
---------------------------------------------------------------
         Helloworld ServiceGrid Tutorial
---------------------------------------------------------------
{% endcomment %}

{% column width=7 %}
{% align center %}

![Step3.jpg](/attachment_files/Step3.jpg)
{% endalign %}

{% endcolumn %}
{% column width=35% %}
[**Deploying the Hello World Application onto the ServiceGrid**]({%currentjavaurl%}/deploying-onto-the-service-grid.html)
{% color grey %}~10 minutes{% endcolor %}
How to deploy the _Hello World Processing Unit_ onto the grid enabled infrastructure (the ServiceGrid) to enable instant fail-over, recovery, SLA management and runtime monitoring capabilities for your application

{% togglecloak id=3 %}  {% sub %}**In this tutorial you will learn...**{% endsub %}{% endtogglecloak %}
{% gcloak 3 %}

**.** {% sub %}What the Service Grid is{% endsub %}
**.** {% sub %}How to Start the Service Grid{% endsub %}
**.** ~How to Use the Grid Manager (GSM) and the Grid Container(GSC)~
**.** {% sub %}How to Deploy an application onto the Service Grid with the Management UI{% endsub %}
**.** {% sub %}How to Monitor the Service Grid and deployed applications during runtime{% endsub %}

{% endgcloak %}
{% endcolumn %}
{% column width=50% %}
{% align center %}

![qsg_service_grid.gif](/attachment_files/qsg_service_grid.gif)
{% endalign %}

{% endcolumn %}
{% endsection %}

{% endpanel %}
{% panel borderStyle=solid|borderColor=#3c78b5|bgColor=white %}

{% section %}

{% comment %}
---------------------------------------------------------------
         Helloworld Scaling Tutorial
---------------------------------------------------------------
{% endcomment %}

{% column width=7 %}
{% align center %}

![Step4.jpg](/attachment_files/Step4.jpg)
{% endalign %}

{% endcolumn %}
{% column width=35% %}
[**Scaling the Hello World Application**](./step-four---scaling-the-hello-world-application.html)
{% color grey %}~10 minutes{% endcolor %}
How to scale the _Hello World Processing Unit_ application

{% togglecloak id=4 %}  {% sub %}**In this tutorial you will learn...**{% endsub %}{% endtogglecloak %}
{% gcloak 4 %}

**.** {% sub %}How to use GigaSpaces' clustering and partitioning capabilities to scale the _Hello World Processing Unit_ application{% endsub %}

{% endgcloak %}

{% endcolumn %}
{% column width=50% %}
{% align left %}

![qsg_helloworld_scaling.gif](/attachment_files/qsg_helloworld_scaling.gif)
{% endalign %}

{% endcolumn %}
{% endsection %}

{% endpanel %}

{% comment %}
=============================================================
         Go Beyond the Basics
=============================================================
{% endcomment %}

{% section %}
## Go Beyond the Basics

After you have learned the basics, you can go to the [more advanced tutorials](./beyond-the-basics.html) to dive into the details, and learn how to implement real world scenarios.
{% refer %}[Back to Quick Start Guide Home](./index.html){% endrefer %}
{% endsection %}

