# aux_sdcard
A/UX SD card configuration and info

# Requirements
* Functioning A/UX-compatible Mac
* SCSI2SD
* Good quality 8GB SD card
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
* Ensure the SCSI2SD card has the latest firmware installed.
* Load the config from scsi2sd_aux_3.1.1-pristine.xml 
* Verify that the SCSI2SD shows 2 drives

# Deploying the image  
* Decompress the image
* dd the image to the SD card
NOTE: I have been unable to get the image to dd properly under Linux. I recommend using a Mac for this part. 

* In Terminal.app, run ```diskutil list``` 
* Note the highest-numbered disk name. It'll be something like "/dev/disk4".
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

* Verify the SD card's partitioning

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

* Run "newconfig" as root. Follow the prompts, and enter your system's IP info when asked.
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

## Specific examples
### Unhappy Mac
Symptom: Black screen with unhappy Mac, error code 000021
Fix: Force SCSI ID 0, then 1

### Flashing floppy icon
Symptom: Grey screen with floppy icon
Fix: Force SCSI ID 0, then 1
