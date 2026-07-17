# chromebook-linux-optimization
[Repo Link](https://github.com/Xobium/chromebook-linux-optimization)

A comprehensive, straight-to-the-point guide to installing, configuring, and optimizing (L)ubuntu for Chromebooks and low-spec laptops.

"The best way to install, configure and optimize Lubuntu for Chromebooks."

This guide is intended for devices which have a low amount of RAM (e.g. 4GB), and use eMMC or microSD for storage.
It includes aggressive disk space saving tips which are most useful for devices with a small amount of storage (e.g. 16-64 GB).

Most of the steps in this guide are still helpful for devices other than the ones mentioned above (e.g. any other low-spec laptop or newer Chromebooks with more RAM and storage), though some steps may be redundant or need small changes.

This guide will be covering what to do during/after the actual installation of Linux, not how to get rid of ChromeOS and unlock your Chromebook. To install Linux on the Chromebook, you will first need to use the MrChromebox Script to unlock the UEFI.

Read the [MrChromebox Documentation](https://docs.mrchromebox.tech/docs/getting-started.html) for instructions on how to do so.

# Optional First Steps: Run Checks on ChromeOS before installing Linux

This is optional and just for hardware checks. If some hardware function is broken in ChromeOS, it most likely won't work in Linux either so don't waste time troubleshooting it later.

Press Ctrl+Alt+T to open Crosh (ChromeOS Terminal)
- [ ] run `diagnostics`
- [ ] run `battery_test 1`
- [ ] plug a USB drive into every USB-A port, make sure you can access the files
- [ ] plug a device into the USB-C ports that you can use to check whether it properly carries power and transfers data
- [ ] make sure AC adapter works in all compatible ports if multiple support power delivery (e.g. a Chromebook with 2 USB-C charging ports)
- [ ] make sure WiFi connects and stays connected
- [ ] pair and test a bluetooth audio device (earbuds, speaker, etc.), mouse, keyboard, or anything else you might use
- [ ] test for dead pixels or bright/dark spots (you can use [this](https://displaytech.org/en/tests/dead.pixel.htm) website to test)
- [ ] test if every single key works, including function keys
- [ ] test trackpad scroll, click, zoom, and other gestures
- [ ] check speaker quality and volume
- [ ] check if the camera and microphone work

# Lubuntu Optimization Guide
Any Ubuntu-based Linux distribution will work with this guide, but I recommend Lubuntu. Make sure you know what you're doing if you choose to use this guide with a non-Ubuntu-based distribution.

### Installation
- [ ] [Download the ISO](https://lubuntu.me/downloads/) for a recent version of Lubuntu. I recommend downloading the latest LTS version (e.g. [26.04 LTS](https://cdimage.ubuntu.com/lubuntu/releases/26.04/release/)).
- [ ] Burn the ISO to a USB flash drive or other external storage drive using tools like Rufus (Windows), Balena Etcher (Cross-platform) or Ventoy. There are many easy tutorials on how to do this, so I won't get into detail here.
- [ ] Boot into the drive to enter the Lubuntu installer.
- Proceed through the installer until you reach "Installation Mode" (typically in the "Customize" section). Make sure you select **Minimal Installation** here.
- Once you get to the "Partitions" section, it is very important that you choose **Manual Partitioning** so we can use a type of filesystem called BTRFS, which can give you much more usable storage space.

:star: If you aren't familiar with manual partitioning or can't figure out the menu, there are many helpful tutorials online. Just make sure you use the partition sizes and formats from this guide rather than the ones in the tutorial.
- Delete every existing partition until the entire drive is shown as "Free Space".
- Create a 256 MB "/boot/efi" partition formatted as **FAT32**
- Create a "/" partition formatted as **BTRFS** that takes up the rest of the space on the drive
- All done, continue with the installer and make sure to remember the password you set! :)
- [ ] After the installation is completed, log in with the username and password you created.
- You should be met with a clean desktop environment

### System Tuning

- [ ] Open the terminal (called QTerminal on Lubuntu). We will complete the rest of the guide using the terminal.

- [ ] Make system logs live inside RAM to reduce disk writes

Make sure the configuration file exists by running:

```bash
sudo mkdir -p /etc/systemd/journald.conf.d
```
When the terminal asks for a password, just type in your password and press enter (Characters may not show up while you are typing it, that is normal). This authorizes the command to run.

Then add the configuration by running:

```bash
echo -e "[Journal]\nStorage=volatile\nRuntimeMaxUse=64M" | sudo tee /etc/systemd/journald.conf.d/volatile.conf
```

Then restart the service:

```bash
sudo systemctl restart systemd-journald
```

- [ ] **IMPORTANT:** Edit the fstab configuration (`/etc/fstab`) to add filesystem compression and configure other parts of your filesystem

This can significantly increase the amount of usable storage space you have, among other benefits like often faster disk speeds and longer drive lifespan.

To edit the configuration run:

```bash
sudo nano /etc/fstab
```
Navigate the text editor using the arrow keys.

To add zstd compression for root (`/`) AND home (`/home`):
- Find or create the lines for both `/` and `/home` and edit both of them to set the options to `defaults,noatime,compress-force=zstd:1,ssd,space_cache=v2,nodiscard`.

  Those lines should look like this:
  
  ```text
  UUID=[RANDOM CHARACTERS, DON'T EDIT THIS] /         btrfs   subvol=/@,defaults,noatime,compress-force=zstd:1,ssd,space_cache=v2,nodiscard 0 0
  UUID=[RANDOM CHARACTERS, DON'T EDIT THIS] /home     btrfs   subvol=/@home,defaults,noatime,compress-force=zstd:1,ssd,space_cache=v2,nodiscard 0 0
  ```

Now, to lower disk usage and optimize writes in certain directories, we will add a couple more lines:
- One for `/tmp` that lives in memory with a size limit of 512 MB.
- And another for `/var/cache/apt/archives`, in memory with a size limit of 2 GB.

Add these lines to the bottom of the fstab. If the line for `/tmp` already exists, just edit it to match:

  ```text
  tmpfs     /tmp                     tmpfs   defaults,noatime,mode=1777,size=512M 0 0
  tmpfs     /var/cache/apt/archives  tmpfs   defaults,noatime,mode=0755,size=2G   0 0
  ```
  
  Once that's done, the full `/etc/fstab` file should look like this:
  
  ```text
  UUID=[RANDOM CHARACTERS, DON'T EDIT THIS] /boot/efi   vfat    defaults   0 2
  UUID=[RANDOM CHARACTERS, DON'T EDIT THIS] /           btrfs   subvol=/@,defaults,noatime,compress-force=zstd:1,ssd,space_cache=v2,nodiscard 0 0
  UUID=[RANDOM CHARACTERS, DON'T EDIT THIS] /home       btrfs   subvol=/@home,defaults,noatime,compress-force=zstd:1,ssd,space_cache=v2,nodiscard 0 0
  tmpfs     /tmp                     tmpfs   defaults,noatime,mode=1777,size=512M 0 0
  tmpfs     /var/cache/apt/archives  tmpfs   defaults,noatime,mode=0755,size=2G   0 0
  ```

Save the file (Ctrl+O, Enter) and close the editor (Ctrl+X).

- [ ] **IMPORTANT**: Configure APT to clear its cache after upgrades are complete

Since we configured the APT cache to live inside memory, we want to make sure it never uses up _too much_ of your RAM.

To create a configuration file run:

```bash
sudo nano /etc/apt/apt.conf.d/00no-cache
```

Add these lines to the file:

```text
APT::Keep-Downloaded-Packages "false";
Binary::apt::APT::Keep-Downloaded-Packages "0";

# Ensures the APT cache folder exists on boot
APT::Get::Pre-Invoke { "mkdir -p /var/cache/apt/archives/partial"; };
```

Save the file (Ctrl+O, Enter) and close the editor (Ctrl+X).

Reboot now.

- [ ] Compress already-existing files to claim back some storage space

First, to see how much storage space the system is already using, run:

```bash
sudo btrfs filesystem usage /
```

Note down or save the output. We will use it in a little bit.

Run this command to apply compression to existing system files:

```bash
sudo btrfs filesystem defragment -r -czstd / /home
```

This should take a little while, don't close the terminal or turn off the laptop.

After the command completes, run this command again:

```bash
sudo btrfs filesystem usage /
```

Compare the new output with the old one. Look at the `Used: X.XX GiB` line specifically.

You should see that you now have less used storage space (and therefore more free space) due to filesystem compression being applied.
This is what gives you extra usable storage space.

- [ ] Further reduce unnecessary disk writes

Create a configuration file by running:

```bash
sudo nano /etc/sysctl.d/99-tuning.conf

```

Add the following lines:

```text
vm.vfs_cache_pressure=75
vm.dirty_background_ratio=3
vm.dirty_ratio=7
vm.page-cluster=0
```
Save the file (Ctrl+O, Enter) and close the editor (Ctrl+X).

Apply the changes by running:

```bash
sudo sysctl --system
```

- [ ] Uninstall large preinstalled apps like LibreOffice, Thunderbird, VLC, etc. (they shouldn't even be installed if you chose Minimal Installation earlier, so you can skip this if they aren't there)
- [ ] Enable ZRAM

ZRAM gives you extra usable RAM by compressing the data inside your laptop's memory in real-time when memory runs low. This can give you a lot more usable memory than the physical amount of RAM in your device.

We will use the `zstd` algorithm with 200% of physical ram capacity for 4 GB devices. 100% is a good setting for 8 GB devices.

To install the tool run:

```bash
sudo apt update && sudo apt install zram-tools
```

To edit its configuration run:

```bash
sudo nano /etc/default/zramswap
```

Add to or edit the file to say the following:

```text
ALGO=zstd
PERCENT=200
PRIORITY=999
```
Make sure the line saying `SIZE=XXX` has a hashtag (#) infront of it

Save the file (Ctrl+O, Enter) and close the editor (Ctrl+X).

To apply the changes run:

```bash
sudo systemctl restart zramswap.service

```

To verify the changes applied properly run:

```bash
zramctl

```

You should see an output that labels "ALGORITHM" as "zstd" and "DISKSIZE" as approximately 8 GB (unless you used a different percentage in the configuration, which is fine as well!)
- [ ] Disable disk/eMMC swap so the system only uses the ZRAM swap we just created
      
Edit the fstab configuration:

```bash
sudo nano /etc/fstab

```

Find something like `UUID=xxxxxxxx-xxxx-... none swap sw 0 0` or `/swapfile none swap sw 0 0`.

Comment those lines out by adding a hashtag (#) to the very beginning of the line.

Save the file (Ctrl+O, Enter) and close the editor (Ctrl+X).

To apply the changes, run:

```bash
sudo swapoff -a

```

- [ ] Increase the system's "swappiness" value.

This makes it more willing to use the new ZRAM we just set up. We want this because ZRAM is incredibly fast compared to the default disk swap setup that we disabled earlier.

To check the value run:

```bash
cat /proc/sys/vm/swappiness

```

To change the value, create a configuration file:

```bash
sudo nano /etc/sysctl.d/99-swappiness.conf

```

Add `vm.swappiness=180` to the file to set swappiness to 180.

Save the file and close the editor.

To apply the changes run:

```bash
sudo sysctl --system

```

- [ ] (Optional) Uninstall Snap packages and disable `snapd`.
  
Because Snap packages use much more storage space than native packages, removing them can save lots of space if you don't rely on Snap packages.

FIRST RUN:

```bash
snap list

```

AND UNINSTALL THEM ALL (by running `sudo snap remove --purge [insert name here]`) BEFORE REMOVING SNAP ITSELF

To disable snap run:

```bash
sudo systemctl disable --now snapd.service

```

To uninstall snap run:

```bash
sudo apt purge --autoremove snapd

```

- [ ] Look through startup apps (`ls ~/.config/autostart`) and remove apps that you don't need to start up automatically. If the file is empty you are already good.
- [ ] Enable automatic fstrim (Standard step for BTRFS, can improve disk performance and lifespan)
      
To enable it run:

```bash
sudo systemctl enable --now fstrim.timer

```

To verify it was successfully enabled run:

```bash
sudo systemctl status fstrim.timer

```

You should see green text saying "active", and you should hopefully *not* see "inactive" or red text saying "failed"
- [ ] Change the I/O scheduler to "mq-deadline" if it isn't already and make the change permanent

First check which scheduler is currently being used by running:

```bash
cat /sys/block/mmcblk*/queue/scheduler

```

If it is already set to mq-deadline (if its name is the only one shown or if it is inside [square brackets]), change nothing.

If it is set to anything else, you can change it:

Create a udev rule:

```bash
sudo nano /etc/udev/rules.d/60-emmc-scheduler.rules

```

Paste the following into the file:

```text
ACTION=="add|change", SUBSYSTEM=="block", ENV{DEVTYPE}=="disk", KERNEL=="mmcblk*", ATTR{queue/scheduler}="mq-deadline"

```

Save the file and close the editor.

Reboot.
- [ ] Remove preinstalled files you don't need and clear caches to save some more storage space

- Remove large backups of old Linux kernel versions
  
To check the current kernel version run:

```bash
uname -r

```

Write this down or remember its version number so you don't delete it later.

To list all installed kernels run:

```bash
dpkg --list 'linux-image*' | grep ^ii

```

To remove old kernels run:

```bash
sudo apt remove --purge linux-image-X.XX.X-XX-generic

```

Replace the "X"s with the number of the old version you want to remove. **Do not delete the current kernel version that you remembered from the previous step**.

Once you have removed the old kernels run:

```bash
sudo update-grub

```

- Now do the same thing but remove old kernel headers

To list all installed headers run:

```bash
dpkg --list 'linux-headers*' | grep ^ii

```

To remove old headers run:

```bash
sudo apt remove --purge linux-headers-X.XX.X-XX

```

Once again, don't delete the headers corresponding to your current kernel version.

Then run:

```bash
sudo apt autoremove

```

- Prune locales (translations for other languages) that you won't use

To install a package that makes it easy to do this, run:

```bash
sudo apt install localepurge

```

Then run:

```bash
sudo localepurge

```

Follow the instructions on screen to remove the languages you won't use.

- Remove man (manual) pages if you don't use them

Create a configuration file:

```bash
sudo nano /etc/dpkg/dpkg.cfg.d/01_nodoc

```

Add the following to the file:

```text
path-exclude=/usr/share/doc/*
path-exclude=/usr/share/man/*
path-exclude=/usr/share/info/*
path-include=/usr/share/doc/*/copyright

```

Save the file and close the editor.

To remove the files run:

```bash
sudo rm -rf /usr/share/doc/* /usr/share/man/* /usr/share/info/*

```

- Thumbnails cache, totally safe to clear whenever you want to free up some space.

```bash
rm -rf ~/.cache/thumbnails/*

```

- Run `sudo apt clean` and `sudo apt autoremove` regularly if using apt often

- [ ] Install the laptop power management package

To install it run:

```bash
sudo apt install tlp tlp-rdw && sudo systemctl enable tlp.service

```

To start it run:

```bash
sudo systemctl start tlp.service && sudo tlp start

```

The default configuration is usually good, but you can configure it further if interested using `sudo nano /etc/tlp.conf`.

- [ ] Speed up boot time by reducing GRUB timeout

To edit the configuration run:

```bash
sudo nano /etc/default/grub

```

- Set `GRUB_TIMEOUT=1` and `GRUB_TIMEOUT_STYLE=hidden`
- Save the file (Ctrl+O, Enter) and close the editor (Ctrl+X).
- To apply the changes, run:

```bash
sudo update-grub

```
---
That's the end of the system tuning! You may want to reboot your device at this point, _just in case_ some changes haven't applied yet.

Your system is now super clean and lightweight and uses very little storage and memory!

In my case, it uses about 500 MB of RAM on startup, and under 6 GB of _total_ storage, even with some extra apps installed.

I hope you were able to follow all the steps alright, and I am happy to help with any questions or problems you have. Just open an issue or discussion :)

---

# Browser Tuning

Tweaks for Firefox and Chromium-based browsers.

These tweaks are a bit more complicated and also less important. Feel free to skip these if they aren't important to you.

If hardware accelerated video decoding isn't working and you want to fix it, run (ONLY IF YOUR DEVICE HAS AN INTEL CPU):

```bash
sudo apt install intel-media-va-driver-non-free vainfo

```

Then run:

```bash
vainfo

```

### Firefox

**Extensions**

`enhanced-h264ify`. Many older Chromebooks have CPUs that don't support the modern AV1 video codec well. You can use this extension to force sites like YouTube to give you videos in a different format.

`U-Block Origin`

(If you removed snap earlier, you may need to reinstall Firefox using a .deb PPA; look it up)
- [ ] Go to about:config inside the URL bar and set:

```text
media.ffmpeg.vaapi.enabled = true
browser.cache.disk.enable = false
browser.cache.memory.enable = true
browser.cache.memory.capacity = 131072

```

### Chromium-Based browsers

**Extensions**
`enhanced-h264ify` (Chrome Web Store). Many older Chromebooks have CPUs that don't support the modern AV1 video codec well. You can use this extension to force sites like YouTube to give you videos in a different format.
`U-Block Origin` (unless you are using a browser with built in ad/tracker blocking)

Add flags to the app's .desktop entry:
- Find the browser's .desktop file in `~/.local/share/applications`
If it isn't there, find it in `/usr/share/applications/` and copy it over:

```bash
cp /usr/share/applications/[insert name here].desktop ~/.local/share/applications/[insert name here].desktop

```

Find the line starting with Exec= and append these flags to the end of that line: `--use-gl=desktop --enable-features=VaapiVideoDecoder --disable-features=UseChromeOSDirectVideoDecoder`

To apply the changes run:
```bash
update-desktop-database ~/.local/share/applications

```

---

If audio is broken in Lubuntu you may be able to fix it using these commands:

```bash
sudo apt update && sudo apt install linux-firmware curl git
cd && git clone https://github.com/WeirdTreeThing/chromebook-linux-audio
cd chromebook-linux-audio
./setup-audio

```

If that successfully fixes it, you can remove the files you downloaded by running:

```bash
rm -rf ~/chromebook-linux-audio

```

### Recommended Apps

abiword - Smaller replacement for LibreOffice or Microsoft Office

Featherpad - Lubuntu text editor, preinstalled

Celluloid - Video and audio player

LXImage-Qt - Lubuntu image viewer, preinstalled

Geary - E-Mail Client. Smaller replacement for Thunderbird or other E-Mail apps

Qpdfview - PDF viewer. Using a browser is also good

Galculator - Lubuntu Calculator, preinstalled

screengrab - Lubuntu screenshot tool, preinstalled
