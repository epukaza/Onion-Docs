---
title: Omega Router
layout: guide.hbs
columns: two
devices: [ Omega2 ]
order: 1
---

## The Omega as a Router {#omega-router}

<!-- high level introduction of what we're doing in this tutorial: turning the omega into a router, brief description of a router -->

This tutorial will show you how you can use the Omega as a WiFi router. A router is a device uses an Ethernet connection to provide a wireless area network. We are going to plug our Omega into a modem to provide internet to an area.

The Ethernet Expansion is required to give your Omega access to an Ethernet port. By using the Ethernet Expansion, we can turn our Omega into a low-cost yet effective router.

<!-- illustration showing the whole system -->

### Overview

| <span style="font-weight:normal">Tutorial Difficulty</span> | Intermediate |
| :--- | :--- |
| Time Required| **10 mins** |
| Required Materials | Omega2 or Omega2+<br>Expansion Dock<br>Ethernet Expansion |


We're going to first setup the hardware, then we'll change some Omega config files that will enable the Omega to forward our connection properly.

### Step 1: Setup the Hardware


Connect your Ethernet Expansion to the Expansion dock, and plug in the Ethernet cable, as shown below:

![Omega Router](https://raw.githubusercontent.com/OnionIoT/Onion-Docs/master/Omega2/Documentation/Doing-Stuff/img/omega-router-pic-1.jpg)

After you have connected everything, power on the Omega.


### Step 2: Setup the Omega

<!-- batch2: explanation of which wifi you're disabling -->

The next step is to disable the WiFi connection on the Omega. We want our Omega to connect to the internet via the ethernet connection and so we're going to turn off the WiFi on our Omega

>We're going to be disabling the WiFi on the Omega so you'll need to make sure that you've established a serial connection with your Omega. For more information, please refer to this [guide on connecting to your Omega.](#connecting-to-the-omega-terminal)
<!-- batch2: expand on this comment - explain why serial is beneficial in this scenario -->

To do this, you will use the `uci` command to change the access point settings of your Omega.

Enter the following command to set the value of `ApCliEnable` to `0`:

```
uci set wireless.@wifi-iface[0].ApCliEnable=0
```

Then run the following command to save your changes:

```
uci commit wireless
```

This will disable the ApCli device which is used to wirelessly connect to an existing router.


Restart the WiFi network to apply your saved changes:
```
wifi
```

<!-- create a new step regarding the ssid name -->
### Step 3: Changing your Omega Router's Settings

Since you probably don't want uninvited guests on your new router, it is recommended that you change your Omega Router's settings from the default setup, especially the password.

To do so, enter the following commands, substituting `OmegaRouter` and `RouterPassword`:

```
uci set wireless.@wifi-iface[0].ssid=OmegaRouter
uci set wireless.@wifi-iface[0].key=RouterPassword
uci commit
```

#### Changing the Encryption Type

If you wish to keep the default encryption type (`psk2`), you can continue to the next step below.

However, if you wish to change the encryption type, find the type you want in the [UCI wireless encryption list](https://wiki.openwrt.org/doc/uci/wireless/encryption), then substitute it into `YourEncryptionType` and run:

```
uci set wireless.@wifi-iface[0].encryption=YourEncryptionType
uci commit
```

***Note: If you don't know what encryption type to use, just keep the default.***

Please keep in mind that 1st generation WPA is [not secure](http://www.pcworld.com/article/153396/wifi_hacked.html).

#### Restarting the Wifi

Once you have finished customizing the WiFi network, run the following command to restart the WiFi network and apply your settings:

```
wifi
```

### Step 4: Enable `eth0`

The Omega is primarily designed as a development board to prototype WiFi-enabled devices, so by default, we have turned off the ethernet interface `eth0` in the firmware. In order to use the Omega as a router, you will need to re-enable this. 

Enable the Ethernet connection by running:

```
uci set network.wan.ifname='eth0'
uci set network.wan.hostname='OnionOmega'
uci commit
```

This will tell the Omega to turn on the `eth0` interface and we will also be referring to this network as `wan`.

Once you have saved and closed the file, run the following command to restart the network service to reload the new configuration:

```
/etc/init.d/network restart
```

### Step 5: Enabling Packet Routing

Next, you will need to configure the Omega to route packets from the ethernet interface (`eth0`) to your WiFi interface (`wlan0`). To do this, you will be editing the `/etc/config/firewall` file:

Find the block that looks something like the following:

```
config zone
        option name 'wan'
        option output 'ACCEPT'
        option forward 'REJECT'
        option masq '1'
        option mtu_fix '1'
        option network 'wwan'
        option input 'ACCEPT'
```

and do the following:

* Change `option forward 'REJECT'` to `option forward 'ACCEPT'`
* Change `option network 'wwan'` to `list network 'wwan'`
* Add `list network 'wan'` after the `list network 'wwan'` line

What you will end up with is something like the following:

```
config zone
        option name 'wan'
        option output 'ACCEPT'
        option forward 'ACCEPT'
        option masq '1'
        option mtu_fix '1'   
        list network 'wwan'  
        list network 'wan'   
        option input 'ACCEPT'
```

What this tells the Omega to do is to add the `wan` network (which we defined in `/etc/config/network` file) to a firewall zone called `wan`. This zone has already been setup to route packets to another firewall zone called `lan`, which contains the `wlan0` interface.

Once you have saved and closed the file, run the following command to restart the firewall with the updated configuration:

```
/etc/init.d/firewall restart
```

### Step 6: Using the Omega Router

And we are ready! To use the Omega Router, you simply need to connect your computer or your smartphone/tablet to the WiFi network that you configured in Step 4, and your devices should be able to access the Internet via the Omega :)

Happy hacking!
