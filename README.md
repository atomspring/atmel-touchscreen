# atmel-touchscreen
Support my Atmel MaxTouch on Linux

I just bought a nice HP Revolve 810 G2, and of course I don't use Windows. This laptop/tablet hybrid supports a really rad little pen so I can write my various schoolnotes and ideas down, something I've been doing since my first HP, a TC1100. Naturally, I expected the stylus to work out of the box as my previous HPs have, but for some reason it didn't, the cursor would only move diagonally no matter where I put the stylus. 

Turns out the source of the trouble is that there is a new piece of hardware powering the stylus and touchscreen, where it used to be a wacom setup (with associated mature software stack) it is now an Atmel Maxtouch system.

Eventually, enough looking around got me to modifying a single line of code in the HID code of the kernel. Only way to implement this change is compiling a kernel. It's easy though:

<code>sudo -i
<code>mkdir -p /usr/src/kernel-hacking
<code>cd /usr/src/kernel-hacking
<code>wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.18.4.tar.xz
wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.18.4.tar.xz.sign

tar xf linux-3.18.4.tar.xz
cd linux-3.18.4/drivers/hid

<b>Make a backup</b>
cp hid-input.c{,.bak}

nano (or gedit or vi) hid-input.c and search for HID_GD_HATSWITCH. Look a few lines above it, then change from - to +

                                  map_rel(usage->hid & 0xf);
                         else
 -                                map_abs(usage->hid & 0xf);
 +                                map_abs_clear(usage->hid & 0xf);
            case HID_GD_HATSWITCH:

where - is the original line, changed to what + is. Save the file and quit back to the command line.

To ensure you did it correctly, run

<code> diff hid-input.c hid-input.c.bak

which ought to output something very close to

615c615
<                               map_abs_clear(usage->hid & 0xf);
---
>                               map_abs(usage->hid & 0xf);

If you have Debian/Ubuntu/Mint et al, setup the control files to allow linux to properly setup the kernel:

cd /usr/src/kernel-hacking
git clone git://kernel.ubuntu.com/ubuntu/ubuntu-trusty.git
cp -a /usr/share/kernel-package ubuntu-package
cp ubuntu-trusty/debian/control-scripts/p* ubuntu-package/pkg/image/
cp ubuntu-trusty/debian/control-scripts/headers-postinst ubuntu-package/pkg/headers/

Ok, we're ready to compile.
cd kernel-3.18.4/
make-kpkg clean
time make-kpkg -j `grep -c CPU /proc/cpuinfo` --initrd --overlay-dir=../ubuntu-package kernel_image kernel_headers

Done? Good. Let's install this sucker,

<code>cd ..
<code>dpkg -i linux-*3.18.4*.deb
<code>reboot now

Make sure you're using the right kernel, in terminal type <code>uname -a</code> and make sure it says 3.18.4. Everything should work.
