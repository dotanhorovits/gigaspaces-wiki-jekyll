---
layout: post
title:  Events and Messaging
categories: TUTORIALS
weight: 400
parent: java-home.html
---


{%comment%}
[< Previous|Tutorial Part III] * [Home|XAP Tutorial] * [Next >|Tutorial Part V] 
 {%endcomment%}

{%summary%}This part of the tutorial explains the different event and messaging services that are available over the space.{%endsummary%}
 

# Overview
{%section%}
{%column width=70% %}
The Space's Messaging and Events support provides messaging handlers that simplify event driven programming. Events are generated when objects are written, updated or taken from the space. With this framework you select events based on its content and designate a method that would be triggered as a result of that event, all through a simple and non-intrusive configuration. There are two main event handlers that are available:
{%endcolumn%}
{%column width=20% %}
<img src="/attachment_files/qsg/Events-Message.png" width="100" height="100">
{%endcolumn%}
{%endsection%}

# Notify Container
The Notify Container is the equivalent of a publish/subscribe semantics. An event listener is registered with the space that declares the type of events it is interested in. Upon receiving an event the listener is triggered. In this case all registered matching subscribers will be notified at the same time. 
 
 
# Polling Container
The Polling Container is the equivalent of a point to point paradigm. Unlike the notify container, the Polling Container blocks contentiously on space connection until a matching event arrives. Polling containers ensures that one and only one listener will be triggered per event even if there are more then one listener that matches that event. The entry will be removed from the space when the listener is invoked.

# Archive Container
The archive container is a mechanism built on top of a polling container to transfer historical data into Big-Data storage (for example Cassandra). The typical scenario is when streaming vast number of raw events through the Space, enriching them and then moving them to a Big-Data storage. Typically, there is no intention of keeping them in the space nor querying them in the space.

{%learn%}{%latestjavaurl%}/archive-container.html{%endlearn%}

 

# Example
Lets take our online processing service; we want to create a notification container that receives events when a payment has been cancelled. 
First, we define the Notifier Container.

{%highlight java %}
@EventDriven
@Notify
public class PaymentListener {

     @EventTemplate
     Payment unprocessedData() {
        Payment template = new Payment();
        template.setStatus(ETransactionStatus.CANCELLED);
        return template;
     }

    @SpaceDataEvent
    public Payment eventListener(Payment event) {
        // process Payment
        return null;
    }
}
{%endhighlight%}



In this example we define: 
{%indent%}
{: .table .table-bordered}
|Annotation | Description|
|:----------|:-----------|
|@EventDriven @Notify| the listener as a notification listener |
|@EventTemplate|       the event that will trigger the listener; a payment that was cancelled|
|@SpaceDataEvent|      the method that processes the event arrives. The return value is null, nothing will be written back into the space|
{%endindent%}


You can also define an event template using an SQLQuery.

{%highlight java %}
@EventTemplate
SQLQuery<Payment> unprocessedData() {
    SQLQuery<Payment> template = new SQLQuery<Payment>(Payment.class,"status = ?");
    template.setParameter(1, ETransactionStatus.CANCELLED);

    return template;
}
{%endhighlight%}


Now we are ready to register the container with the space:

{%highlight java %}
public void registerNotifierListener() {
     SimpleNotifyEventListenerContainer notifyEventListenerContainer = new SimpleNotifyContainerConfigurer(
	      space).eventListenerAnnotation(new PaymentListener())
	     .notifyContainer();
     notifyEventListenerContainer.start();
}
{% endhighlight %}

You can now execute a simple test that writes a Payment object into the space that will trigger the notification event:



{%highlight java %}
public void notifyTest() {
     Payment payment = new Payment();
     payment.setStatus(ETransactionStatus.CANCELLED);
     space.write(payment);
}
{%endhighlight%}


Lets assume we want to implement a polling container that receives audit notifications.

{%highlight java %}
@EventDriven
@Polling
@NotifyType(write = true, update = true)
public class AuditListener {
	@EventTemplate
	Payment unprocessedData() {
		Payment template = new Payment();
		template.setStatus(ETransactionStatus.AUDITED);
		return template;
	}

	@SpaceDataEvent
	public Payment eventListener(Payment event) {
		// process Payment
		System.out.println("Polling Received a payment:"+i);
		return null;
	}
}
{%endhighlight%}

By default all events will trigger the notification. In our example we are restricting the events to be received by using the {{@NotifyType}} annotation. In our example we are only interested in write and update events. 

{%learn%}{%latestjavaurl%}/event-processing.html{%endlearn%}

# FIFO Support
Sometimes it is necessary to process events in the order the way they have been created. By default events are not ordered. XAP supports FIFO (First In, First Out) processing of events.  To enable FIFO operations you can turn on FIFO support for classes which will participate in such operations.    

Here is an example how you can enable FIFO support for a class using the {{@SpaceClass}} annotation:

{%highlight java %}
@SpaceClass(fifoSupport=FifoSupport.OPERATION)
public class Payment implements Serializable {

	private static final long serialVersionUID = 1L;
	private String paymentId;
	private Integer payingAccountId;
}
{%endhighlight%}

{%learn%}{%latestjavaurl%}/fifo-support.html{%endlearn%}




 

# JMS
In addition to the polling containers you can also use a JMS facade on top of the space to deliver events. The JMS facade is designed to enable integration with external feeders that cannot or were not designed to work with the space based API. 

{%learn%}{%latestjavaurl%}/jms---basics.html{%endlearn%}

 


# Master Worker Pattern
{%section%}
{%column width=70% %}
The Master-Worker Pattern (sometimes called the Master-Slave or the Map-Reduce pattern) is used for parallel processing. It follows a simple approach that allows applications to perform simultaneous processing across multiple machines or processes via a {{Master}} and multiple {{Workers}}. 
In GigaSpaces XAP, you can implement the Master-Worker pattern using several methods:

- [Task Executors](/sbp/map-reduce-pattern---executors-example.html) - best for a scenario where the processing activity is collocated with the data (the data is stored within the same space as the tasks being executed).
- [Polling Containers]({%latestjavaurl%}/polling-container.html) - in this case the processing activity runs in a separate machine/VM from the space. This approach should be used when the processing activity consumes a relatively large amount of CPU and takes a large amount of time. It might also be relevant if the actual data required for the processing is not stored within the space, or the time it takes to retrieve the required data from the space is much shorter than the time it takes to complete the processing.

{%endcolumn%}
{%column width=20% %}

[<img src="/attachment_files/qsg/the_master_worker.jpg" width="200" height="200">](/attachment_files/qsg/the_master_worker.jpg)

{%endcolumn%}
{%endsection%}

{%learn%}/sbp/master-worker-pattern.html{%endlearn%}




<ul class="pager">
  <li class="previous"><a href="./java-tutorial-part3.html">&larr; Processing Services</a></li>
  <li class="next"><a href="./java-tutorial-part5.html">The Processing Unit &rarr;</a></li>
</ul>

{%comment%}
# What's Next

Part IV of this tutorial will introduce you to the concept of a Processing Unit which is the fundamental unit of deployment in GigaSpaces XAP.


!GS6:Images^Jump arrow green.bmp! {color:green}{*}Next step{*}{color} - [Part IV|Tutorial Part V] of this tutorial will introduce you to the concept of a Processing Unit which is the fundamental unit of deployment in GigaSpaces XAP.

# 
{align:center}[< Previous|Tutorial Part III] * [Home|XAP Tutorial] * [Next >|Tutorial Part V]{align}

{%endcomment%}