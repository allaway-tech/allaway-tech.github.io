---
layout: post
title: Raspberry Pi QLab playhead manual
date: 2024-07-10
categories: [QLab, Raspberry Pi] # Can be anything
tags: [qlab,raspberrypiplayhead] # Must be lowercase
# img_path: /media/posts/images/2024-07-08-qlab-playhead/
---

TODO: move images to the media.allaway.tech server

This is the manual for the Raspberry Pi playhead project. If you are looking for the instructions to make one then you can find that post here. (TODO: link to 2024-07-10-qlab-playhead)

![Picture montage of a raspberry pi with QLab logo and 7 segment display number over the top](/media/posts/images/2024-07-08-qlab-playhead/splash-image.png)

## Getting started
To boot the pi playhead you will require a USB A to micro USB cable. This cable connects between a USB port on the host mac (running a QLab session) and the data port on the Raspberry Pi Zero. Once the pi playhead is connected it will start the boot process automatically. Once the system has loaded you will see the message:

![A rendering of a seven segment screen reading LOADIN](/media/posts/images/2024-07-08-qlab-playhead/loadin.png)

Followed by the version number of the software that the playhead is running. At the time of writing this guide that should be:

![A rendering of a seven segment screen reading V0.1.3](/media/posts/images/2024-07-08-qlab-playhead/v0.1.3.png)

Once the playhead has successfully connected to the QLab session the it will display a six character truncated cue number of the cue that the playhead is sitting on.

## Settings

To control the pi playhead you will need to add it as a network destination in QLab.

TODO: screenshots of settings a new network to QLab

The playhead assign two IP addresses. One for the playhead itself and one for the mac. The playhead will always default to the IP address `192.168.123.1/32` and the mac to `192.168.123.2/32`. The playhead also listen from messages from QLab on port `53001` as this is the default that Qlab replies to OSC messages on.

### Blank screen
To turn the screen off or on send one of the following message to the playhead using a QLab network cue.
Off:
```OSC
/settings/blank true
```

On:
```OSC
/settings/blank false
```
 > Using this setting will only turn the screen off. The pi playhead will continue to request the position of the playhead in QLab but not write the result to the screen.
 {: .prompt-warning}

### Brightness
To set the brightness of the screen send the following message to the playhead using a QLab network cue:
```OSC
/settings/brightness X
```
Where X is in the range 1 - 7.
The default is TODO: check default.

### Refresh rate
To set the refresh rate of the screen send the following message to the playhead using a QLab network cue:
```OSC
/settings/refresh X
```
Where X is a time in seconds expressed as a decimal. 
The default is: 0.25
 > Use this setting wisely. For every refresh rate period the pi playhead will first send a ICMP ping message to ascertain if the mac is there and the send an OSC message requesting the position of the pi playhead. The lower the value of X the more network traffic that the pi playhead will produce.
 {: .prompt-warning }
 
### Update the software version
  > Updating the software version is only recommended for those familiar with a Linux operating system
  {: .prompt-danger}


