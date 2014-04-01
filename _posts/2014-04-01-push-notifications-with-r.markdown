---
layout: post
title:  "Push notifications from R"
date:   2014-04-01 20:10:39
categories: r
---

Have you ever stared at your computer screen waiting for a simulation to end? Have you ever forgotten to put a variable to track its progress? Mobile world can help you deal with this. Thus, if you have simulations running and want to know when they are finished or you may want updates on the counter which tracks its progress, you can use your phone to check on it while sipping coffee. All you need is `R`, `httr` library (`RCurl` would work as well) and a free account at [Pushbullet](pushbullet). Pushbullet for Android is also free which is cheaper than its competitor. Once you get a free account, you are automatically assigned an API key which you can find in your account settings.

Setting up notifications is painless. After you install the application on your mobile device, you need to find out your device ID. Just replace your API key with the one you are provided and run the following in `R`. 

{% highlight r %}
library(httr)
GET('https://api.pushbullet.com/api/devices',
    authenticate("McAG4V8p4izK35439e8SwWOJ822QHdWL",""))
{% endhighlight %}

From JSON-like output, just extract manually the device ID and you are done. You can now easily push notifications.

{% highlight r %}
api <- "https://api.pushbullet.com/api/pushes"
curl.opts <- list(device_iden='u1qSJddxeKwOGuGW', type='note',
                  title='Simulation is finished', body='Do a new one!')
POST(api, authenticate("McAG4V8p4izK35439e8SwWOJ822QHdWL", ""),
     body=curl.opts)
{% endhighlight %}

`POST` command will provide a lot of output by default. Just put `invisible` around it and it goes silent. As the above code is executed, the notification goes through and appears on the phone instantly.

[pushbullet]: https://www.pushbullet.com/
