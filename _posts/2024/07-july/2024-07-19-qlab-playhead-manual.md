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

TODO: gif screenshots of settings a new network to QLab

The playhead assign two IP addresses. One for the playhead itself and one for the mac. The playhead will always default to the IP address `192.168.123.1/32` and the mac to `192.168.123.2/32`. The playhead also listen from messages from QLab on port `53001` as this is the default that Qlab replies to OSC messages on.

### Blank screen
To turn the screen off or on send one of the following message to the playhead using a QLab network cue.
Off:
```OSC
/settings/blank true
```
(The blank setting is on and nothing is displayed on the screen)

On:
```OSC
/settings/blank false
```
(The blank setting is off and the playhead position is displayed on the screen)
 > Using this setting will only turn the screen off. The pi playhead will continue to request the position of the playhead in QLab but not write the result to the screen.
 {: .prompt-warning}

### Brightness
To set the brightness of the screen send the following message to the playhead using a QLab network cue:
```OSC
/settings/brightness X
```
Where X is in the range 1 - 7.
The default is: 3

> Your mileage may vary with this setting. Although the screen I used responds to 7 different brightness levels but I can only see about 3 changes by eye.
{: .prompt-info}

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

#### The easy/safe way
Get a new SD card and follow the how to make guide from start to finish. If you fail to rebuild the update then you can always swap back to the old SD card.

#### The proper way
  > Updating the software version via this method is only recommended for those familiar with a Linux operating system
  {: .prompt-danger}
  1. Take your pi playhead and connect it to a screen and keyboard.
  2. Connect to a Wi-Fi network with internet access
  3. Run the command `sudo apt update && sudo apt upgrade -y` (update the operating system on the Raspberry Pi)
  4. Navigate to the project folder
  5. Run the command `git pull origin` (get new updates from the online repository)
  6. Resolve any merge conflicts (this should only happen if you have modified the original install so I'm assuming you'll know how to do this yourself)
  7. Follow any instructions on the release page to fix any breaking changes
  8. Test that the functionality still matches what you would expect
  
### Licence
TODO: copy MIT licence from GitHub
