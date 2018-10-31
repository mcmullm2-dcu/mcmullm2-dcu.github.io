---
layout: post
title: Running an Android Virtual Device on an AMD PC
tags: [sda,android,avd]
---
Many Android tutorials assume that users are working on Intel-based machines. Those
of us using AMD processors are used to running code in other environments without
difficulty, so when setting up an Android Virtual Device (AVD), it can be frustrating
to find that the normal instructions don't work!

After some trial and error, the steps listed below seemed to do the trick. Instead of
turning off Hyper-V and using HAXM instead, it's the opposite. Here are the steps I took:

1 Turned off Intel HAXM in Android Studio (under SDK Manager, SDK Tools)
2 (Activated Virtualization in the BIOS)[https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/virtualization/sect-virtualization-troubleshooting-enabling_intel_vt_and_amd_v_virtualization_hardware_extensions_in_bios]
3 Installed (Windows Hypervisor Platform in Windows Features)[https://blogs.msdn.microsoft.com/visualstudio/2018/05/08/hyper-v-android-emulator-support/]
4 Uninstalled all previous AVDs
5 Created new device, Nexus 5X, Marshmallow, x86_64, 2GB RAM (I increased the amount of RAM as previous AVDs seemed to crash a lot. However, this may not be necessary)
6 Launched the new AVD from AVD Manager (Took about 4 minutes, a big improvement)
7 Quit AVD -- this saved the state and made future launches almost instant

So instead of waiting 20-30 minutes to launch, only to crash again, I ended up with
an AVD that booted straight away, and ran reliably.
