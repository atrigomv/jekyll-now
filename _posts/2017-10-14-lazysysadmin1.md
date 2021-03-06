---
layout: post
title: Vulnhub LazySysAdmin 1
---

This pretend to be the walkthrough of LazySysAdmin: 1 vulnhub machine. As every penetration test, the first step was to make a ports and service discovery in order to detect misconfigured or vulnerable applications running on the server. As it's shown below, a web application is running on 80/tcp port which seems to be generated using Silex cms generator.

![evidence1]({{ site.baseurl }}/images/lazysysadminI_01.png)

That web application seems to be static-based, so we continue finding the way. If we use a web crawler tool, such as dirb or dirbuster on port 80/tcp, we can find differents directories that revelate the use of Wordpress cms and PHPMyAdmin service. The problem was that Wordpress instance seems to be updated and wp-scan didn't show us nothing interesting (no core or plugins-based exploits found).

![evidence2]({{ site.baseurl }}/images/lazysysadminI_02.png)

Trying to log into mysql from the attacker machine was impossible due to it's seems to be used a black lists and my IP was banned. So, in this point I was confuse just because I had an static web application running on port 80/tcp, a well-done bastioned Wordpress instance, an "impossible" MySQL service...so, what can I do? Try harder :) . I was deeper in the information-gathering step and I could perform a samba null session which gave me a valid username of the host, "togie" (the name of "togie" appear in some Wordpress pages as well).

![evidence3]({{ site.baseurl }}/images/lazysysadminI_03.png)

I tried to use this user within Wordpress and PHPMyAdmin private login form but it was unsuccessful. Instead of that, I performed a brute-force attack against ssh panel using [patator](https://github.com/az0ne/patator) tool and "unix_password" dictionary which is included within Kali distro.

![evidence4]({{ site.baseurl }}/images/lazysysadminI_04.png)

Bingo! It is possible to do login into the host using "togie" user and "12345" password through ssh service. Once I got a shell, the first step was discover the type of Unix distro deployed and the exactly kernel version used. Unfortunately, "togie" user had a restricted shell which didn't allow me execute most of unix commands so, the first step was try to evade that restriction. To do that, I detected that "togie" user was allowed to execute perl code, so executing the next line I could obtain an un-restricted shell:

```
perl -e 'exec "/bin/sh";'

```
When I got a handly shell, I discovered the next trouble which didn't allow me to perform a privilege escalation attack: the machine was not installed GCC compiler and I was not able to install it due to lack permission. So, I had three options: to looking for privilege escalation exploit for 4.4.0 kernel version written in python or perl, translate one of the exploits written in C language to python or perl or, the easier way, deploy a Ubuntu machine in my lab with the exact kernel version of the victim machine and try to compile in it one of the C language-based exploits. Although the first two options are obviously cooler and more "hacker style", I chose the last option and compile a version of [Dirty-COW attack](https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs) into a new machine. Then, I started an Apache service and, within the victim machine, I retrieved the binary using "wget" command. Finally, the Dirty-COW exploit was executed successfully, and I got a valid user called "firefart" with root privileges who I used to retrieved the root flag:

![evidence5]({{ site.baseurl }}/images/lazysysadminI_05.png)
