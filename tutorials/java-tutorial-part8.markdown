---
layout: post
title:  Web Processing Unit
categories: TUTORIALS
weight: 800
parent: java-home.html
---



{%comment%}

[< Previous|Tutorial Part VII] * [Home|XAP Tutorial] * [Next >|Tutorial Part IX]  
{%endcomment%}


 
{%summary%}This part of the tutorial will introduce you to web deployment on top of the grid.{%endsummary%}

# Overview

{%section%}
{%column width=70% %}

In this part of the tutorial we will show you how you can deploy a standard WAR file onto the Service Grid. XAP allows you to deploy web applications (packaged as a WAR file) onto the Service Grid. The integration is built on top of the Service Grid Processing Unit Container. The web container used behind the scenes is Jetty. 

{%endcolumn%}
{%column width=20%%}
<img src="/attachment_files/qsg/web.png" width="100" height="100">

{%endcolumn%}
{%endsection%}

The integration allows you to make use of the following Service Grid features:

- Dynamic allocation of several instances of a web application (probably fronted by a load balancer).
- Management of the instances running (if a GSC fails, the web application instances running on it will be instantiated on a different GSC).
- SLA monitor based dynamic allocation and de-allocation of web application instances.



# Deployment
The web application itself is a pure, JEE based, web application. The application can be the most generic web application, and automatically make use of the Service Grid features.
 
Here is an example how you can deploy a web application on top of the service grid. We can accomplish this in two ways; by command line or with the admin UI.

### Command line deployment
You deploy a war file just like you deploy a pu jar.

{% inittab d1 |top %}
{% tabcontent Unix %}
{%highlight java%}
GS_HOME/bin/gs.sh deploy tutorial.war
{%endhighlight%}
{% endtabcontent %}
{% tabcontent Windows %}
{%highlight java%}
GS_HOME\bin\gs.sh deploy tutorial.war
{%endhighlight%}
{% endtabcontent %}
{%endinittab%}

### Deploy with Web Admin UI
You can deploy a war file just like a PU with the Web Administration UI:

{%section%}
{%column%}
Host view

[<img src="/attachment_files/qsg/Deploy-web1.png" width="120" height="80">](/attachment_files/qsg/Deploy-web1.png)
{%endcolumn%}

{%column%}
Upload war file

[<img src="/attachment_files/qsg/Deploy-web2.png" width="120" height="80">](/attachment_files/qsg/Deploy-web2.png)
{%endcolumn%}
 
{%column%}
Application display

[<img src="/attachment_files/qsg/Deploy-web3.png" width="120" height="80">](/attachment_files/qsg/Deploy-web3.png)
{%endcolumn%}

{%column%}
Service display

[<img src="/attachment_files/qsg/Deploy-web4.png" width="120" height="80">](/attachment_files/qsg/Deploy-web4.png)
{%endcolumn%}

{%column%}
Web page display

[<img src="/attachment_files/qsg/Deploy-web5.png" width="120" height="80">](/attachment_files/qsg/Deploy-web5.png)
{%endcolumn%}
{%endsection%}

There are fully functional examples on GitHub available.

{%try%}https://github.com/Gigaspaces/xap-tutorial{%endtry%}



### Using a Space
The web application can define a space either embedded or remote using Spring or direct in java code. There are several ways how a space and other components can be used and configured within a web application. Here are some scenarios:

- Creating a space using java code:
{%highlight java%}
UrlSpaceConfigurer spaceConfigurer = new UrlSpaceConfigurer("/./xapTutorialSpace");

GigaSpace gigaSpace = new GigaSpaceConfigurer(spaceConfigurer).gigaSpace();
{%endhighlight%}


- Using PU.xml to create a space:
A web application is still just a processing unit. This means that a META-INF/spring/pu.xml can be added, where we define the standard XAP components. 

Here is the layout of the war file structure:
{%highlight java%}
|----META-INF
|--------spring
|------------pu.xml
|------------pu.properties
|------------sla.xml
|--------MANIFEST.MF
|----WEB-INF
|--------lib
|--------web.xml
|----index.jsp
|---- .jsp
{%endhighlight%}

