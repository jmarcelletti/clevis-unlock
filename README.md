# Clevis Unlock

Clevis Unlock is a systemd service file + a BASH script that attempts to unlock data block devices at boot time. This was meant to replace the default way Clevis does this which is passing passwords over fifos w/ no concern for success.

## How this works
It iterates through a list of all block devices on the system and determines if they are LUKS or not.
If they are LUKS, it attempts to `clevis unlock` them. If it returns a success (0) or already unlocked (5) it will just skip to the next device.
If it fails, it will sleep for a second and keep retrying up to N times (60 by default, configurable by param $1)

You still need to setup `/etc/fstab` and `/etc/crypttab` as you would without this package, as it pulls all the bits it needs from them. This has the added bonus of letting this script completely fail and normal methods would do it their way. TLDR: You are (hopefully) no worse off for running this as long as you set it up to do it the "normal way" too.

## Installation / Using

You can modify the paths but by default you should copy `clevis-unlock.sh` to `/usr/sbin/clevis-unlock` and `chmod 755` it. You should copy the `clevis-unlock.service` to `/lib/systemd/system/clevis-unlock.service` or `/etc/systemd/system/clevis-unlock.service` and `systemctl daemon-reload` -> `systemctl enable clevis-unlock.service`. 

When you reboot, as long as you have the proper entries for /etc/crypttab and /etc/fstab the system should unlock/mount as normal. 

## Why?

I didn't like the process of passing things over the FIFO and hoping it worked. This has a little more stability (in my head anyway).. It also works for us on Ubuntu 16.04, which the normal one did not. Granted we had to backport 20+ packages and compile them to make this all work on 16.04 anyway, but it was the driving force behind writing this. We ultimately decided that this way was better not only for 16.04 (because it worked), but 18.04/20.04 because we liked it.

I would probably suggest making the timeout longer in production because a server without its data drive is probably not very valuable. 
