## Remote unlocking of a LUKS encrypted drive on Kali / Debian

Generating a new SSH key:
```
ssh-keygen -t rsa -b 4096
```

We then update and install some essential packages we will need for our process:
```
apt-get update && apt-get dist-upgrade
apt-get install busybox cryptsetup dropbear kmod
```

Now we deal with the Dropbear SSH access. We copy over our SSH public key to the RPi image :
```
echo {our public SSH key} > /etc/dropbear-initramfs/authorized_keys
```

…and limit the SSH connection to allow interaction with the cryptroot application only.
```
nano /etc/dropbear-initramfs/authorized_keys
```

We paste the following before the ssh public key begins, but on the same line:
```
command="export PATH=/bin:/sbin:/usr/bin:/usr/sbin; /scripts/local-top/cryptroot && kill -9 `ps | grep -m 1 'cryptroot' | cut -d ' ' -f 3`"
```

Here is an example file with this added to it:
```
command="export PATH=/bin:/sbin:/usr/bin:/usr/sbin; /scripts/local-top/cryptroot && kill -9 `ps | grep -m 1 'cryptroot' | cut -d ' ' -f 3`" ssh-rsa KEY_REMOVED_FOR_EXAMPLE root@HOST
```
 
We need to force cryptsetup into the initramfs:
```
echo 'export CRYPTSETUP=y' > /usr/share/initramfs-tools/conf-hooks.d/forcecryptsetup
```

Update the initramfs:
```
update-initramfs -u -k all
```

Update grub:
```
update-grub2
```

Now, you should be able to reboot your system and remote into it using ssh-keys and then unlock the system drive.