For example in the pu.xml file we can define our space that we want to be accessible from the web application.
{%highlight xml%}
<os-core:space id="space" url="/./xapTutorialSpace" />
<os-core:giga-space id="xapTutorialSpace" space="space"/>
{%endhighlight%}


Here is an example of a simple JSP that uses it:
{%highlight java%}
GigaSpace gigaSpace = (GigaSpace) getServletContext().getAttribute("xapTutorialSpace");
{%endhighlight%}


{%learn%}{%latestjavaurl%}/web-processing-unit-container.html{%endlearn%}


# Jetty Instance
{%section%}
{%column width=70% %}

Jetty itself is configured using Spring, and allows you to control all aspects of both the Jetty instance created, and the web application context. There are two flavors of how Jetty instances are created (by default). The first is the plain mode, where a Jetty instance is created for each web processing unit instance running within a GSC. The second is the shared mode, where a single Jetty instance is created, and shared between all the different web processing unit instances running on the same GSC. A custom Jetty instantiation and handling can also be configured.
{%endcolumn%}
{%column width=20% %}

[<img src="/attachment_files/qsg/web_app_archi.jpg" width="200" height="200">](/attachment_files/qsg/web_app_archi.jpg)

{%endcolumn%}
{%endsection%}

{%learn%}{%latestjavaurl%}/web-processing-unit-container.html{%endlearn%}






# HTTP Session management
The XAP-Jetty integration comes with a support for storing the javax.servlet.http.HttpSession in the Space allowing supporting session replication and failover between different web application instances deployed into GigaSpaces containers.

The GigaSpaces HTTP Session Management provides the following:

- Avoid using database as the persistent storage media - Better performance and simplify the deployment
- Allow management and control of full lifecycle of web application - From the load-balancer to the data
- Dynamic scalability - Allows the web application to scale up or down based on SLA
- Continuous High-Availability - Allows the application to survive any system failures
- Non intrusive session management or explicit Data Grid API access for fine grain control - Allows multiple web servers to share the same session in transparent manner





### Enabling HTTP Session Management
There are several ways to configure Space session based support, depending on the definition of which space and how the space was started.

For example, if the session will be stored on a remote clustered space with a local cache, the URL can be: jini://*/*/sessionSpace?useLocalCache (assuming the name of the space is sessionSpace). If the session should be stored on a space that should be started within the web application.

- Using Deploy property
Here is how you can deploy an existing WAR file without changing it (or use Spring) to use XAP's HTTP Session Management:
{%highlight java%}
gs deploy -properties embed://jetty.sessions.spaceUrl=jini://*/*/sessionSpace?useLocalCache xapTutorialSpace.war
{%endhighlight%}
In this example we connect to a remote space called sessionSpace and create a local cache for the session caching.

- Using Spring
When deploying, include within the META-INF/spring/pu.properties file the jetty.sessions.spaceUrl property value. This should be set with the space url defining where the sessions will be stored.

### Deploying with the Web Admin UI:

There are fully functional examples on GitHub available.{%try%}https://github.com/Gigaspaces/xap-tutorial{%endtry%}


{%section%}
{%column%}
Deploy http space

[<img src="/attachment_files/qsg/Deploy-session1.png" width="120" height="100">](/attachment_files/qsg/Deploy-session1.png)
{%endcolumn%}

{%column%}
Define deployment

[<img src="/attachment_files/qsg/Deploy-session2.png" width="120" height="100">](/attachment_files/qsg/Deploy-session2.png)
{%endcolumn%}

{%column%}
Inspect data grid

[<img src="/attachment_files/qsg/Deploy-session3.png" width="120" height="100">](/attachment_files/qsg/Deploy-session3.png)
{%endcolumn%}

{%column%}
Deploy war file

[<img src="/attachment_files/qsg/Deploy-session4.png" width="120" height="100">](/attachment_files/qsg/Deploy-session4.png)
{%endcolumn%}
{%endsection%}

