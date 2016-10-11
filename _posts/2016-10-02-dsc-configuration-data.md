---
layout: post
section-type: post
title: Making the most of DSC Configuration Data
category: tech
tags: [ 'DSC', 'PowerShell' ]
---

The intricacies of DSC Configuration Data can be somewhat tricky to understand at first but today I want to show you that once you have played around with it and created some example configurations it will all make sense.

Let’s connect some of the dots so it becomes crystal clear.

Here is a static DSC Configuration file that outlines some basic configuration for a fleet of Web Servers.

![powershell](/img/posts/2016-10-02-dsc-configuration-data/1.png)

As you can see we have a fair amount of code repetition here. However, this will still produce us our six mof files for our statically assigned nodes.

Let’s make this look better.

Configuration data is essentially a Hashtable (stored as a .psd1 file) that we can pass in at compile time that allows us to more dynamically create our mof files.

You will notice you have multiple templates all ready to go in the ISE (Ctrl + J) for Configuration Data and also Configuration:

![powershell](/img/posts/2016-10-02-dsc-configuration-data/2.png)

Lets Choose ‘DSC ConfigurationData’ and create our-self a .psd1 file.

![powershell](/img/posts/2016-10-02-dsc-configuration-data/3.png)

In our Configuration Data we’re going to pack it full of nodes and features that fall into the category of our role (in this case WebServer).

Now that we have our Configuration Data stored locally we can use Import-LocalizedData to parse our file to ensure its configuration. This can be used for creating Pester tests for Configuration Data and DSC In general.

![powershell](/img/posts/2016-10-02-dsc-configuration-data/4.png)

Now that we’ve our .psd1 file saved we can simply reference it from our Configuration script.

![powershell](/img/posts/2016-10-02-dsc-configuration-data/5.png)

As you can see this is making the most of DSC capabilities with Configuration Data to make our Configuration file smaller and more dynamic.

For more information on DSC and Configuration Data see the official documentation here.

Hopefully this helps shed some light on the ‘Why and How’ of Configuration Data and DSC.
