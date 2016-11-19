---
layout: post
title: Hello World!
subtitle: (i.e., "Official" Jekyll Support on Windows 10 WSL)
description: "A round-about intro to my blog and how to use Jekyll on Windows."
tags: [Jekyll, Windows 10, WSL]
categories: [Windows]
---

Hello friends, I'm here to tell you that you can now have your cake and eat it, too! I have been dabbling with Jekyll for a few months, trying to get this blog off the ground. I can now say I have found the most straightforward way to do it in Windows.

<img src="{{ site.url }}/images/Blog/2016/both worlds.png" alt="best of both worlds">

But before proceeding with my obligatory "Hello World" blog, some full disclosure. I am a solid Windows user--have been since Windows 95. Don't get me wrong, I've had a special place in my heart for Linux/Unix since college. I am all about open source and open architectures and my faith in Microsoft has at times been tested (Windows Vista, 8, death of Silverlight, anyone?). But their recent efforts with open sourcing .NET and the cross-platform .NET Core[^1] are very heartening, to say the least. I love C#, and if I can't run Visual Studio Enterprise[^2] on it, it's not my base OS (sorry, I am not a die-hard emacs or vi user!). These days, I mostly stick with Linux on my Raspberry Pis and occasionally on virtual machines.

[^1]: <https://www.microsoft.com/net/core>
[^2]: <https://2buntu.com/articles/1529/visual-studio-code-comes-to-linux/>

<!-- more -->

So if you've ever tried to run Jekyll on Windows, you can understand why it's not an officially supported platform[^3]. To be fair, this is a Ruby application that was never designed to cater to Windows. As such, it is non-trivial getting all the required tools and versions to play nicely in Windows. I started out with the <a href="http://jekyll-windows.juthilo.com/" target="_blank">de-facto excellent approach</a> to get Jekyll up and running on Windows. I'm sure if I had stuck with the versions listed by the author I would have been fine. But I tend to want the latest and greatest! After following the instructions and getting Ruby, the Ruby Dev Kit, and Jekyll and all other required gems installed, I was getting dependency errors when trying to serve up a default blog[^4] with Jekyll 3.2.1. I saw there had been a problem with Jekyll 3.2.0 on Windows, with a quick update to 3.2.1 to fix it[^5], so it seemed like this latest version should work. After several `gem update && gem cleanup` commands, I finally got it to serve up a default webpage without errors. However, the styles weren't applied and the pages didn't render correctly. Luckily I stumbled on an idea to try a previous version[^6] of Jekyll:

[^3]: <http://jekyllrb.com/docs/windows/>
[^4]: <http://jekyllrb.com/docs/quickstart/>
[^5]: <https://jekyllrb.com/news/2016/08/02/jekyll-3-2-1-released/>
[^6]: <http://stackoverflow.com/a/38699373>

{% highlight bash %}
gem uninstall jekyll -v 3.2.1
gem install jekyll -v 3.1.6
{% endhighlight %}

This worked! I then ran `gem update` and got back to Jekyll 3.2.1 and it continued to work. But it didn't sit well with me to have the tool so unstable on the same set of source files. Now, I'm obviously no Ruby or web developer. I just need something simple that allows me to easily spit out a static website to host on Github. At this point, I was ready to try another more Windows friendly HTML generation engine. But I kept gravitating back to Jekyll because of the amount of support it enjoys--that and it has the best theme variety!

I uninstalled everything and tried the other method listed on Jekyll's Windows page using Chocolately[^7] , as well as a portable Jekyll package[^8]--both with similar unsettling results. It all kind of seemed like a hack job. Around this time I considered using a Linux virtual machine to host a Jekyll installation, but it seems fate was on my side. I had just a day or two previously received the Windows 10 Anniversary update (version 1607). What does this have to do with the price of rice in China, you ask?

[^7]: <https://davidburela.wordpress.com/2015/11/28/easily-install-jekyll-on-windows-with-3-command-prompt-entries-and-chocolatey/>
[^8]: <http://www.madhur.co.in/blog/2013/07/20/buildportablejekyll.html>

Enter the Windows Subsystem for Linux (WSL)[^9].

[^9]: <https://blogs.msdn.microsoft.com/wsl/2016/07/08/bash-on-ubuntu-on-windows-10-anniversary-update/>

<a href="http://b192stech.xyz/blog/2016/06/16/bash-on-ubuntu-on-windows/"><img src="http://bhavin192.square7.ch/blog/wp-content/uploads/2016/06/WSL-featured.png" alt="WSL love"></a>

What this means is that we now can compile and run Linux binaries natively on Windows 10! Mind blown. Now this was something I had to play with. I want to mention that I didn't find any examples of specifically running Jekyll from WSL, but I did come across plenty of people worried about potential vulnerabilities exposed in general by enabling this feature. However, I found <a href="https://threatpost.com/windows-10-attack-surface-grows-with-linux-support-in-anniversary-update/119778/" target="_blank">this insightful article</a> which helped put my mind at ease. Anyway, your security is in your own hands so proceed at your own risk.

