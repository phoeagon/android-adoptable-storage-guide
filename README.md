# Using SD card for Android storage expansion (adoptable storage) - 2025 [No ROOT required]


This is a practical guide on using Android adoptable storage to offload certain apps and data into an external storage media, often a micro SD card, in devices with limited
internal storage.

This guide is intended for folks with some technical knowledge. If you don't understand "Developer options", etc, please _DO NOT FOLLOW THESE STEPS_!

_You are responsible for any and all outcomes from following these steps, including any loss of data and/or loss of function of your devices._

# TL;DR

(Please go through the rest of doc. Don't DR it)

```
adb shell
```

```
sm set-force-adoptable on
sm list-disks
# should return something like: disk:179,0
sm partition disk:179,0 mixed 10  # 10 is the 10% of portable storage
sm list volumes
sm format private:179,34
sm mount private:179,34
df -h
reboot
```

Very importantly:
1.  DO NOT format the whole card as private. Many devices' software hates it and behaves weird when it detects an SD card without an external/portable partition.
2. `sm partition disk:179,0 mixed 10`: use the correct disk identifier from `sm list-disks`. The number here represent the ratio of the _first_ partition, which is usually the external/portable storage (which cannot be used to store apps). Contrary to instructions like [these](https://gist.github.com/mvidaldp/84926c3e6e0d980a9833d8dbee93f0c8) on the devices I tested, most have the external/portable/exFAT partition at the front of this ratio.
3. `mixed 10`: also giving a very small percentage doesn't work. Your tablet/phone may refuse to continue if this ratio is too low. 10% reliably worked for me. Also, note that you likely cannot resize the partitions without losing all data, ideally you never run out of storage on both the external/portable storage and the internal/encrypted storage.
4. `df -h` this shows actual disk usage stats. Verify the last couple of lines where you should see your new private storage on the SD card. Its total capacity should match that from `mixed 10` or whatever ratio you specified. If it does not (e.g., it has `1-ratio` of total capacity), go through `sm partition ...` again with different parameters.
5. `reboot`: reboot your device and validate that it works properly, before adding more data to the card.

# Prior art

Since adoptable storage became available on Android, much has been written on GitHub about utilizing it to get more effective storage space for Android devices. Sources like [this](https://gist.github.com/mvidaldp/84926c3e6e0d980a9833d8dbee93f0c8),
[this](https://oopsmonk.github.io/posts/2017-02-13-android-adoptable-storage/) are very helpful for beginners setting it up. In addition, this [Reddit post](https://www.reddit.com/r/Android/comments/496sn3/lets_clear_up_the_confusion_regarding_storage_in/)
discussed a lot of the nuances and caveats of Android storage.

# Overview

Many prior art literatures were created when adoptable storage first became available in Android N. Recently, I purchased a handful of <$100 Android devices and attempted to use SD cards to expand their storage (because
obviously $100 devices don't have much storage), some of which from more reputable manufacturers (e.g., Moto) while others are effectively unbranded. I found a few caveats and therefore seeks to create a new guide on the
best practices on creating an adoptable storage in Android. This guide discusses different ways Android uses storage spaces, how different setup results in different storage utilization, and relevant caveats.

# Warning

Device manufacturers don't often test out deveoper options thoroughly, especially in things like adoptable storage.

_You are responsible for any and all outcomes from following these steps, including any loss of data and/or loss of function of your devices._

For the most part, the steps here don't rely on having ROOT or unlocked bootloaders. Generally speaking, if you have ROOT or unlocked bootloader access,
you have a lot more options, like creating symbolic links to organize your storage space. However, as also highlighted below, adoptable storage space is
encrypted and essentially opaque to any other devices if without the encryption key, which could only be extracted in a ROOTed device. By setting up
adoptable storage space, you are placing data in a partition that is encrypted, cannot be resized, cannot be otherwise extracted data from in case of a
data corruption.

# Implications

