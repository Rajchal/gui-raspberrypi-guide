# Setup you need to follow

````bash
sudo nano /boot/firmware/config.txt
````

- It should contain

````bash
dtoverlay=vc4-kms-v3d
gpu_mem=128
````

- Reboot and check

````bash
sudo reboot
````

## Now install X, Openbox, and Dependencies

````bash
sudo apt update
sudo apt install --no-install-recommends \
    xserver-xorg \
    x11-xserver-utils \
    xinit \
    openbox \
    dbus-x11 \
    lxterminal \
    xterm \
    mesa-utils
````

- Also do this

````bash
sudo dpkg --remove --force-depends x11-xserver-utils:armhf
sudo apt install --reinstall x11-xserver-utils
````

- This too

````bash
sudo apt install feh unclutter
````

## Configure `.xinitrc` to launch openbox

````bash
vim ~/.xinitrc
````

- Add

````bash
#!/bin/sh
exec openbox-session
````

- Make it executable

````bash
chmod +x ~/.xinitrc
````

## edit to firmware

- use vim to edit /boot/firmware/config.txt

```bash
hdmi_force_hotplug=1
hdmi_group=2
hdmi_mode=82
````

- clean up weird x configs

````bash
sudo rm -f /etc/X11/xorg.conf
sudo rm -f ~/.xserverrc
````

- reboot again

## Now the holy grail code which i spent all night to debug this is the keyyyyyy

- Create minimal conf

````bash
sudo mkdir -p /etc/X11/xorg.conf.d
sudo nano /etc/X11/xorg.conf.d/99-pi.conf
````

- Paste this goodies

````bash
Section "Device"
    Identifier  "VC4"
    Driver      "modesetting"
    Option      "kmsdev" "/dev/dri/card1"
EndSection

Section "Monitor"
    Identifier "Monitor0"
    Option "DPMS" "false"
EndSection

Section "Screen"
    Identifier "Screen0"
    Device     "VC4"
    Monitor    "Monitor0"
    DefaultDepth 24
EndSection

````

- Now use startx

````bash
startx
````


> Note: if something goes worng try the code below and should work after a reboot

- Disable broken `card1`

````bash
sudo nano /etc/udev/rules.d/99-disable-card1.rules
````

- Add this

````bash
KERNEL=="card1", SUBSYSTEM=="drm", TAG+="uaccess", OPTIONS+="ignore_device"
````

- Now reload the rules

````bash
sudo udevadm control --reload
sudo udevadm trigger
````

- You can cinfirm the `card1` is no longer available to your user with:

````bash
ls -l /dev/dri
````

- and

````bash
test -r /dev/dri/card1 && echo "still readable" || echo "not readable"
````

Thankyou hope this helps and you dont have to wake all night like me
