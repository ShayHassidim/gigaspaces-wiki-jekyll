---
layout: post100
title:  JMS Integration - Security
categories: XAP100
parent: jms-advanced.html
weight: 300
---


To use the Enterprise Messaging Grid security facilities, you must specify a user name and password when you create the connection:

- Using the unified model:

{% highlight java %}
Connection connection = connectionFactory.createConnection(username,password);
{% endhighlight %}

- Using the point to point domain:

{% highlight java %}
QueueConnection queueConnection = queueConnectionFactory.createQueueConnection(username,password);
{% endhighlight %}

- Using the publish/subscribe domain:

{% highlight java %}
TopicConnection topicConnection = topicConnectionFactory.createTopicConnection(username,password);
{% endhighlight %}

{% info %}
To use authentication, you must define a secured space with the same user/password.
{%endinfo%}
