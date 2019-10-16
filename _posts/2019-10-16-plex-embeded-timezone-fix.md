---
layout: post
title: Timezone fix for Plex on Raspberry Pi
description: Correct the timezone on embedded Plex Media Player for a Raspberry Pi.
---
Plex Media Player (or PMP) for Raspberry Pi is an easy and light weight way to get Plex Media Player running on a Pi. Besides flashing the image to an SD card it requires no setup.

A default timezone is configured but the interface provides no (easy) way to change this. This is how you fix it in a few minutes.
<!-- more -->

On a regular basis there are new images published for the [nightly Plex Media Player for embedded systems](https://nightlies.plex.tv/public-test/plexmediaplayer/). After flashing a new image, i keep forgetting how to change the timezone. This article reminds me how to do it.

There are a lot of timezones available on the image, it just requires some adjustments to permanently use them.

## SSH into Plex Media Player on the Pi
Start off by finding the IP address of the Pi. The Plex interface does not show this so you will have to scan your network or find it by going into your router and checking for the connected devices.

I can not explain this for every router make and model, so i assume you are able to figure this out.

Now that you have the IP of the Pi, SSH into it.

The default username is `root` and the password is `plex`.

```
ssh root@IP_HERE
```

## Figure out your timezone
Next, figure out which timezones are available and pick yours.

Go into the "zoneinfo" directory and list all available regions:
```
cd /usr/share/zoneinfo && ls -ld */
```

Pick a region and list all the available timezones in that region. In this case i will pick `Europe`.
```
ls -la Europe
```

That will show you a list of available timezones in (in this case) Europe. It will look something like this.
```
drwxr-xr-x    2            976 Jun 25 18:18 .
drwxrwxr-x   18            981 Jun 25 18:18 ..
-rw-r--r--    1           2949 Jun 25 18:18 Amsterdam
-rw-r--r--    1           1751 Jun 25 18:18 Andorra
-rw-r--r--    1           1197 Jun 25 18:18 Astrakhan
-rw-r--r--    1           2271 Jun 25 18:18 Athens
-rw-r--r--    1           3687 Jun 25 18:18 Belfast
-rw-r--r--    1           1957 Jun 25 18:18 Belgrade
...
```

Choose one and remember it's exact name. I will pick `Amsterdam`.

## Changing the timezone on boot
With the region and timezone picked, a startup script must be created. This will execute on every boot so that the defined timezone is also available after a reboot or power loss.

Create a new file that is executed on every boot.
```
nano /storage/.config/autostart.sh
```

Paste the following content in it, **but adjust it with your region and timezone**.

Since my region is Europe and i picked Amsterdam as my timezone, this is what it looks like. If your timezone is not Europe/Amsterdam, you will have to change that.
```
rm /var/run/localtime
ln -sf /usr/share/zoneinfo/Europe/Amsterdam /var/run/localtime
```

Save the file `ctrl + x`, `y`.

The last step is to give this script the right permission to be executed.
```
chmod +x /storage/.config/autostart.sh
```

That's it. Reboot your Pi by typing `reboot` and when Plex boots up, the clock in the top corner should display the correct time.

From now on, every time PMP boots, the timezone fix will be applied and the displayed time is in your timezone.

## After updating PMP
I think it's obvious, but when you flash a new image to the SD card of the Pi, you will have to do these steps again. That is also the main reason why i wrote it here, so i can quickly do it again whenever i flash a new image.
