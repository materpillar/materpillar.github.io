---
layout: post
title:  "The Revival of my Linkstation Pro with Debian 9.22"
date:   2018-11-26 16:21:41 +0200
categories: project
---

* Plug in empty disk into Linkstation
* Start the Linkstation. It will beep high-low to show it can't find boot files.
* Use TFTP package from Buffalo to install orginial uImage.buffalo and initrd.buffalo
* Use Firmware Updater 1.15 to install original firmware and make sure to enable Debug options and activating update boot
* Pull Debian 9 package from Website to Diskstation (in share folder for example)
* use telnet to copy config-debian to /boot


{% highlight bash %}
run sh config-debian
{% endhighlight %}

* mv uImage.buffalo -> uImage.buffalo.tmp
* mv uinitrd.buffalo -> initrd.buffalo.tmp
* copy debians uImage und initrd.buffalo into boot
* reboot
* ssh to Linkstation and install



{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/