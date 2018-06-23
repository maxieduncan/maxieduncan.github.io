---
layout: post
category : projects
title: "Upgrading LineageOS on Nexus 4 using OSX"
tags : [android, lineageos, nexus4]
---
{% include JB/setup %}

The instructions at [Upgrading LineageOS | LineageOS Wiki](https://wiki.lineageos.org/upgrading.html) give you a good high level guide.

You’ll want to install [Android Debug Bridge (adb)  |  Android Developers](https://developer.android.com/studio/command-line/adb) if you don’t already have it installed.

You need to [enable developer mode](https://esausilva.com/2010/10/02/how-to-set-up-adb-android-debug-bridge-in-mac-osx/)  on your phone and turn on Android Debugging:
* To enable Developer Options tap seven times in Build Number: `Settings > About Phone > Build Number`
* Enable: `Settings > Developer Options > Android Debugging`

I had problems connecting to my phone but changing USB cables fixed things. I also got an “unauthorised” message when listing the device via `adb devices` which I resolved by revoking the debug access on the phone then clicking OK on the dialog that popped up on the phone when I tried reconnecting.

Rebooting into recovery mode is easily done with: `adb reboot recovery`.

At this point I also wanted to update TWRP which took a while to complete as it wasn’t obvious that the button at the bottom right of the Install menu allows you to switch between `.zip` and `.img` files. I simply couldn’t find the new TWRP image I’d downloaded until I did this.

Installing the LineageOS brought up another issue. I’m updating a Nexus 4 and received the message:
> this package is for device: mako; this device is .

Annoying, as “mako” is the correct package for the Nexus 4. I read that you can unzip the file and modify `META-INF/com/google/android/updater-script`  to remove this check but before I did this I thought I’d try updating to the latest version of the release I was already running and happily that did the trick.

Before rebooting I needed to update Google apps, as because of licensing they’re not provided as part of Lineage OS. Unfortunately the “nano” version of OpenGApps was too large to install but MindTheGapps installed without issue. However when when I rebooted the phone the Google apps kept crashing so I fell back to the OpenGApps “pico” version which doesn’t crash as much. I guess the Google apps hardware requirements have pretty much surpassed what my old phone can handle.

The whole process took me a few hours because of the issues detailed above and having to re-familiarise with tools I haven’t used in a while. The end result is that I can play with the latest version of Android on old hardware that’s no longer supported; which is fantastic even if the process is somewhat tedious.
