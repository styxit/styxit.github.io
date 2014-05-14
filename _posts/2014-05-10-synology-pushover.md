---
layout: post
title: Synology Pushover notifications
description: Recieve Synology push notifications using Pushover.
---

![Synology logo](http://i.imgur.com/ltwAj3G.png){: .pull-right}
Recently I became the owner of a [Synology DS214+](http://www.synology.com/nl-nl/products/overview/DS214+). Since my HTPC was running out of space and consuming a lot of power, a Nas with lots of storage seemed right. It was all self explanatory and after some reseach i was able to get NZBget, Sickbeard, Couchpotato and [HTPC-Manager](http://htpc.io) running exacly the way i wanted. Next i wanted Pushover notifications!
<!-- more -->

In DSM5 (the Synology controlpanel) there are a lot of options to configure and play-around with. One of these nice options is notifications. After a specific event, you can recieve a notification. Event examples: `Backup to remote volume failed`,  `Overheating shutdown`, ` Disk I/O error` etcetera. Some of these things might be important to you so you want to know what's going on right away.

The notification methods Synology has buildt-in to notify you are; email, sms, msn, skype and phone push notifications using the DsFinder app for your phone. For me, the msn, skype and sms methods were not interesting. (I don't use skype or msn on my phone and I think sms is just out-dated) I want to recieve notifications on my phone via push. The Synology Android/IOS app can do this, but the fact that i need to install a specific app for it was a no-go.

At the moment [Pushover](http://pushover.net) is a popular application to recieve notifications on your mobile device(s) and can be used by many third party applications. I already have Sickbeard, Couchpotato and some work related stuff send messages to me through Pushover. I had really hoped Synology would also support this, but no.

## Time to get my hands dirty
![Pushover on android](/images/pushover-screenshot.png){: .pull-right}
I wanted to find a way to Let the Synology send my push messages to pushover. This quest started by looking into the synology code. As far as i know there is no sourcecode available so i had to do some searching. I ssh'ed into the synology and just looked at all the files is could find that looked like they had to do something with the notifications. I quit pretty soon. The source is all binaries and i could not really make anything of it, except for the webinterface code, but that was not what i was looking for.

The solotion i came up with was actually very easy, i just didn't look in the right place. It turned out, the SMS service was my saviour. In the `Notification -> SMS` section there is a button to add you own sms service. This is nothing more than an url with some parameters that recieve the notification message along with the login details for the sms service. The host for this sms service, including the parameters, can completely be customized. So i descided to write my own sms-service, but instead of sending me an sms, is sends me a Pushover notification.

## Do it yourself (configuration)

>If you do not have a Pushover account and/or the Pushover App installed on your device, this guide is probably not usefull to you.

I wrote a tiny php script that handles the http request from Synology and transfers it into a Pushover notification. 

### Register Pushover application
I assume you already have a pushover account and the pushover app installed on your mobile device. To get a Pushover Application API key, you need to create a new application at [pushover.net](pushover.net). Just login and "Register new application".

Most fields can be left blank, use `Synology` or `DiskStation` as the application title and upload an icon you like and hit "Create application" [I used this icon.](http://i.imgur.com/uPaCgDN.png)

Save the application/api Token, or leave this page open. You will need it later on.

### Install custom sms service in php
The script i wrote can be found as one of my [Github gists](https://gist.github.com/styxit/e34d4c6f8cb23d55f5af#file-synology-pushover-php). It needs Curl and it's recommended to run this on your synology webserver.

**1.** Enable the webserver for your synology if you haven't done so already. `ConfigPanel -> Web Services -> Enable webstation` This creates a new share called `web`, the root of your webserver.

**2.** Add my [synology-pushover.php script](https://gist.githubusercontent.com/styxit/e34d4c6f8cb23d55f5af/raw/0e63d5c69fc3a18c21a88d3251295785c01625ed/synology-pushover.php) to the root of your webserver. Make sure the filename is `synology-pushover.php`.

### Configure DSM notifications to custom server
Like i said, in DSM i used the sms service to recieve notificatons. You need to configure a custom sms service in DSM and point it to my script, running on the synology webserver. The Synology help documentation provides some information about how a sms service url should look like.

**1.** Add sms provider: `Control Panel -> Notification -> SMS`

**2.** Use `Local php` as the name and the following snippet for sms url. (make sure it starts with 'http' and ends in 'Hello+World')
{% highlight text %}
http://localhost/synology-pushover.php?userkey=username&appkey=pwd&to=1234&text=Hello+World
{% endhighlight %}

**3.** In the next step, you tell Synology what variables to use and what they mean. Use the following:
* userkey=username -> Username
* appkey=pwd -> Password
* to=1234 -> Phone number
* text=Hello+World -> Message content

You now created a custom sms provide, pointing to the pushover script at you local webserver.

**4.** Obtain your pushover api keys and enter these in the sms server username and password fields.
  - `Username` Your pushover user-key. Can be found on [https://pushover.net](https://pushover.net) (on the right).
  - `Password` The Pushover API Token for the Synology app you created.
  - `Confirm password` Again the Pushover API Token for the Synology app you created.

**5.** Since this was originally an sms provider, you must configure atleast a primary phone number. Although, it will not be used to send sms.

**6.** Disable the sms interval.

**7.** Click button to test. `Send a test SMS message` and check your Pushover to see the message.

If all went well, you have recieved a notification on your Pushover device.

In the advanced-tab you can configure wich event notifies which service. Check the sms box to enable/disable pushover for a spscific event. (I checked them all)

## Conclusion
Adding Pushover to Synology was not difficult. Maybe this is something for the next DSM update for the people of Synology to look at.

I am aware of the pushover email app. This lets you recieve notifications by emailing them to a custom pushover email address. That way is much easier, but does not provide a custom app-title, icon and if you have other apps emailing you, these will all be listed in the same group on your phone, not able to distinguish the synology notifications from other messages.

If you found this article usefull or think there is missing something, please let me know.
