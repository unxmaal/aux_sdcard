# aux_sdcard
A/UX SD card configuration and info

# Requirements
* Functioning A/UX-compatible Mac (reference http://www.aux-penelope.com/hardware.htm for supported hardware)
* SCSI2SD (v5 or v5.1 are recommended)
* Good quality 8GB microSD card
* Good quality SD/USB adapter

# Recommendations  
* Newer Mac with USB 3.0
* Functional old SCSI HDD with Mac OS 7.5.3+
* Lido
* LaCie Silverlining

# SD Card image layout 
The image has two "LUNs":
* 2GB "drive" with A/UX and a MacPartition for booting
* 5GB "drive" mounted in A/UX as /opt, that includes a Jagubox mirror and another more recent mirror

# Configuring the SCSIS2SD 
* You also need the `scsi2sd_aux_3.1.1-pristine.xml` file that is in the GitHub repo so download/save that as well.

* Go here (http://www.codesrc.com/mediawiki/index.php/SCSI2SD) and scroll down to "Files" to download the correct scsi2sd-util.dmg for the version of the card you have (i.e., you need either SCSI2SD v5, 5.1 or 6)

  * SCSI2SD v5/5.1: [http://www.codesrc.com/files/scsi2sd/latest/mac/scsi2sd-util.dmg](http://www.codesrc.com/files/scsi2sd/latest/mac/scsi2sd-util.dmg)
  * SCSI2SD v6: [http://www.codesrc.com/files/scsi2sd-v6/latest/mac/scsi2sd-util6.dmg](http://www.codesrc.com/files/scsi2sd-v6/latest/mac/scsi2sd-util6.dmg)
  * Quick Start Guide: [http://www.codesrc.com/mediawiki/index.php/SCSI2SD_UserManual](http://www.codesrc.com/mediawiki/index.php/SCSI2SD_UserManual)
* Open DMG and drag `scsi2sd-util` application somewhere (Desktop, Downloads, etc)
* Open Terminal
* Drag `scsi2sd-util` and drop into Terminal window, then press Return to launch (or you could manually type the full path to the scsi2sd-util application)
* Connect SCSI2SD to Mac via micro-USB cable and the `scsi2sd-util` GUI should then connect and indicate the card was recognized (via the bottom status bar)
* Select File menu -> Open file...
* Browse to directory where you saved `scsi2sd_aux_3.1.1-pristine.xml`
* Click "Save to device" button and XML configuration (SCSI settings, microSD card partition offsets, etc) will write to the hardware - DO NOT DISTURB, this process takes a few seconds and you will see the amber LED on the card blink w/ read/write activity
* Ensure the SCSI2SD card has the latest firmware installed per the bottom status bar. Current firmware is [here](http://www.codesrc.com/files/scsi2sd/latest/firmware/) for v5/5.1 and [here](http://www.codesrc.com/files/scsi2sd-v6/latest/firmware.dfu) for v6.
* I personally wouldn't bother with checking the firmware (since it's a manual, multi-step process) unless you bought your hardware before 2018 
* When complete, disconnect SCSI2SD from micro-USB
* Verify that the SCSI2SD shows 2 drives are enabled

# Deploying the image (using dd)
* You *might* be more comfortable working with a GUI to write the microSD image; you can grab ApplePi-Baker from https://www.tweaking4all.com/software/macosx-software/macosx-apple-pi-baker/ if you prefer. 

  If you do, skip down to the section called "Deploying the image (using ApplePi-Baker)" to write the disk image.
* Decompress the image
* `dd` the image to the SD card
NOTE: I have been unable to get the image to `dd` properly under Linux. I recommend using a Mac for this part. 

* In Terminal.app, run ```diskutil list``` 
* Note the highest-numbered disk name. It'll be something like `/dev/disk4`.
* Insert the SD card into your USB/SD adapter, and connect the adapter to your Mac.
* In Terminal.app, run ```diskutil list``` again.
* Note the highest-numbered disk name. It should have changed.
* You can also verify based on the "size" reported for the disk's volume 0. 

The disk shouldn't need to be formatted, as you are overwriting its partition information with new data.

* Unmount the SD card
```diskutil unmountDisk /dev/<your disk name>```

* Dump the image to the disk
You'll set the output file to be "r" plus the name of the SD card. This tells OS X to write to the "raw" device. The write will take much longer, but it has a better chance of making a functional SD install.

```sudo dd if=aux_3.1.1_working.img of=/dev/r<your disk name>```

# Deploying the image (using ApplePi-Baker)
* Grab latest DMG from [https://www.tweaking4all.com/software/macosx-software/macosx-apple-pi-baker/](https://www.tweaking4all.com/software/macosx-software/macosx-apple-pi-baker/), open and drag application to /Applications
* Launch and ignore any compatibility warnings ("needs to be updated") related to macOS 10.14 "Mojave"
* Replace the first 90% of the above **"Deploying the image"** section with:
* Insert microSD card into USB card reader, SD card adapter, etc
* Launch ApplePi-Baker and enter admin password (sudo)
* Select removable drive/microSD card on left-side window (e.g., ``/dev/disk3`` but this will vary so make sure all USB drives, etc besides the SD card are disconnected)
* From the right-side window (*"Pi-ingredients: IMG Recipe"*):
* Click on ... button to browse HD and select IMG file you downloaded from the above GitHub link  
* Full path to image file will populate text field
* Click on Restore Backup. 
* Go make dinner; this will take a while (at least 30 minutes; in some cases, much longer) 
* Take note of the microSD card's device name (`/dev/diskX`, where `X` is a number) from the left-side window in ApplePi-Baker. You'll use this for the next section.


# Verify the SD card's partitioning

```diskutil list```

You should see something similar to this:
```
/dev/disk5 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     Apple_partition_scheme                        *7.9 GB     disk5
   1:        Apple_partition_map                         32.3 KB    disk5s1
   2:             Apple_Driver43                         27.6 KB    disk5s2
   3:             Apple_Driver43                         37.9 KB    disk5s3
   4:              Apple_Patches                         262.1 KB   disk5s4
   5:             Apple_Driver43                         16.4 KB    disk5s5
   6:                  Apple_HFS MacPartition            16.8 MB    disk5s6
   7:            Apple_UNIX_SVR2                         536.9 MB   disk5s7
   8:            Apple_UNIX_SVR2                         248.6 MB   disk5s8
   9:            Apple_UNIX_SVR2                         1.3 GB     disk5s9
```

* Example:
```
$ diskutil list
....
/dev/disk5 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     Apple_partition_scheme                        *7.9 GB     disk5
```
My SD card is 8GB, and it's at /dev/disk5 .

```$ sudo dd if=aux_3.1.1_working.img of=/dev/rdisk5
Password:
15450113+0 records in
15450112+0 records out
7910457344 bytes transferred in 41375.848618 secs (191185 bytes/sec)
```
As you can see, this took nearly 11 hours on my cMP without USB 3.0. 

Verify it:

```diskutil list
...
/dev/disk5 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     Apple_partition_scheme                        *7.9 GB     disk5
   1:        Apple_partition_map                         32.3 KB    disk5s1
   2:             Apple_Driver43                         27.6 KB    disk5s2
   3:             Apple_Driver43                         37.9 KB    disk5s3
   4:              Apple_Patches                         262.1 KB   disk5s4
   5:             Apple_Driver43                         16.4 KB    disk5s5
   6:                  Apple_HFS MacPartition            16.8 MB    disk5s6
   7:            Apple_UNIX_SVR2                         536.9 MB   disk5s7
   8:            Apple_UNIX_SVR2                         248.6 MB   disk5s8
   9:            Apple_UNIX_SVR2                         1.3 GB     disk5s9

```

# A/UX setup

# Logins
User: root
Password: password

You should change these, or maybe just not connect a 25 year old OS to any insecure network.

I recommend creating your own user. You will need it if you care to ssh into your A/UX system.

# Networking
* Determine a static IP for your system

NOTE: I only have the built-in ethernet interface. If you have another card, you will need to determine its name. 

* Run "newconfig" as root. Follow the prompts, and enter your system's IP info when asked. If you have a separate networking card (i.e., an Apple "SONIC" NuBus Ethernet Card) and it's compatible/recognized, you will see interfaces ao0 (built-in Ethernet), as0, as1, etc. Make note of the correct interface to be configured and enabled at boot.   
* Reboot.

* Change the system IP in the following files:
  * at the end of /etc/networks
  * /etc/hosts
  * /usr/local/etc/sshd_config , for listenAddress

* Determine your default gateway's IP
* Change it in /etc/rc, for the "route add default" entry

* Determine your nameserver
* Change it in /etc/resolv.conf

# Enabling SSH
* Add a non-root user

```
adduser -h /users/newusername newusername
```

* Set a password for the new user
```
passwd newuser
```

* Edit /etc/rc, and uncomment the line at the bottom for starting sshd
* Reboot, or start sshd manually
* From another host, ssh yournewuser@yourauxbox
  * It will always take a long time to authenticate you.

# Troubleshooting

## Boot Problems
You *will* have problems booting this image. 

A/UX-compatible Macs stored certain boot parameters in PRAM. It stored other boot parameters as flags set on volumes. It also stores some boot parameters in a parallel hellscape of hate and rage. 

Here are the steps I've taken to get *my* Quadra 950 to boot. Yours will be different.

### Zap PRAM
* Power off. 
* Hold cmd-opt-shift, press the on-keyboard power button, quickly press and hold 'r' before the boot chime. 
* Keep holding this combo for two chimes

### Force booting off a specific SCSI ID
* Power off. 
* Hold cmd-opt-shift, press the on-keyboard power button, quickly press and hold '0' before the boot chime. 
* This boots SCSI ID 0.

* Power off. 
* Hold cmd-opt-shift, press the on-keyboard power button, quickly press and hold '1' before the boot chime. 
* This boots SCSI ID 1.

Your SCSI2SD card shouldn't be at a different ID than 0 or 1, but as you see above, you can select the ID easily.

### Boot off of floppy disk to set Startup Disk
A Floppy Emu with a System 7 version of "Disk Tools.img" or a regular bootable floppy can serve as a failsafe to get you to Control Panel, where you can manually run Startup Disk or Disk First Aid if there's a bigger problem. (like if you got your SD card out of the clearance bin at Discount Auto Parts) 

## Specific examples
### Unhappy Mac
Symptom: Black screen with unhappy Mac, error code 000021
Fix: Force SCSI ID 0, then 1

### Flashing floppy icon
Symptom: Grey screen with floppy icon
Fix: Force SCSI ID 0, then 1
