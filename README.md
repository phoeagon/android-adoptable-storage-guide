# Using SD card for Android storage expansion (adoptable storage) - 2025 [No ROOT required]


This is a practical guide on using Android adoptable storage to offload certain apps and data into an external storage media, often a micro SD card, in devices with limited
internal storage.

This guide is intended for folks with some technical knowledge. If you don't understand "Developer options", etc, please _DO NOT FOLLOW THESE STEPS_!

_You are responsible for any and all outcomes from following these steps, including any loss of data and/or loss of function of your devices._


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

Generally
