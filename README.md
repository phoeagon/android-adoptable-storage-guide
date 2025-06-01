# Using SD card for Android storage expansion (adoptable storage) - 2025 [No ROOT required]


This is a practical guide on using Android adoptable storage to offload certain apps and data into an external storage media, often a micro SD card, in devices with limited
internal storage.

This guide is intended for folks with some technical knowledge. If you don't understand "Developer options", etc, please _DO NOT FOLLOW THESE STEPS_!

_You are responsible for any and all outcomes from following these steps, including any loss of data and/or loss of function of your devices._

# TL;DR

(Please go through the rest of doc. Don't DR it)

Ideally you go through these steps when you first setup the device, or at least before you run out of spaces.

1. Go to Settings - System. Click on "Build number" 10 times to enable "Developer options"
2. Go to Settings - System - Deveoper options. Enable "USB debugging" (and optionally "wireless debugging" if you so prefer). Enable "force allow apps on external".

(You don't need root, but you need ADB access)

```
adb shell
```

```
sm set-force-adoptable on
sm list-disks
# should return something like: disk:179,0
sm partition disk:179,0 mixed 10  # 10 is the 10% of portable storage
# by the time you reach here, potentially any data on the SD card is lost.
sm list volumes
sm format private:179,34
sm mount private:179,34
df -h
reboot
```

Very importantly:
1.  DO NOT format the whole card as private (`sm partition disk:179,0 private`). Many devices' software hates it and behaves weird when it detects an SD card without an external/portable partition.
2. `sm partition disk:179,0 mixed 10`: use the correct disk identifier from `sm list-disks`. The number here represent the ratio of the _first_ partition, which is usually the external/portable storage (which cannot be used to store apps). Contrary to instructions like [these](https://gist.github.com/mvidaldp/84926c3e6e0d980a9833d8dbee93f0c8) on the devices I tested, most have the external/portable/exFAT partition at the front of this ratio.
3. `mixed 10`: also giving a very small percentage doesn't work. Your tablet/phone may refuse to continue if this ratio is too low. 10% reliably worked for me. Also, note that you likely cannot resize the partitions without losing all data, ideally you never run out of storage on both the external/portable storage and the internal/encrypted storage.
4. `df -h` this shows actual disk usage stats. Verify the last couple of lines where you should see your new private storage on the SD card. Its total capacity should match that from `mixed 10` or whatever ratio you specified. If it does not (e.g., it has `1-ratio` of total capacity), go through `sm partition ...` again with different parameters.
5. `reboot`: reboot your device and validate that it works properly, before adding more data to the card.

After these steps, you should now be able to move individual apps onto your SD card's adoptable storage.

## Optional steps

(These steps are often omitted in the instructions, likely because they have to be carried out on the UI, separate from the `adb shell` tricks. Many people missed these steps, and ended up with all OBB files still in the SD card)

(These steps in effect makes the system mount some paths of your SD card to `/storage/emulated/` instead of internal storage. This makes games write asset files (OBB, see below) onto your SD card. Depending on your usage (again see below), you may
end up with too much space in internal storage and too little on your SD card. In that case, move apps back from SD card to internal storage).

If you use games a lot, go to Settings - Storage. Pick "SD card" from the drop down. (If there are two, go to the encrypted/internal storage one. If you can't tell, try both).
Click on the hamburger menu on upper right, choose "Migrate data".
Go through the process. This would move a lot of data into your SD card. Make sure that it has enough spaces.
If it doesn't have enough spaces, delete apps or data from apps and try again. It's best to do this early before there are too many things in your storage.

If go did this, go through these steps to ensure `/storage/emulated` is mounted correctly:

`adb shell`
`mount`

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

Note that since you don't have root on your device, the "internal/encrypted" partition of your SD card is effectively a huge blob, and cannot be manipulated by any other devices, or if your Android device dies or otherwise loses its encryption key.

In effect, you are marrying the SD card to that Android device. You can't move the card to a different device and expect it to work (worse yet, after you move it back, the partition could be corrupted, in which case all data in that encrypted partition is gone).
You can make a full copy/dump of your card onto another one of the same size, but you cannot clone it to a larger card and also resize it larger (not without ROOT).

Once you picked a card and a mix ratio (`sm partition disk:179,0 mixed 10`), you are permanently stuck with it on the device if without losing data in the encrypted partition. If your device has enough storage, you can migrate data back to internal storage, recreate
partitions on a larger card and move back. However, you cannot achieve this process in an external device as it lacks the encryption key.

If you have ROOT access, [this github repo](https://github.com/Aufschlauer/swap-adoptable-storage) contains instructions on resizing the encrypted partition. However, without root access, the only repeatable success we have is to use `dd` to do sector level data
copy to clone the whole SD card to a different one.

## SD card cloning

`dd if=/dev/sde of=SomeBigFileImage bs=1M`

This should make a backup of your SD card (`/dev/sde`) as a file on your hard drive `SomeBigFileImage`.

Alternatively, you can do a straight copy to a different SD card:

`dd if=/dev/sde of=/dev/sdf bs=1M`

Obviously, check the file names and device names carefully. If you got it wrong and data is overwritten, _it's gone permanently_.

To replace an SD card of the same size (say the original one failed), you can write it back to the new card

`dd if=SomeBigFileImage of=/dev/sde bs=1M`

In theory Android could implement some nonce mechanism to prevent such a rollback attack. In reality, this `dd` method is widely reported to be working (though I never personally tried).

This should provide a good way to periodically back up data on your SD card, should it fail one day.

Note that even if you `dd` to a larger card, you cannot resize the partitions since you don't have the partition key. I suspect in theory, resizing the external/public exFAT folder should be possible, but it has yet to be exercised repeatedly by the community. In any case, that's likely
not the primary target you want to increase in size anyway.

# App storage usage

See https://www.reddit.com/r/Android/comments/496sn3/lets_clear_up_the_confusion_regarding_storage_in/ for more details.

Generally speaking, your apps use storage space in the following ways:

1. APK files and JIT-compiled artifacts: The core APK files and any automatically compiled outputs.
2. User data (e.g., chat log, cache space): Usually social network apps, messagers, etc uses a lot of space to store avatar, emoji, video and photos. Some apps also (arguably incorrectly) uses it for caches but do not properly wire them to the "clean cache" feature in Android or provide separate ways in app to clean the cache.
3. For mostly games, asset files that are separate from APK. These used to be OBB files but are now through "Play Asset Delivery". They behave similarly in terms of storage. Despite OBB files being no longer an option for new apps on Play, you may still encounter them in popular apps today, such as "Azur Lane".

For 1. and 2., they can be individually moved, per app, to internal storage or adoptable storage. For 3, there is a global preference that determines how `/storage/emulated` is mounted, through the "Setting - Storage - SD card/phone - migrate data" process. It has to be entirely from internal storage, or entirely
from adopted storage, but cannot be pulled. this means that if you have a lot of game assets in your device, you may end up with a lot of storage use in 3, such that it doesn't fit either internal storage or external storage. In that case, you have no choice but to get a larger SD card (or if you can replace 
storage chips on your device, in which case you are probably not on this page).

## Exception: System apps

Many docs from "Android N" days cite "move to SD card" as an opt-in feature per-app. This is largely correct, but is now mostly irrelevant due to the "Force allow apps on external" option in "Developer options". However, there is one notable exception to this rule: system apps.

System apps cannot be moved to adoptable storage, even with the developer option "Force allow apps on external".

Generally speaking, system apps could have large APK files and user data, but since most phones don't preinstall games, hopefully OBB files. I don't know if the steps above to move OBB files will apply to system apps, since I don't have such a system game app to test with.

For system apps, you can disable them, which removes their update APKs from `/data` (which is usually large) in internal storage, as well as their data (at least in my experiences. If not, you can always manually clear its app data storage).

Some apps have "Go versions" or "Lite versions" that are much smaller in size and caters users with low-end devices. Notably, "GMail Go", "Google Map Go", "Instagram Lite", "Twitter Lite" are worth checking out. On the other hand "Tiktok Lite" is still very large.

If you still don't have enough space after going through these, you probably have to ROOT the device or at least unlock the bootloader to boot once from a custom recovery (e.g., TWRP) to remove bloatware. Such is beyond the scope of this doc.

# Data migration and move

The storage "migrate data" step on UI effectively moves data and mounts your encrypted adoptable storage to `/storage/emulated`, which effectively moves OBB files, and other misc files to your adoptable storage (SD card).

This step is often ommitted in most instructions, and is indeed option. If you don't have games or apps (much rarer) that uses external assets, you don't need to follow those steps.

Also it's not clear to me whether these steps could be achieved by `adb`, or how the global mount point preference is stored or manipulated. However, luckily a stock/un-rooted Android provides a way to do this on the UI.

