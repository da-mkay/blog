---
layout: single
title: "Compile pure-ftpd v1.0.49 on Ubuntu 18.04"
date: 2021-04-16 19:24:21 +0200
categories: linux ubuntu
tags: linux ubuntu pure-ftpd compile
---

I was running a Ubuntu 16.04 server that uses pure-ftpd as FTP server. Since Ubuntu 16.04 will no longer be supported by April 30th, it was time to upgrade to Ubuntu 18.04. That went pretty flawlessly, except that I was no longer able to connect to pure-ftpd using the FileZilla FTP client. I got the following error message:

{% highlight plaintext %}
GnuTLS error -110 in gnutls_record_recv: The TLS connection was non-properly terminated
{% endhighlight %}

This is caused by a bug in pure-ftpd which is already fixed in recent versions, but the package in Ubuntu 18.04 is too old (v1.0.46). One solution would be to upgrade to Ubuntu 20.04, because it ships with pure-ftpd v1.0.49 where the bug is fixed. However, I am running an old 32-bit server machine and Ubuntu dropped support for the 32-bit architecture.

So the only way for me was to compile a recent version of pure-ftpd on my own. But I wanted to keep the way I can configure pure-ftpd on Ubuntu systems, i.e. by adjusting configuration files in `/etc/pure-ftpd/conf`, because pure-ftpd itself does not come with configuration scripts (it's configured by startup arguments only). My solution now is to take the pure-ftpd source package of Ubuntu 20.04 (Focal) and compile it on my 32-bit Ubuntu 18.04 (Bionic) machine. The resulting deb files can then be installed as usual and your existing configuration (if you already have one) just works ;-) Here is what I did:

{% include disclaimer.md %}

First, temporarily add the source repository of Ubuntu 20.04 (Focal) to your sources.list:
{% highlight bash %}
$ echo 'deb-src http://de.archive.ubuntu.com/ubuntu/ focal universe' | sudo tee -a /etc/apt/sources.list
$ sudo apt update
{% endhighlight %}

Then create a working directory and change to it:
{% highlight bash %}
$ mkdir pureftpd
$ cd pureftpd
{% endhighlight %}

Now, load the source package of pure-ftpd, currently v1.0.49:
{% highlight bash %}
$ apt source pure-ftpd
$ cd pure-ftpd-1.0.49
{% endhighlight %}

Install the required build dependencies:
{% highlight bash %}
$ sudo apt build-dep ./
{% endhighlight %}

Compile pure-ftpd and create deb packages:
{% highlight bash %}
$ dpkg-buildpackage -rfakeroot -uc -b
{% endhighlight %}

Make sure there are no pure-ftpd packages installed:
{% highlight bash %}
$ sudo apt remove pure-ftpd pure-ftpd-common
{% endhighlight %}

Then install the new packages:
{% highlight bash %}
$ cd ..
$ sudo dpkg -i pure-ftpd-common_1.0.49-4_all.deb pure-ftpd_1.0.49-4_i386.deb
{% endhighlight %}

Don't forget to revert your sources.list file (comment out or remove the last line which should be the one added in the first step):
{% highlight bash %}
$ sudo nano /etc/apt/sources.list
{% endhighlight %}
Save the file using `ctrl+o`+`return` and exit nano with `ctrl+x`.