{%section%}
{%column%}
Select web URL

[<img src="/attachment_files/qsg/Deploy-session51.png" width="120" height="100">](/attachment_files/qsg/Deploy-session51.png)
{%endcolumn%}

{%column%}
Web page

[<img src="/attachment_files/qsg/Deploy-session6.png" width="120" height="100">](/attachment_files/qsg/Deploy-session6.png)

{%endcolumn%}

{%column%}
Inspect data grid

[<img src="/attachment_files/qsg/Deploy-session65.png" width="120" height="100">](/attachment_files/qsg/Deploy-session65.png)

{%endcolumn%}

{%column%}
Inspect session

[<img src="/attachment_files/qsg/Deploy-session7.png" width="120" height="100">](/attachment_files/qsg/Deploy-session7.png)
{%endcolumn%}
{%endsection%}


{%learn%}{%latestjavaurl%}/http-session-management.html{%endlearn%}


# Global HTTP Session Sharing
{%section%}
{%column width=70% %}

XAP lets you share HTTP session data across multiple data centers, multiple web server instances or different types of web servers. Here are few scenarios where HTTP session sharing is required:

- Multiple different Web servers running your web application
  You may be porting your application from one web server to another and there will be a period of time when both types of servers need to be active in production.
- Web Application is broken into multiple modules
  When applications are modularized such that different functionalities are deployed across multiple server instances. For example, you may have login, basic info, check-in and shopping functionalities split into separate modules and deployed individually across different servers for manageability or scalability. In order for the user to be presented with a single, seamless, and transparent application, session data needs to be shared between all the servers.
- Reduce Web application memory footprint
  The web application storing all session within the web application process heap, consuming large amount of memory. Having the session stored within a remote process will reduce web application utilization avoiding garbage collocation and long pauses.
- Multiple Data-Center deployment
  You may need to deploy your application across multiple data centers for high-availability, scalability or flexibility, so session data will need to be replicated

{%endcolumn%}
{%column width=20% %}

[<img src="/attachment_files/qsg/httpSessionSharing1.png" width="200" height="200">](/attachment_files/qsg/httpSessionSharing1.png)

{%endcolumn%}
{%endsection%}

{%learn%}{%latestjavaurl%}/global-http-session-sharing.html{%endlearn%}


# Load Balancing
{%section%}
{%column width=70% %}

When deploying a highly available web site, usually a load balancer is used to load balance requests between at least two instances of a web containers that run the web applications. When using XAP to deploy web applications, running more than one instance of a web application becomes very easy. XAP comes with a built-in utility allowing you to dynamically update an Apache httpd web server load-balancing configuration, based on deployed web applications.

{%endcolumn%}
{%column width=20% %}

[<img src="/attachment_files/qsg/httpd_lb_agent.jpg" width="200" height="200">](/attachment_files/qsg/httpd_lb_agent.jpg)

{%endcolumn%}
{%endsection%}

The integration dynamically creates and updates the mod_proxy_balancer configuration, based on the state of the deployed web applications. Once changes occur (relocation / failover / changes to the number of web application instances), the utility identifies the change, updates the balancer configuration, and sends a soft restart to Apache to take the new configuration into account.

 
 
{%learn%}{%latestjavaurl%}/apache-load-balancer-agent.html{%endlearn%}



<ul class="pager">
  <li class="previous"><a href="./java-tutorial-part7.html">&larr; Space Persistency</a></li>
  <li class="next"><a href="./java-tutorial-part9.html">Big Data &rarr;</a></li>
</ul>

{%comment%}
# What's next

Part IX of this tutorial will introduce you to XAP's Big Data integration.

 # What's Next
!GS6:Images^Jump arrow green.bmp! {color:green}{*}Next step{*}{color} - [Part IX|Tutorial Part IX] of this tutorial will introduce you to XAP's Big Data integration.


# 
{align:center}[< Previous|Tutorial Part VII] * [Home|XAP Tutorial] * [Next >|Tutorial Part IX] {align}
{%endcomment%}

 

