---
layout: post
title: ASP.NET Core on an ARMv6 Raspberry Pi
subtitle: The .NET Core Can't Help on a Pi Zero, But Mono Can...
description: "A tutorial on how to serve up ASP.NET Core websites using Mono 4.6.2."
tags: [Raspberry Pi, ASP.NET, ASP.NET Core, Mono]
categories: [Linux]
---

Sorry for the hiatus, but I'm back now. I am crazy excited about .NET Core and I think it is going in an awesome direction. We use it at work and I wish I had more Raspberry Pi 2 & 3s laying around since they are capable of running it. The problem is that I still have ARMv6 Pi 1s and Zeros that need to be put to use! I worked this out a few years ago on my Pi 1, but the latest version of Mono that I could get on my Pi at that time was 4.0.2. That version can run older ASP.NET 5 projects, but not ASP.NET Core. I will show you how to get Mono 4.6.2, libuv 1.22.0, and ASP.NET Core 2.0 to all play nicely together on a Pi Zero.

<img src="https://cdn.shopify.com/s/files/1/2187/3161/products/cba5e4e317b457b3ddf81ab1dc6b8c5e_1024x.png?v=1520446002" alt="PiZero">

<!-- more -->

First off, since I am using a Pi Zero, I needed either a USB Wifi dongle or an Ethernet adapter. I have used both but I prefer a wired connection since I plan to stream from this Pi in the future. <a href="https://www.amazon.com/gp/product/B00RM3KXAU/ref=oh_aui_detailpage_o04_s00?ie=UTF8&psc=1" target="_blank">This adapter</a> worked perfectly right out of the gate. Then make sure you grab the latest version of <a href="https://www.raspberrypi.org/downloads/raspbian/" target="_blank">Raspian Stretch</a> to image your Pi's SD card with. I used the Lite version since I am running headless. Remember that to communicate with the headless OS, enabling SSH is required. As directed,[^1] create a blank text file named "ssh" on the root of the Boot partition on the SD card. Further, if you are using WIFI, you need to create a text file named "wpa_supplicant.conf". Copy the following text in that file and update at a minimum your ssid and psk:

[^1]: <https://www.raspberrypi.org/forums/viewtopic.php?t=191252>

{% highlight bash %}
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="your_real_wifi_ssid"
    scan_ssid=1
    psk="your_real_password"
    key_mgmt=WPA-PSK
}
{% endhighlight %}

Now that we can bash, let's install Mono. This is where it used to be difficult. As I mentioned previously, when I first started playing around with ASP.NET, the default version of Mono on Rasbian was 3.2.8. I attempted to build Mono from source but ran into several problems, most likely because my ARMv6 Pis do not support hard floating point. At that time, I found two particularly helpful posts on how to point to a Jessie-specific Mono 4.0.2 repository which enabled me to run ASP.NET 5 applications.

<a href="http://www.hanselman.com/blog/HowToRunASPNET5Beta3OrGoLangOnARaspberryPi2.aspx" target="_blank">http://www.hanselman.com/blog/HowToRunASPNET5Beta3OrGoLangOnARaspberryPi2.aspx</a>
<a href="https://www.thedju.net/articles/asp-net-core-mono-linux" target="_blank">https://www.thedju.net/articles/asp-net-core-mono-linux</a>

Fast-forward and now Rasbian's Mono version is 4.6.2, so we can just install it! Easy-peasy:

{% highlight bash %}
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install mono-complete
{% endhighlight %}

Now check out your Mono version. We're all set!

{% highlight bash %}
pi@raspberrypi:~ $ mono --version
Mono JIT compiler version 4.6.2 (Debian 4.6.2.7+dfsg-1)
Copyright (C) 2002-2014 Novell, Inc, Xamarin Inc and Contributors. www.mono-project.com
        TLS:           normal
        SIGSEGV:       normal
        Notifications: epoll
        Architecture:  armel,vfp+hard
        Disabled:      none
        Misc:          softdebug
        LLVM:          supported, not enabled.
        GC:            sgen
{% endhighlight %}

