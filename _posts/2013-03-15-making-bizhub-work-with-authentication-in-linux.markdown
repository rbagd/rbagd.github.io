---
layout: post
title:  "Making Bizhub work with authentication in Linux"
date:   2013-03-15 12:00:00
categories: linux
---

Since my university department acquired a Konica Minolta Bizhub printer/scanner/copier and decided that authentication is needed for its daily usage, this has led to printing problems as Linux since default drivers weren't able to function anymore. There is very little information about this on the Internet and I spent some time to get it to work by trying to combine what I found on the net and by lots of pure testing. I hope that the following is gonna work for someone who is facing a similar issue, although this remains an ugly work-around.

I am using Ubuntu Linux myself but this would probably work in any Linux distribution with some minor adjustments and provided that printing is done through `CUPS`. [This page](https://blinky.si.umich.edu/confluence/pages/viewpage.action?pageId=9994335) was really useful to get me started but I was forced to make some changes so as to make the printer work. I will explain here in details the steps I used, some of which are just taken from the referenced web page.

First step is to download official Konica Minolta drivers and utilities for your machine. In my case it was the driver `KO423U.ppd` (it was for Bizhub 223) and the printing utility for UNIX, namely `kmpu1.8`. There's a deb file in `kmpu` which installs itself in `/opt/` folder and also installs some necessary filters for printing into your system. Once you installed the utility, place the downloaded `PPD` file for your machine in `/etc/lp/ppd/` folder. Also, you have to replace the default Konica Minolta filter by the one installed.

{% highlight bash %}
rm /usr/lib/cups/filter/kmfilter
cp /usr/lib/kmpu/printutility.filter.cups /usr/lib/cups/filter/kmfilter
chmod 755 /usr/lib/cups/filter/kmfilter
{% endhighlight %}

This finishes the installation of the necessary software and you can proceed. The next step is to set up the printer. When in `kmpu` directory, run the following commands

{% highlight bash %}
./kpappendprinter
./kpconfig -a
{% endhighlight %}

kpappendprinter command will ask you for 3 options: local print queue name (let's shorten it by lpqn), IP address of the printer and the port. I don't know if there's any way to guess these, I got all of them from a functioning setup on a MacOS. Recall that you can access this info in a Mac by pointing your browser to `http://localhost:631/printers` and enabling `CUPS` web-based interface if needed. In my case, port option was empty which I was unable to specify in `kpappendprinter` command and had to modify manually `/etc/cups/printer.conf` so as to delete the print part from `DeviceURI`.

Using `kpconfig` command is pretty self-explanatory, just be sure to pick the `PPD` file you downloaded previously and aimed for your printer model. Once this is finished, restart the `CUPS` service. In Ubuntu, that would be

{% highlight bash %}
service cups restart
{% endhighlight %}

Normally, you should be ready for printing. Try launching

{% highlight bash %}
kp -z readme.txt
{% endhighlight %}

`-z` option allows to see the full `lp` command used to print the file which is gonna be useful later and will help to get things done a little faster. If your printer uses authentication, you're gonna need to specify the necessary options such as the username, the password (or PIN) and the authentication server. Once again, I managed to get some of the information from a working MacOS installation and your login should be given to you by whoever set up the printer. In my case, the options were

{% highlight bash %}
Authentication type: Recipient User
Authentication username: JOHNDOE
Authentication password: 1234
Authentication server: Server1
{% endhighlight %}

You may also specify some additional printing options such as saving toner, double-sided printing, etc. That is up to you. Once you finish, a document should get printed and also a helpful `lp` command appears looking similar to

{% highlight bash %}
lp -d lpqn -o T=ka -ocopies=1 -oz -oJCLBOXHOLDTYPE='PrivateF' -oJCLKMUSERNAME='JOHNDOE'
 -oJCLKMUSERKEY='B6A48C4A87AF146C6910215G456271EC' 
 -oJCLKMCERTSERVTYPE='Server1' readme.txt
{% endhighlight %}

Afterwards, I got pretty frustrated since I still wasn't able to print neither `PS` or `PDF` files. After some investigation, I realized that the option `-o T=ka` specifies ASCII filter to be used for printing and somehow I couldn't manage to find how to change it using `kp`. But if you launch the `lp` command above without this filter option, you should be able to print `PS` files (e.g. `test.ps`).

My next step was to make printing work for PDF files and I am going to explain why that is useful very soon. In Ubuntu, I had to create two files. First, `/etc/cups/mime.types` containing the line

{% highlight bash %}
application/pdf pdf string(0,%PDF)
{% endhighlight %}

and `/etc/cups/mime.convs` containing the line

{% highlight bash %}
application/pdf application/postscript 33 pdftops
{% endhighlight %}

If these files already exist in your system, check that these lines are there and uncommented and it should be fine.

The reason why I wanted to make `PDF` printing work is because it's my main printing need but also because I was unable to print from any application with GUI even with correctly specified `lpoptions`. So I thought that to circumvent this problem, I'll just print everything to a `PDF` in a way that I like it and then send the generated `PDF` file to the Konica Minolta printer. This is what I did and it actually works for me. `Print to file` option works flawlessly and is available in probably most Gnome applications. I also had issues when dealing with large `PDF` files as they seemed to jam the printer quite often. However, I noticed that these problems rarely happen when printing a single page. Therefore, if you also encounter these problems, I suggest installing `pdftk` and splitting up your `PDF` into multiple files with a single page each before printing. You may use the following Nautilus script:

{% highlight bash %}
#!/bin/bash
pdftk $NAUTILUS_SCRIPT_SELECTED_URIS burst
exit 0
{% endhighlight %}

Place the script in `$HOME/.gnome2/nautilus-scripts` and don't forget to make it executable. Lastly, create a Nautilus script for printing which would contain:

{% highlight bash %}
#!/bin/bash
lp -d p2 -oz -oJCLBOXHOLDTYPE='PrivateF' -oJCLKMUSERNAME='JOHNDOE' \
 -oJCLKMUSERKEY='B6A28C4A87AF146C6930215F456291EC' -oJCLKMCERTSERVTYPE='Server1' \
 -oTonerSave='True' -oPageSize='A4' $NAUTILUS_SCRIPT_SELECTED_FILE_PATHS
exit 0
{% endhighlight %}
and with these two scripts printing gets as easy as a few right-clicks away.

This post was originally published [here](https://blogs.fsfe.org/rytis/2013/03/15/making-bizhub-work-with-authentication-in-linux/).
