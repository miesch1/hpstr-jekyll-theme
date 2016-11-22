---
layout: post
title: Active Directory Lookups using SIDs -- the Fast Way in C# (.NET 3.5+)
description: "How to lookup user accounts given a SID or Name."
tags: [Active Directory, C#, Windows, .NET]
categories: [Coding]
---

I was recently tasked with developing an application to easily add, remove and edit local standard user accounts in Windows. My goal was to replicate much of the functionality in the Local Account Management MMC snap-in, but in a more user-friendly and informative way. Imagine these screens combined, with other properties visible, such as bad logon count, last password set, password expiration date, etc.:

<img src="{{ site.url }}/images/Blog/2016/WindowsLocalAccountMangement.png" alt="Windows Local Account Mangement screenshot">

I started playing with the usual `DirectoryEntry` objects in the System.DirectoryServices namespace, but luckily I stumbled onto <a href="http://www.macaalay.com/2010/06/28/active-directory-c/" target="_blank">this excellent post</a> before I got too far down that path. I know this is old news, but it turns out .NET 3.5 introduced the new System.DirectoryServices.AccountManagement namespace to make managing user accounts and groups on a local machine and in Active Directory much simpler and faster. That namespace, along with the API usage examples from the aforementioned post, gave me a nice jump start into my project.

<!-- more -->

Everything was going peachy as I started implementing my own AccountManagement API. However, I was noticing some major delays in the execution of my methods to search for the Group and User principals that I am after. I didn't want to hard-code the user or group names, so I decided to try searching by SID rather than by user or group names. Unfortunately, this didn't improve the speed at all. Here are the methods I was using:

{% highlight cs %}
private static GroupPrincipal GetGroup(string accountSid)
{
  PrincipalContext oPrincipalContext = GetPrincipalContext();
  return GroupPrincipal.FindByIdentity(oPrincipalContext, IdentityType.Sid, accountSid);
}

private static UserPrincipal GetUser(string accountSid)
{
  PrincipalContext oPrincipalContext = GetPrincipalContext();
  return UserPrincipal.FindByIdentity(oPrincipalContext, IdentityType.Sid, accountSid);
}
{% endhighlight %}

On XP machines (don't ask!), each of the `FindByIdentity` calls above take 2-3 seconds to complete--long enough I didn't like it locking up my GUI thread. But the same calls in Windows 10 were taking over 20 seconds! Note that the required context for my calls is local machine (note that I found a post[^1] which indicated the delay may not be so bad if searching an Active Directory domain). It didn't take long to discover a plethora of recommendations to use a `PrincipalSearcher` rather than `FindByIdentity` for better performance.

[^1]: <http://stackoverflow.com/questions/7533790/findbyidentity-performance-differences>

For example, the following code[^2] executes well under a second (near instantaneous in the Debugger) in both XP and 10:

[^2]: <http://stackoverflow.com/a/14673947/3757184>

{% highlight cs %}
UserPrincipal oUserPrincipal = new UserPrincipal(oPrincipalContext);
oUserPrincipal.SamAccountName = userName;
return new PrincipalSearcher(oUserPrincipal).FindOne() as UserPrincipal;
{% endhighlight %}

Notice that in the code above I am back to searching for the `SamAccountName`. Unfortunately, the `Sid` member of the `User/GroupPrincipal` objects is readonly, thus that member can't be assigned as a query parameter--as others noted[^3]. Fortunately for us, the `PrincipalSearcher` class includes a `FindAll` method which can be used to return all User or Group principals in the provided context. Thus, with a bit of help from LINQ (and <a href=" http://stackoverflow.com/a/37725273/3757184" target="_blank">this answer</a>), we can easily do our own query to get the object associated with the desired SID. Here are my final implementations:

[^3]: <http://stackoverflow.com/questions/18852323/search-ad-only-having-sid>

{% highlight cs %}
private static GroupPrincipal GetGroup(string accountSid)
{
  PrincipalContext oPrincipalContext = GetPrincipalContext();
  GroupPrincipal oGroupPrincipal = new GroupPrincipal(oPrincipalContext);
  return new PrincipalSearcher(oGroupPrincipal).FindAll().Cast<GroupPrincipal>()
  .FirstOrDefault(x => x.Sid.Value.Equals(accountSid, StringComparison.OrdinalIgnoreCase));
}

private static UserPrincipal GetUser(string accountSid)
{
  PrincipalContext oPrincipalContext = GetPrincipalContext();
  UserPrincipal oUserPrincipal = new UserPrincipal(oPrincipalContext);
  return new PrincipalSearcher(oUserPrincipal).FindAll().Cast<UserPrincipal>()
  .FirstOrDefault(x => x.Sid.Value.Equals(accountSid, StringComparison.OrdinalIgnoreCase));
}

private static PrincipalContext GetPrincipalContext()
{
  return new PrincipalContext(ContextType.Machine, Environment.MachineName);
}
{% endhighlight %}

There, much better. And you can call them like this:

{% highlight cs %}
SecurityIdentifier adminsSid = new SecurityIdentifier(WellKnownSidType.BuiltinAdministratorsSid, null);
GroupPrincipal adminGroup = GetGroup(adminsSid.Value);

SecurityIdentifier guestSid = new SecurityIdentifier(WellKnownSidType.AccountGuestSid, null);
UserPrincipal guestUser = GetUser(adminSid.Value);
{% endhighlight %}

Again, this code executes fast enough that it's tempting to leave it on my GUI thread (I won't though!). Not sure what the deal is with `FindByIdentity` (at least on a local machine), but these calls should work for both local machine and on a domain.

As a side note, I ran into a several gotchas when calling members of the `UserPrincipal` objects. Thus, I still had to drop down to `DirectoryEntry` and even native `IADsUser` objects to get my final solution working right. But in the end, I could replace all of the functionality from the screenshot above and more. Maybe I'll throw my API on GitHub someday if there's any interest.