At this point, it was important for me to ensure my old ASP.NET 5 application still worked. So I  tried installing DNVM as outlined by dave dot ninja <a href="http://dave.ninja/asp-net-5/asp-net-on-linux-part-2-installing-the-net-framework/" target="_blank">here</a>. I hit a roadblock at the first command:

{% highlight bash %}
curl -sSL https://raw.githubusercontent.com/aspnet/Home/dev/dnvminstall.sh | DNX_BRANCH=dev sh && source ~/.dnx/dnvm/dnvm.sh
{% endhighlight %}

Running the dnvminstall.sh produces a 404 Not Found error. Sure enough, the URL for dnvm.sh in that script no longer exists. I knew DNVM is deprecated, but obsolete or not, this is the only way to run ASP.NET 5 on a Pi Zero. I found <a href="https://github.com/aspnet/dnvm/issues/521" target="_blank">someone else</a> looking for this, as well as confirmation that I would have to reference it in commit-specific links. Going back to Scott Hanselman's post mentioned above, I modified his steps (his example was for the older KVM tool) to install the latest version of DNVM I could find (which happened to be tagged 1.0.1):

{% highlight bash %}
sudo apt-get install git
git clone --branch 1.0.1 https://github.com/aspnet/Home.git
cd Home/
sh dnvminstall.sh
{% endhighlight %}

At this point, we know the dnvm.sh script copied to /home/pi/.dnx/dnvm/dnvm.sh by dnvminstall.sh is invalid, so I just overwrote it with the dnvm.sh from the repository and continued with the installation. 

{% highlight bash %}
cp dnvm.sh /home/pi/.dnx/dnvm/dnvm.sh
source /home/pi/.dnx/dnvm/dnvm.sh
dnvm upgrade
dnvm list
{% endhighlight %}

You should see your mono runtime listed:

{% highlight bash %}
Active Version              Runtime Architecture OperatingSystem Alias
------ -------              ------- ------------ --------------- -----
  *    1.0.0-rc1-update2    mono                 linux/osx       default
{% endhighlight %}

As Scott further described, the "Kestrel" web server requires the libuv library. I modified his steps to build the current version of libuv. But trying to run "dnx run" yielded an error that Mono was unable to load DLL 'libuv'. Argh. I looked back at some notes and remembered that Mono is looking for the library in /usr/local/lib. Added the symbolic links to that folder instead but still no dice. I went to my Pi 1 and verified that his steps worked for Mono 4.0.2 with /usr/local/lib and libuv 1.9.1. Back to dave dot ninja's updated DNVM steps, I realized he had posted slightly different steps for building libuv. This worked perfectly (modified with updated version):

[^4]: <http://dave.ninja/asp-net-5/asp-net-on-linux-part-2-installing-the-net-framework/>

{% highlight bash %}
sudo apt-get install automake libtool curl
curl -sSL https://github.com/libuv/libuv/archive/v1.22.0.tar.gz | sudo tar zxfv - -C /usr/local/src
cd /usr/local/src/libuv-1.22.0
sudo sh autogen.sh
sudo ./configure
sudo make
sudo make install
sudo rm -rf /usr/local/src/libuv-1.22.0 && cd ~/
sudo ldconfig
{% endhighlight %}

Now let's see if we're up:

{% highlight bash %}
cd Home/samples/1.0.0-rc1-update1/HelloMvc/
dnu restore
dnx web
{% endhighlight %}

If you see this then you are there!

{% highlight bash %}
Hosting environment: Production
Now listening on: http://*:5004
Application started. Press Ctrl+C to shut down.
{% endhighlight %}

Browse to your Pi's IP address at the port listed above. After a minute or so you should see the sample ASP.NET 5 MVC website!

<img src="{{ site.url }}/images/Blog/2018/HelloMvc.png" alt="HelloMvc">

Awesome. Now you can create an ASP.NET Core project in Visual Studio and run it on your Pi 1 or Pi Zero. Here is a great tutorial on how to accomplish this:

<a href="https://www.c-sharpcorner.com/article/running-asp-net-core-2-0-via-mono/" target="_blank">https://www.c-sharpcorner.com/article/running-asp-net-core-2-0-via-mono/</a>

I am planning a follow-up post to show how to create a simple ASP.NET Core website that streams video from the Raspberry Pi camera.