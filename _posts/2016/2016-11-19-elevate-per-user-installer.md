---
layout: post
title: How to Elevate a Per User Installer Using WiX
description: "Was non-trivial for me to accomplish this."
tags: [WiX, installer]
categories: [Coding]
---

I am a huge fan of the open source WiX Toolset[^1] for authoring Windows installers. I've used Installshield and Visual Studio Setup projects for installers past. I even had to reverse engineer an old VB Package and Deployment Wizard installer for a customer. It only took me 16 hours to re-author it with WiX (and that included testing). Sure, sometimes it would be nice to have a GUI for spitting out the XML, but with the control WiX offers me there's no going back. If you've never experienced the power of WiX (including custom actions in C#), you are missing out!

[^1]: <http://wixtoolset.org/>

This week I ran into a situation where I needed to do something that I didn't think would be so hard to figure out.
1. I really needed a per-user install.
2. I really needed the installer to prompt for elevation.

<!-- more -->

I wanted a per-user install so that the application is only visible in Add/Remove Programs for the user who installed it. And my installer performed custom actions which required elevation and I didn't want to require my users to launch the MSI with `msiexec` from a command line with elevated privileges. Turns out that most people are looking for the opposite--a per-user install that doesn't prompt for elevation, like <a href="https://blogs.msdn.microsoft.com/astebner/2007/11/18/using-wix-3-0-to-create-a-per-user-msi-that-does-not-prompt-for-elevation-on-windows-vista/" target="_blank">this excellent post</a> details. So according to those instructions, I figured I would just leave ALLUSERS undefined and then set the opposite InstallPrivileges attribute, namely `Package/@InstallPrivileges="elevated"` (and `Package/@InstallScope="perUser"` for good measure).

With those settings, running my installer from a standard user account without admin rights was not prompting for elevation and my installer was failing. Crap. Back to Googleing, it was surprising how many posts <a href="http://stackoverflow.com/questions/9279713" target="_blank">like this one</a> I found which were answered only by saying that per-user installs should never require elevation. That may very well be the case, but I have a legitimate need to do this. Luckily, I found a post[^2] that offered the clue I needed: `Package/@InstallScope` doesn't  support per-user, elevated packages!

[^2]: <https://sourceforge.net/p/wix/mailman/message/26220559/>

All I had to do was omit the `Package/@InstallPrivileges` (defaults to elevated) and `Package/@InstallScope` attributes (continuing to leave ALLUSERS undefined). I also incorporated the trick I found from the MSDN blog above to prevent a user from setting ALLUSERS from an msiexec command line install. 

{% highlight xml %}
<!-- NOTE: If you need to create a per-user installation (meaning it's not -->
<!-- visible in Add/Remove Programs from other logons) that prompts for -->
<!-- elevation, omit both the Package/@InstallPrivileges="elevated" and
<!-- Package/@InstallScope="perUser". -->
<Package InstallerVersion="200" Compressed="yes" />

<!-- Set the "All Users" option. -->
<!-- NOTE: For a per-user installation, the value of ALLUSERS below must be empty -->
<!-- as well. This property cannot be set to empty, but it does default to empty. -->
<!-- Thus just leave it off. -->
<!-- <Property Id="ALLUSERS" Value="" />-->

<!-- This condition adds an item to the LaunchCondition table of the MSI to block a user -->
<!-- from setting  this property to something other than blank. -->
<Condition Message=”!(loc.LaunchCondition_AllUsers)”>
    NOT ALLUSERS
</Condition> 

<!-- This condition adds an item to the LaunchCondition table of the MSI to block a user -->
<!-- from installing this product unless they have administrative privileges on the system. -->
<Condition Message="You must have Administrative rights on this machine to install $(var.ProductName).">
        <![CDATA[Privileged]]>
</Condition>
{% endhighlight %}

There you go, problem solved.