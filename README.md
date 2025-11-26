# Using SD Cards for Android Storage Expansion (Adoptable Storage) --- 2025

**No Root Required**

This guide explains how to use Android's adoptable storage feature to
extend usable storage space with a microSD card---especially useful for
devices with limited internal storage.

It assumes you are comfortable with technical settings such as
*Developer options*.\
If you are not, **do not follow these steps**.

> **You are solely responsible for any data loss, device malfunctions,
> or other consequences of following this guide.**

------------------------------------------------------------------------

# TL;DR

(But really, read the full document.)

You should ideally set up adoptable storage **during initial device
setup**, or at least before the device runs out of space.

1.  Go to **Settings → System**. Tap **Build number** 10 times to enable
    Developer Options.
2.  Go to **Settings → System → Developer options**.\
    Enable **USB debugging** (and optionally *Wireless debugging*).\
    Enable **Force allow apps on external**.

You don't need root, but you *do* need ADB access:

``` bash
adb shell
```

``` bash
sm set-force-adoptable on
sm list-disks
# Output looks like: disk:179,0
sm partition disk:179,0 mixed 10   # 10% portable / 90% adoptable
# Warning: This wipes the SD card.
sm list-volumes
sm format private:179,34
sm mount private:179,34
df -h
reboot
```

### Important notes

1.  **Do NOT fully format the card as private
    (`sm partition ... private`).**\
    Many devices behave poorly when the SD card lacks a portable
    partition.

2.  For `sm partition disk:179,0 mixed 10`, the number after **mixed**
    defines the **portable** partition size.\
    On most devices tested, the portable/exFAT partition is at the
    *front*.

3.  Very small percentages (e.g., `mixed 1`) may cause the device to
    reject the setup.\
    `mixed 10` is reliable.

4.  Verify the resulting partition sizes with `df -h`.\
    If they don't match expectations, re-run `sm partition` with new
    parameters.

5.  Reboot and confirm normal operation **before** installing more apps.

Once complete, you can move individual apps to your SD card's adoptable
storage.

------------------------------------------------------------------------

# Optional Steps (Often Missing From Other Guides)

Many guides omit these UI steps, but they are **required** for games and
large asset files.

This process makes the system mount part of your SD card at
`/storage/emulated/`, causing OBB/asset data to be stored on the SD
card.

If you play games:

1.  Go to **Settings → Storage**.
2.  Choose **SD card** from the menu.\
    If two appear, select the **encrypted/adopted** one.
3.  Open the ⋮ menu → **Migrate data**.
4.  Follow the steps. Ensure the SD card has enough free space.

If it fails due to insufficient space, uninstall apps or clear
data/cache and try again.

To verify mount points:

``` bash
adb shell
mount
```

------------------------------------------------------------------------

# Prior Art

Earlier adoptable storage documentation was written around Android N.
Some excellent references include:

-   https://gist.github.com/mvidaldp/84926c3e6e0d980a9833d8dbee93f0c8
-   https://oopsmonk.github.io/posts/2017-02-13-android-adoptable-storage/
-   Reddit discussion:
    https://www.reddit.com/r/Android/comments/496sn3/lets_clear_up_the_confusion_regarding_storage_in/

This guide updates the process based on real-world testing with modern
budget Android devices (\<\$100) from both brand-name and unbranded
manufacturers.

------------------------------------------------------------------------

# Warning

Manufacturers rarely fully test Developer Options features---especially
adoptable storage.\
Again:

> **You are responsible for any data loss or device issues resulting
> from these steps.**

This guide does not require root or an unlocked bootloader.\
However, keep in mind:

-   Adoptable storage is **encrypted**.
-   Only the device that encrypted it can read it.
-   The encrypted partition **cannot be resized**, manipulated, or
    recovered externally.
-   If the device dies, so does access to the SD card's encrypted data.

Using adoptable storage effectively marries your SD card to that
specific device.

------------------------------------------------------------------------

# Implications

Because the encrypted partition can only be accessed by the original
device:

-   Moving the card to another device will **not** work.
-   Moving it *back* may corrupt the partition.
-   You **can** clone the card using `dd` (sector-by-sector), but only
    onto a card of the **same size**.
-   You **cannot** clone to a larger card and expand the partition
    without root.

A root-only tool for resizing adopted storage exists here:\
https://github.com/Aufschlauer/swap-adoptable-storage

Without root, you can only clone via `dd` to identically-sized cards.

## SD Card Cloning

Backup to an image:

``` bash
dd if=/dev/sde of=SomeBigFileImage bs=1M
```

Clone directly to another SD card:

``` bash
dd if=/dev/sde of=/dev/sdf bs=1M
```

Restore backup:

``` bash
dd if=SomeBigFileImage of=/dev/sde bs=1M
```

Be extremely careful with device names. Mistakes can permanently destroy
data.

Even when cloning to a larger card, the adopted partition cannot be
expanded without its encryption key.

------------------------------------------------------------------------

# App Storage Usage

Apps consume storage in three main ways:

1.  **APK & compiled artifacts** -- Core app binaries.
2.  **User data** -- Chats, cached media, thumbnails, local databases,
    etc.
3.  **Game asset files** -- OBB (legacy) or Play Asset Delivery
    (modern).

For #1 and #2, you can move apps individually to adopted storage.

For #3, asset storage location is controlled only by the **global**
"migrate data" step in Settings, not per-app settings.

This means:

-   Game assets cannot be split between internal & external storage.
-   If your assets exceed either storage's size, you must upgrade to a
    larger SD card.

------------------------------------------------------------------------

# System Apps

System apps cannot be moved to adoptable storage, even with **Force
allow apps on external**.

But:

-   Disabling a system app removes its updated APKs (freeing space).
-   Many popular apps have "Go" or "Lite" versions that are
    significantly smaller:
    -   Gmail Go
    -   Google Maps Go
    -   Instagram Lite
    -   Twitter Lite\
        (TikTok Lite is still large.)

If you still lack space, root or a custom recovery may be
necessary---outside the scope of this guide.

------------------------------------------------------------------------

# Data Migration and Movement

The **Migrate data** UI option:

-   Moves many files (including OBB/PAD assets) to the adoptable storage
    partition.
-   Changes how `/storage/emulated` is mounted.
-   Is often omitted in guides but essential for gamers.

There is currently no reliable ADB-only equivalent for this global
preference.

------------------------------------------------------------------------

# End

Use adoptable storage carefully, plan ahead, and always back up
important data.