Enabling this feature in Windows was a breeze. Just enable it in Windows Features, run `bash` from a command prompt, then set your UNIX user name and password (my machine was already in Developer mode). See <a href="https://msdn.microsoft.com/en-us/commandline/wsl/install_guide" target="_blank">this excellent guide</a> for more details. 

<a href="{{ site.url }}/images/Blog/2016/Win10Bash.jpg"><img src="{{ site.url }}/images/Blog/2016/Win10Bash.jpg" alt="Bash on Win10" style="max-width:100%;"></a>

Now that is amazing! Ubuntu bash commands available on Windows, including `apt-get` to obtain Linux binaries. So now to install Ruby. I found some great tutorials on how to do this with RVM. I found the steps <a href="https://richonrails.com/articles/rails-on-windows-10-via-wsl" target="_blank">in this Ruby on Rails blog post</a> the most helpful. To summarize: 

{% highlight bash %}
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
\curl -sSL https://get.rvm.io | bash -s stable
echo "[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*" >> ~/.bashrc
source ~/.bashrc
rvm install 2.3.1
{% endhighlight %}

After running the previous command to install Ruby, my command line spit out some garbage text and appeared hung. Luckily, the Rails post mentioned above addressed this as well (thank you!):

>RVM will do some stuff, and ask you for your password. Enter the password you set up earlier, then push enter. You will likely be prompted 2-3 times to enter your password. The prompts might be hard to read (they were on my machine due to a weird terminal corruption glitch) so if it appears stuck, try typing your password and pressing enter.

That worked fine, then I could complete my installation of Ruby.

{% highlight bash %}
rvm use 2.3.1 --default
gem update --system
gem update
{% endhighlight %}

Awesome! On to installing Jekyll. Again, from the <a href="https://jekyllrb.com/docs/quickstart/" target="_blank">Jekyll Quick-start guide</a>:

{% highlight bash %}
gem install jekyll bundler
jekyll new myblog
{% endhighlight %}

Unfortunately, after the second command, I was greeted with the following:

{% highlight bash %}
...
  Bundler   1.13.1
  Rubygems  2.6.6
  Ruby      2.3.1p112 (2016-04-26 revision 54768) [x86_64-linux]
  GEM_HOME  /home/mike/.rvm/gems/ruby-2.3.1
  GEM_PATH  /home/mike/.rvm/gems/ruby-2.3.1:/home/mike/.rvm/gems/ruby-2.3.1@global
  RVM       1.27.0 (latest)
  Git       not installed
  rubygems-bundler (1.4.4)
--- TEMPLATE END ----------------------------------------------------------------

Unfortunately, an unexpected error occurred, and Bundler cannot continue.

First, try this link to see if there are any existing issue reports for this error:
https://github.com/bundler/bundler/search?q=parent+directory+is+world+writable+but+not+sticky&type=Issues

If there aren't any reports for this error yet, please create copy and paste the report template above into a new issue. Don't forget to anonymize any private data! The new issue form is located at:
https://github.com/bundler/bundler/issues/new
{% endhighlight %}

Again, the Rails post from above to the rescue!

>One gotcha to look out for, if you have an issue with bundler blowing up, running the following command should fix it:
{% highlight bash %}
chmod +t -R ~/.bundle
{% endhighlight %}

Which allowed me to complete the remaining <a href="https://jekyllrb.com/docs/quickstart/" target="_blank">Jekyll Quick-start guide</a> commands:

{% highlight bash %}
jekyll new myblog
cd myblog
bundle exec jekyll serve
{% endhighlight %}

Success! I browsed to `http://localhost:4000` and the default webpage was rendered correctly! However, I did have a warning about the file system watcher not being supported. After a bit of research, it became evident that WSL does not currently support  this feature, and the workaround is to use the no-watch argument. At the time of this writing, I am using Jekyll 3.3.3 and the following warning is exhibited:

{% highlight bash %}
bundle exec jekyll serve
...
      Generating...
                    done in 2.852 seconds.
                    --watch arg is unsupported on Windows.
                    If you are on Windows Bash, please see: https://github.com/Microsoft/BashOnWindows/issues/216
...
{% endhighlight %}

I don't really care about the real time re-generation, so I just serve my sites with the following command:

{% highlight bash %}
bundle exec jekyll serve --no-watch
{% endhighlight %}

One last tidbit from the Rails post from above (standard Linux but I include it here because it was an excellent reminder for me): 

>You may wish to create a symlink to your Documents folder in your Windows environment. To do so run the following command: `ln -s /mnt/c/Users//Documents ~/Documents`. Make sure to replace `` with your username.

Also, for reference see <a href="https://gorails.com/setup/windows/10" target="_blank">this this other thorough guide</a> I found, which includes alternative steps for installing Ruby.

So now I have a stable Jekyll build environment with easy access to/from my Windows OS. I would love to continue about my journey picking out this theme and modifying it to better fit my tastes. But this is long-winded enough, so that will have to wait for another post. All in all, the initial pain of setting up Jekyll on WSL was much less than it was directly on Windows. I have had no problems building this site with Jekyll in WSL and I would definitely recommend this approach if you are on Windows.