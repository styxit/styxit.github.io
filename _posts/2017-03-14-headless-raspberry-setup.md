---
layout: post
title: Headless Raspberry Pi setup with wifi
description: How to setup a Raspberry Pi, connected to your network over wifi
---

Once in a while want to setup a new Raspberry Pi without the need to connect it to a display, keyboard or ethernet cable. The [Raspberry Pi 3](https://www.raspberrypi.org/products/raspberry-pi-3-model-b/) and [the recently introduced Raspberry Pi Zero W](https://www.raspberrypi.org/products/pi-zero-wireless/) have an onboard wifi chip. This means it can run and connect to the internet without having an ethernet cable connected to it. I wrote this article mainly for myself so i can reproduce the steps, but figured this is probably useful to others.

<!-- more -->

> This article will get you up and running with a Raspberry Pi, connected to your wifi network and accessible over ssh, without ever needing to connect anything to it, besides power.

**Note 1**: I used a Mac when going through the process. Most of the steps are probably almost the same on other operating systems.

**Note 2**: I am using the Raspbian Jessie Lite image which does not have any graphical interface besides the commandline. This guide may also work for the Jessie image *with* the Pixel desktop, however, i did not verify this.

## Requirements:
- Raspberry Pi with integrated wifi chip - Pi 3 or Pi Zero W
- Compatible micro SD card - at least 4GB
- Power source for the Raspberry Pi
- A wifi access point you want to connect the Pi to

## Prepare SD card with Raspbian jessie lite
Get a Raspbian Jessie lite image. Download [the latest Raspbian LITE image](https://www.raspberrypi.org/downloads/raspbian/) from the [official Raspberry Pi website](https://www.raspberrypi.org/). Extract the downloaded zip file (double click). You will now have a `.img` file. In my case it is called `2017-02-16-raspbian-jessie-lite.img`.

Connect you SD card to your computer and use [Etcher](https://etcher.io/) to flash the Raspbian `.img`-file to the SD card. The selected device will be **fully erased**!

![Etcher](/images/posts/headless-raspberry-pi-setup/etcher.png)

You now have a standard Raspbian image which is ready to boot, but not yet ready to fully work headless!

## Configure ssh and wifi
After flashing the image to the SD card, the drive has been ejected. Disconnect and connect the SD card so it gets detected by macOS again. A `boot` drive should appear. Open a terminal window and `cd` to the boot drive using the following command:

{% highlight shell %}
$ cd /Volumes/boot
{% endhighlight %}

### Enable ssh
By default, a clean Raspbian image will have ssh disabled. You would need to boot the system, connect a keyboard and display to the Pi and enable ssh. This step will enable ssh at first boot.

In the `boot` drive, create a file called `ssh`. Just an empty file with exactly that name.
{% highlight shell %}
$ touch ssh
{% endhighlight %}
This will enable ssh when you Pi boots.

### Connect wifi
To make the Pi connect to your Wifi access point at first boot, store the wifi connection details on the Pi's `boot` drive.

{% highlight shell %}
$ nano wpa_supplicant.conf
{% endhighlight %}

Paste the following content in the `wpa_supplicant.conf` file, adjust it with your wifi details and save it with `ctrl + x`. Make sure you pick the configuration that matches your Raspbian version.

For Raspbian Jessie:

```
network={
    ssid="YOUR_SSID"
    psk="YOUR_WIFI_PASSWORD"
    key_mgmt=WPA-PSK
}
```

For Raspbian Stretch and later:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
network={
    ssid="YOUR_SSID"
    psk="YOUR_WIFI_PASSWORD"
    key_mgmt=WPA-PSK
}
```

This will use DHCP to obtain an IP address.

## First boot and connect with ssh over wifi
The SD card is now ready to boot. Disconnect the SD card from your computer and put it in the Pi.

Connect a power source to the Pi and the boot sequence should begin. If all went well, after about 30 seconds the Pi should be fully booted and connected to your wifi network.

Use a terminal to connect to it over ssh:
{% highlight shell %}
$ ssh pi@raspberrypi.local
{% endhighlight %}

The default password for the `pi`-user on a clean Raspbian install is `raspberry`.

## Finishing up
You should be able to connect to the Pi over ssh. Run `sudo raspi-config` on the Pi to update the packages and configure everything.

### Hostname or IP?
If you would like to connect by providing an IP instead of the hostname (maybe because you're about to change the hostname), use `ping` to figure out the IP that has been assigned to the Pi and use that when connecting over ssh.

{% highlight shell %}
$ ping raspberrypi.local
PING raspberrypi.local (10.0.1.68): 56 data bytes
{% endhighlight %}

In this case, the IP of the Pi is `10.0.1.68`.
