---
layout: single
title: "Cross-compile digikam 7.2.0 for Windows on Ubuntu 20.04"
date: 2021-04-18 18:37:00 +0200
categories: linux ubuntu
tags: linux ubuntu digikam compile
---

Digikam is an open source photo management tool. Each version is released for various operating systems, including Windows. However, at some point you may need to compile your own version, for example when you want to change the source code (more on that in a future article). In the following article I show you how to compile the latest digikam version 7.2.0 for Windows operating systems. You need a linux system for that and a decent machine. In my case I am running Ubuntu 20.04 in a VirtualBox virtual machine that uses 4 CPU cores (each having 3.7 to 4.6 GHz) and 16 GiB of RAM. You need approximately 21 GiB of free disk space.

{% include disclaimer.md %}

We start with installing some packages that we need for compiling digikam:
{% highlight bash %}
$ sudo apt install git make autoconf automake autopoint bison flex gperf libtool libtool-bin lzip ruby p7zip-full intltool python gcc g++ libssl-dev icoutils subversion nsis cmake qtbase5-dev curl
{% endhighlight %}

Now, go to your home directory and clone the digikam repository:
{% highlight bash %}
$ cd ~
$ git clone --progress --verbose --branch v7.2.0 --depth 1 https://invent.kde.org/graphics/digikam.git digikam-v7.2.0
Cloning into 'digikam-v7.2.0'...
POST git-upload-pack (180 bytes)
POST git-upload-pack (189 bytes)
remote: Enumerating objects: 6932, done.
remote: Counting objects: 100% (6932/6932), done.
remote: Compressing objects: 100% (5227/5227), done.
remote: Total 6932 (delta 2267), reused 4375 (delta 1628), pack-reused 0
Receiving objects: 100% (6932/6932), 244.63 MiB | 6.55 MiB/s, done.
Resolving deltas: 100% (2267/2267), done.
Note: switching to 'e98a50e68f76f630a7d1ae22c716324b9d1987a7'.

[output truncated]
{% endhighlight %}
Here, we checkout the branch `v7.2.0` and store all the data in the folder `digikam-v7.2.0`. Note that the folder name is important as you will see later.

Go to the folder `digikam-v7.2.0/project/bundles/mxe/` and start the first script:
{% highlight bash %}
$ cd digikam-v7.2.0/project/bundles/mxe/
$ ./01-build-mxe.sh
{% endhighlight %}
Now go and grab some beer/coffee (or just water in case you're underage) and watch some tv show (or do your homework in case you're a student) because this script will run for a while. It downloads and builds MXE, the thing that's used for cross-compiling. In my case the script took almost 3 hours.

When the first script succeeded it's time to run the second script that will build some more libraries:
{% highlight bash %}
$ ./02-build-extralibs.sh
{% endhighlight %}
Here, it took about 14 minutes to complete.

The next script will compile digikam itself. But before we run it, we need to adjust the configuration a little bit:
{% highlight bash %}
$ nano config.sh
{% endhighlight %}

Search for the text `DK_BUILDTEMP=~/dktemp` and change it to `DK_BUILDTEMP=~`.\
Moreover, change `DK_VERSION=master` to `DK_VERSION=v7.2.0`.\
Finally, change `DK_UPLOAD=1` to `DK_UPLOAD=0`.\
Save the file with `crtl+o`+`return` and exit nano with `ctrl+x`.

Here is an explanation of the changes: Usually, the script will download digikam's sources (or more specifically the branch specified by `DK_VERSION`) and store them in a folder based on `DK_BUILDTEMP` and `DK_VERSION`. However, when that folder already exists, it don't have to download the sources again. Do you remember that I said that the path `~/digikam-v7.2.0`, which we used while cloning the repository, is important?\
Since we already downloaded the code, we set `DK_BUILDTEMP` and `DK_VERSION` accordingly, such that it will use that same path and don't need to load the sources again.\
Using `DK_UPLOAD=0` we prevent any upload-attempt of the final build.

OK, using the config above we achieved that the next script will not download the sources again. But there is one more thing: when the script sees that the folder already exists, it will reset the working directory. Thus, you will loose all your changes. That includes the configuration made above and any changes to the sources that you may want to apply. So let's change the script a little bit:

{% highlight bash %}
$ nano 03-build-digikam.sh
{% endhighlight %}

Beginning with line 86 you will find the following:
{% highlight bash %}
# Build digiKam in temporary directory and installation

if [ -d "$DK_BUILDTEMP/digikam-$DK_VERSION" ] ; then

    echo "---------- Updating existing $DK_BUILDTEMP"

    cd "$DK_BUILDTEMP"
    cd digikam-$DK_VERSION

    git reset --hard
    git pull

    rm -fr po
    rm -fr build.mxe
    mkdir -p build.mxe

else

    # ...

fi
{% endhighlight %}

Add a `#` in front of the two `git` lines. Save the file using `ctrl+o`+`return` and exit nano with `ctrl+x`.

At this point it's a good idea to make some changes to the source code if you want, because the next command will compile digikam:
{% highlight bash %}
$ ./03-build-digikam.sh
{% endhighlight %}
You may watch another tv show episode (or finish your homework in case you're a student). In my case it took 42 minutes to complete.

As a final step, we create an installer package for our digikam build which will take another 12 minutes:

{% highlight bash %}
$ ./04-build-installer.sh
{% endhighlight %}

And here we go: inside `~/digikam-v7.2.0/project/bundles/mxe/bundle` you will find your installer, archive and checksum files:

{% highlight bash %}
$ ls -lah bundle/
total 217M
drwxrwxr-x  2 mkay mkay 4,0K May  5 10:40 .
drwxrwxr-x 12 mkay mkay 4,0K May  5 10:35 ..
-rw-rw-r--  1 mkay mkay 103M May  5 10:38 digiKam-7.2.0-Win64.exe
-rw-rw-r--  1 mkay mkay  192 May  5 10:40 digiKam-7.2.0-Win64.exe.sum
-rw-rw-r--  1 mkay mkay  289 May  5 10:40 digiKam-7.2.0-Win64.sum
-rw-rw-r--  1 mkay mkay 115M May  5 10:40 digiKam-7.2.0-Win64.tar.xz
-rw-rw-r--  1 mkay mkay  202 May  5 10:40 digiKam-7.2.0-Win64.tar.xz.sum
{% endhighlight %}

Now, go to your windows machine and install that thing :)
