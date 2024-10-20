## Contents

* [Android](#android)
* [Archives](#archives)
* [Audio](#audio)
* [Bootloader](#botloader)
* [Compression](#compression)
* [Docker](#docker)
* [Dropbox](#dropbox)
* [File conversions](#file-conversions)
* [File manipulation](#file-manipulation)
* [File synchronization](#file-synchronization)
* [Filesystems](#filesystems)
* [Find](#find)
* [Fonts](#fonts)
* [git](#git)
* [Image manipulation](#image-manipulation)
* [Keyboard](#keyboard)
* [Mouse](#mouse)
* [Networking](#networking)
* [Programming](#programming)
* [Screen](#screen)
* [Shell](#shell)
* [Software](#software)
* [SSH](#ssh)
* [System config](#System-config)
* [Touchpad](#touchpad)
* [Video manipulation](#video-manipulation)
* [Virtualization](#virtualization)

# Android

## Reliably transfer files android <-> linux
To reliably transfer files from/to android <-> linux, use `android-file-transfer`: [https://github.com/whoozle/android-file-transfer-linux](https://github.com/whoozle/android-file-transfer-linux). On Ubuntu/Debian:
```
sudo apt-get intall android-file-transfer
```

# Archives

## To create a tar.gz archive and preserve permissions and symlinks
sudo tar -p -z -chf archive.tar.gz files

## To restore archive above with original links
sudo tar -p -xhf archive.tar.gz

# Audio

## Extract the audio track from a video

1. Extract the audio stream and copy it into the new file (m4a is the mpeg4 container for audio data only):
```
ffmpeg -i video.mp4 -map 0:a -acodec copy audio.m4a
```

2. Extract the audio stream and re-encode it into a mp3 file:

```
ffmpeg -i video.mp4 -map 0:a -acodec libmp3lame audio.mp3
```

## Commands to control the volume (alsa)
Increase volume
```
amixer -D pulse set Master 2%+
```
Decrease volume
```
amixer -D pulse set Master 2%-
```
Toggle output 
```
amixer -D pulse set Master toggle
```

## Keyboard shortcuts to control music player

Most music players follow the MPRIS specification (https://wiki.archlinux.org/title/MPRIS).

The `playerctl` tool can be used to control the player, e.g.:

- `playerctl play-pause`
- `playerctl next`
- `playerctl previous`

## Fix the no sound issue with SND HDA Intel (Ubuntu)

https://www.linuxuprising.com/2018/06/fix-no-sound-dummy-output-issue-in.html

# Bootloader

## Change GRUB's menu timeout

To set timeout to 5 seconds, add the following to `/etc/default/grub`:
```
GRUB_RECORDFAIL_TIMEOUT=5
```

# Compression

## Reduce an executable's size using live-compression

Particularly useful with go binaries as they tend to be quite large.
Require the `upx-ucl` package. Upx compresses an executable and embeds the decompressor into it:
```
sudo apt-get install upx-ucl
upx <executable>
```

# Docker

## To remove all containers
```
docker ps -a|grep -v CONTAINER|awk {'print $1'}|xargs docker rm
```

## To remove all \<none\> images
```
docker images|grep -v IMAGE|grep "<none>"|awk {'print $3'}|xargs docker rmi
```

## If network access in containers seems to be broken
```
systemctl stop docker
systemctl stop docker.socket
ifconfig docker0 down
brctl delbr docker0
systemctl start docker
```

# Dropbox

## Mark files/directories to be ignored by Dropbox

Dropbox uses file extended attributes for various things.
A specific attribute "user.com.dropbox.ignored" tells Dropbox to ignore a file or directory (and all its content).

To mark a file/directory to be ignored:

```
setfattr -n user.com.dropbox.ignored -v 1 my_file
```

To remove the mark:

```
setfattr -x user.com.dropbox.ignored my_file
```

## Recursively mark all directories ending in ".git" to be ignored by Dropbox:

```
find ~/Dropbox -type d -name "*.git" -exec setfattr -n user.com.dropbox.ignored -v 1 {} \;
```

# File conversions

## Remove trailing spaces at the end of a line
sed -i 's/[[:space:]]*$//' source.c

## PDF conversion

Sometimes PDF are not readable by all software. In this case, Ghostscript can help: it can read the file and write it in a more compatible format:
```
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/prepress -dNOPAUSE -dQUIET -dBATCH -soutput.pdf input.pdf
```

## Convert images to PDF using ImageMagick

ImageMagick's convert utility can convert images to PDF. However, converting to PDF is disabled/forbidden by default on Ubuntu.

To enable it:

Edit `/etc/ImageMagick-N/policy.xml` (where `N` is your ImageMagick version) and change:

```
<policy domain="coder" rights="none" pattern="PDF" />
```

to:

```
<policy domain="coder" rights="read|write" pattern="PDF" />
```

# File manipulation

## To undelete files (note: seems to work better than `photorec` for jpegs)
```
magicrescue -r jpeg-jfif -r jpeg-exif -d ~/output /dev/hdb1
```

## To recover photos and videos
```
photorec
```

## To delete all duplicate files in the current dir
```
fdupes . -d -N
```

## Convert a file from UTF-8 to ISO-8859-15:
```
recode UTF8..ISO-8859-15 file.txt
```

## Convert newlines from LF (UNIX) to CR-LF (DOS):
```
recode ../CR-LF file.txt
```

## To convert txt files to ps
Note: a2ps cannot read UTF-8 files! (use recode to convert from UTF-8)
```
a2ps --columns=1 -l165 -o output.ps input.txt
```

## To replace multiple occurences of space in `myfile`
```
cat myfile|tr -s \
```

# File synchronization

## Mirror a directory to an external drive
To mirror the `/home/joe/work directory` to the external drive mounted in `/media/joe/sdd_drive/`, run as root:
```
rsync -r -H -p -E -X -o -g --specials --devices -l -t -U /home/joe/work /media/joe/sdd_drive/work/
```

## To rsync movies from PC to tablet
Mount the tablet and open a terminal in its Videos folder, then type:
```
rsync -ra --delete  ~/Videos/Cartoons/ .
```

# Filesystems

## Issues mounting an NTF partition
If you get the following message when trying to mount an ntfs partition:
```
Windows is hibernated, refused to mount.
Failed to mount '/dev/sdj2': Operation not permitted
The NTFS partition is in an unsafe state. Please resume and shutdown
Windows fully (no hibernation or fast restarting), or mount the volume
read-only with the 'ro' mount option.
```
This mount option should fix it:
```
mount -o remove_hiberfile /dev/sdj2 mnt
```

## Best way to display mountpoints and their hierarchy
```
findmnt
```

# Find

## Recursively search all occurences of "fork" in all "*.c" files from the current directory

```
find . -name "*.c" -exec grep -H fork {} \;
```

## Same as above with grep only (also prints the line number); add "-i" to be can insensitive on the "fork" pattern

```
grep -rn fork --include=*.c
```

## Print the file name and exec more than one command

find . -name "*.c" -type f -printf "%p\n" -exec cat {} \; -exec echo "" \;

## To pass multiple arguments to find
```
find . -iname "*.c" -o -iname "*.h"
```

## Locate all SUID binaries
```
find / -perm -4000 2>/dev/null
```

## To locate all files writable by anyone
```
sudo find / -perm -o+w
```

## Display files whose' status have changed in the last 10 hours
```
find . -type f -cmin -600 
```

## Display files that have been created or changed in the last 10 hours
```
find . -type f -mtime -10
```

## Delete all files in /tmp that are older than 1 day
find /tmp -type f -mtime +1 -exec rm -f {} \;

## To find files in /home within a certain date range (between June 11th 2010 and June 18th 2010)
```
touch --date "2010-06-11" /tmp/start
touch --date "2010-06-18" /tmp/end
find /home -type f -newer /tmp/start -not -newer /tmp/end
```

## To delete all .txt files under 10k
```
find -iname "*.txt" -size -10k -delete
```

# Fonts

## To install and register ttf fonts (per user):
Create the .fonts directory and copy the ttf file in it:
```
mkdir ~/.fonts
```
Then update the font cache:
```
sudo fc-cache -fv
```

# git

## To throw discard local commits

```
git reset --hard origin/<branch name>
```

## To display the repository's remote path (url)

```
git remote -v
```

## To update a repository's remote path

1. Check the repository's current path:
```
$ git remote -v
origin	ssh://git@ssh.hesge.ch:10572/flg_courses/virtualization.git (fetch)
origin	ssh://git@ssh.hesge.ch:10572/flg_courses/virtualization.git (push)
```

1. Update the repository with the new path:
```
$ git remote set-url origin git@ssh.hesge.ch:10572/flg_courses/virtualization/virtualization.git
```

## Checkout a git repository before a given date

To checkout a git repository before a given date:

```
git rev-list -n1 --before="2020-06-25 08:00" master | xargs git checkout
```

# Image manipulation

## Crop a 1000x400 pixels area at coordinates 1700,900 (uses imagemagick)
```
convert -crop 1000x400+1700+900 src.jpg dest.jpg
```

## To rotate or flip a jpeg losslessly
```
jpegtran -rotate 90 image.jpg
jpegtran -flip horizontal image.jpg
```

# Keyboard

## Set keyboard delay (in ms) and repetition rate (char/sec)
```
xset r rate 250 50
```

## Force an update of the keyboard config
Ubuntu:
```
dpkg-reconfigure keyboard-configuration
systemctl restart keyboard-setup.service
```
Debian:
```
dpkg-reconfigure keyboard-configuration
apt-get install console-setup
# or if already installed: dpkg-reconfigure console-setup
```

## Add useful key bindings to clear screen, move cursor, etc. (ctrl+l, ctrl+e, etc.)

If bash is set to use vi mode (with `set -o vi`), ctrl+l, ctrl+e, etc. won't work as these are emacs key bindings. To use these key bindings, set bash to emacs mode (default) with:

```
set -o emacs
```

Also, key bindings can be added/changed system-wide by modifying `/etc/inputrc`:
```
"\C-l": clear-screen
"\C-k": kill-line
"\C-a": beginning-of-line
"\C-e": end-of-line
```
You can also display the current bindings with `bind -P`

# Mouse

## Disable middle button

Install sxhkd:
```
sudo apt-get install sxhkd
```

Create `~/.config/sxhkd/sxhkdrc` with the following content:
```
button2
     :
```

Create `~.config/autostart/sxhkd.desktop` with the content below. This desktop autostart file starts sxhkd when user logs into their desktop environment:
```
[Desktop Entry]
Type=Application
Name=sxhkd
Exec=sxhkd
Comment=disable middle mouse button
StartupNotify=false
Terminal=false
Hidden=false
```

# Networking

## To list wireless devices
```
rfkill list

```

## To unblock all wireless devices
```
sudo rfkill unblock all
```

## To list network interfaces managed by NetworkManager and their states:
```
nmcli
```

## To activate an interface that's listed as "unavailable" in NetworkManager: 
```
nmcli r wifi on
```

## To scan all open http ports on a given subnetwork
```
nmap -p 80 192.168.1.*
```

## iptables: to block TCP traffic for user joe
```
iptables -A OUTPUT -p tcp -m owner --uid-owner joe -j DROP
```

## To remove all rules from iptables
```
iptables -F
```

# Programming

## C formatter for VSCodium/VSCode

First install clang-format on the system:
```
sudo apt-get install clang-format -y
```

Then, in VSCodium:
- Install "Clang-Format" extension
- Press "Ctrl + Shift + i" to indent the current file


## To generate a static binary with golang
```
CGO_ENABLED=0 GOOS=linux go build -a -ldflags="-s -w" -installsuffix cgo
```

## In C, one can pass an array "in place" as an argument:

```
void printtab(int *tab, int size) {
    for (int i = 0; i < size; i++)
        printf("%d\n", tab[i]);
}

void main() {
    printtab((int[]){ 13,5,9,-10}, 4);
}
```

## In C, how to get the offset in bytes of a field in a structure
```
offsetof(struct xxx, field)
```

## In C, to printf a string that doesn't end with 0
```
char t[] = {'a','b','c'}; // does NOT have a 0 terminal
printf("%s*\n", t, 3);
```

## To find errno error numbers
```
apt-get install moreutils
```
Then, use the `errno` command
```
$errno 95
EOPNOTSUPP 95 Operation not supported
```

## make: how to autogenerate the help, example of a Makefile:
```
help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

pdf: $(SRCS)  ## Generate pdf files
	pandoc ....

clean:  ## Delete generated pdf files
	rm -f *.pdf
```

# Screen

## Reset screen resolution to default
```
xrandr -s 0
```

## Turn off screen
```
# standby, suspend, off
xset dpms force standby
```

## Change screen brightness
Retrieve the device to change the brightness for:
```
xrandr -q|grep " con"
```
Change XXX's brightness (from 0.0 to 1.0):
```
xrandr --output XXX --brightness 1.0
```

# Shell

## To repeat the last command (bash)
```
!!
```

## To recall the argument of the last command (bash)
```
ls yoyo
vi !$
```

## Replace "ls" in the last command by "tree" (bash)
```
^ls^tree^
```

## To ignore a command's alias, e.g. rm here (bash)
```
\rm
```

## Clear bash history
```
history -c
history -w
```

## Check whether a file exists (bash)
The -a argument means "AND"
```
if [ -f file1 -a -f file2 ]; then
  ...
```

## filename manipulation (bash)
```
path=/home/blah/pipo.c
filename=$(basename "$path")
extension="${filename##*.}"
filename_without_extension="${filename%.*}"
```

## Make "q" in man to not clear the screen
```
export LESS='iX'
```

## Disable the "beep" sound when pressing backspace and there is nothing to delete
```
xset b off
```

## Display a file hierarchy in a pretty form
Default character set (non 7-bit ASCII):
```
tree
```
Since LaTeX/pandoc doesn't like the default charset, here is to specify basic ASCII characters:
```
tree --charset ascii
```

## Use cut to remove the first 28 characters from stdin
```
cut -c 28-
```

## Display all files in the current dir (and sub-dirs) from the largest to the smallest
```
ls -laS *|grep -s ^-|cut -c 28-|sort -nr|more
```

# Software

 ## PDF Annotation

 ### Best apps
- xournalpp [https://github.com/xournalpp/xournalpp](https://github.com/xournalpp/xournalpp)

## Video Editing

### Best apps
- ffmpeg [https://ffmpeg.org/](https://ffmpeg.org/)
	- 19 FFmpeg Commands For All Needs: https://catswhocode.com/ffmpeg-commands/
- OpenShot [https://www.openshot.org/](https://www.openshot.org/)
- Kdenlive [https://kdenlive.org/en/](https://kdenlive.org/en/)
- Lossless-cut [https://github.com/mifi/lossless-cut](https://github.com/mifi/lossless-cut)

## Screen Recording/Streaming

### Best apps
- OBS Studio [https://obsproject.com/](https://obsproject.com/)
- SimpleScreenRecorder [https://www.maartenbaert.be/simplescreenrecorder/](https://www.maartenbaert.be/simplescreenrecorder/)
- Kazam [https://github.com/hzbd/kazam](https://github.com/hzbd/kazam)
- VokoscreenNG [https://github.com/vkohaupt/vokoscreenNG](https://github.com/vkohaupt/vokoscreenNG)

## Video Conferencing

### Best apps
- Jitsi [https://jitsi.org/](https://jitsi.org/)
- BigBlueButton [https://bigbluebutton.org/](https://bigbluebutton.org/)
- [https://obs.ninja](https://obs.ninja)

## Code Editing

### Best apps
- vim [https://www.vim.org/](https://www.vim.org/)
- Geany [https://www.geany.org/](https://www.geany.org/)
- VSCodium [https://vscodium.com/](https://vscodium.com/)
- IntelliJ IDEA [https://www.jetbrains.com/idea/](https://www.jetbrains.com/idea/)

### vim: convert tabs into spaces
```
:set expandtab
:set tabstop=4
:retab
```

### vscode: to preview a markdown (`.md`) file
- Press "ctrl+alt+v"

## File Manager

### Nautilus to stop tracking recent files
```
sudo apt-get install activity-log-manager
turn off "Record file and application usage"
```

## Debuggers (gdb frontends)

- gdbgui [https://www.gdbgui.com/](https://www.gdbgui.com/)

# SSH

## If you get an error authenticating using a ssh keypair
For instance github would output the following error message: "Permission denied (publickey).".
The ssh client doesn't find the right private key. There 2 solutions:

1. Explicitely tell ssh which key to use with the `-i` option:
```
ssh -i ~/.ssh/the_priv_key ...
```
1. Add the key to ssh-agent:
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/the_private_key
```

# System config

## To disable Ubuntu "major update" annoying recurrent popup

Add the following to `/etc/update-manager/release-upgrades`;
```
Prompt=never
```

## Switching boot target to text

Obtain default target:
```
systemctl get-default
```

Switch to text mode:
```
systemctl set-default multi-user.target
reboot
```

## Disable auto-mounting of hot-plugged devices (usb keys, etc.)

The service dealing with automounting filesystems at runtime is `udisks2`. To disable it temporarily (until next reboot):
```
systemctl stop udisks2
```

To disable it permanently (across reboots):
```
systemctl disable udisks2
```

## For current shell's variables to be visible when using sudo

```
sudo -E
```

## Setup autologin with LightDM (XFCE)

Add the following to `/etc/lightdm/lightdm.conf` under `[Seat:*]` where `xxx` is the user that should automatically logs in: 
```
autologin-user=xxx
autologin-user-timeout=0
```

## Display kernel error and warning messages

```
dmesg -T -lerr,warn -H
```

## Activate autologin on Xubuntu (lightdm)

Create /etc/lightdm/lightdm.conf.d/lightdm.conf with:
```
[Seat:*]
autologin-user=YOUR_USER
autologin-user-timeout=0
greeter-hide-users=true
```

## Prevent services from spamming the logs

Some services write tons of messages to the system logs. Here we change the verbosity of the rtkit-daemon.service to warning:

1. Create a system-wide config dir for the service and inside, create a log config file:
   ```
   /etc/systemd/system/rtkit-daemon.service.d/log.conf
   ```
1. In `log.conf`, set the desired loglevel (0=emergency, 1=alert, 2=critical, 3=error, 4=warning, 5=notice, 6=info, 7=debug):
   ```
   [Service]
   LogLevelMax=4
   ```
1. Reload the daeon and restart the service:
   ```
   systemctl daemon-reload
   systemctl restart rtkit-daemon.service
   ```

## Clear the path cache to executables

Bash caches the path to executables found in the PATH variable environment. If you remove a program that was installed in the PATH, bash might complain it doesn't find it in the cached directory.

This cache can be cleared with:

```
hash -r
```

or to clear a single entry (where the program is `go`):

```
hash -d go
```

## Disable unattended-upgrades

To prevent the system from automatically downloading and installing updates:

```
sudo dpkg-reconfigure unattended-upgrades
```

## Issue: touchpad not working after suspend

The fix depends on the touchpad.

Solution 1:
```
modprobe -r i2c_hid && modprobe i2c_hid
```

```
modprobe -r psmouse && modprobe psmouse
```

Even better: you can create a systemd service that runs after a suspend to enable the touchpad again:

https://askubuntu.com/questions/1124045/touchpad-scroll-not-working-properly-after-suspend/1125645


## Systemd essentials

[https://www.digitalocean.com/community/tutorials/systemd-essentials-working-with-services-units-and-the-journal](https://www.digitalocean.com/community/tutorials/systemd-essentials-working-with-services-units-and-the-journal)

## To prevent suspend when the lid is closed
Add the following to `/etc/systemd/logind.conf`
```
[Login] 
HandleLidSwitch=ignore 
HandleLidSwitchDocked=ignore
```
Source [here](https://ostechnix.com/linux-tips-disable-suspend-and-hibernation/)

## To make text boot the default under systemd
```
systemctl set-default multi-user.target
```

## To make GUI boot the default under systemd
```
systemctl set-default graphical.target
```

## To blacklist a module from the boot process
- add it to: `/etc/modprobe.d/blacklist.conf`
- regenerate the initramfs: `update-initramfs -u`

## To set hostname on "modern" distros
```
hostnamectl set-hostname xyz
```
And if on a server distrib, edit `/etc/cloud/cloud.cfg` and change:
```
preserve_hostname: true
```
to:
```
preserve_hostname: false
```

## GPT disks
Use `gdisk` instead of `fdisk`

## Get UUID of a device
```
blkid /dev/sda1
```

## To retrieve the current virtual desktop number
```
n=`xprop -root _NET_CURRENT_DESKTOP`
```
## To get the PID of a process
- `pidof`: returns a list if more than one process of this name
- `pgrep`: returns only 1 value even if more than one process of the same name

## To get information about a link
```
readlink
```

## See all thread of a given multi-threaded process
```
ps -L PID
```

## To force a package to overwrite files from another already installed package
- **CAUTION!**
```
sudo apt-get -o Dpkg::Options::="--force-overwrite" install new_xxx_package
```

## To change the timezone
create a symbolic link from desired timezone /usr/share/zoneinfo to /etc/localtime
```
ln -s /usr/share/zoneinfo/Europe/Zurich /etc/localtime
```

## Enable autologin from the console (on tty2 for instance)
- Check the current login command for tty2 with
```
systemctl cat getty@tty2 | grep Exec
```
- Override the ExecStart setting for service getty on tty2
```
sudo systemctl edit getty@tty2
```
- Then add (just added "-a joe" to the output of the systemctl cat command above)
```
[Service]
ExecStart=
ExecStart=-/sbin/agetty -a joe --noclear %I $TERM
```
- Pressing ctrl+alt+F2 should now automatically log in user joe!
- Source: [https://askubuntu.com/questions/659267/how-do-i-override-or-configure-systemd-services](https://askubuntu.com/questions/659267/how-do-i-override-or-configure-systemd-services)

# Touchpad

## To inspect touchpad settings
```
synclient
```

## Command to disable/enable the touchpad
```
synclient TouchpadOff=1
synclient TouchpadOff=0
```

## To disable vertical edge scrolling
```
synclient VertEdgeScroll=0
```

## To turn off, suspend or standby the screen
```
xset dpms force off
xset dpms force suspend
xset dpms force standby
```

# Video manipulation

## To rotate a video
- 0 = 90° counter clockwise + vertical flip
- 1 = 90° clockwise
- 2 = 90° counter clockwise
- 3 = 90° clockwise + vertical flip
```
ffmpeg -i input.mp4 -vf 'transpose=1' -c:v libx264 -c:a copy -crf 20 output.mp4
```

## To trim a video file without re-encoding
Trim the video file to keep everything between 1min 17sec to 1h 5min and 33sec:
```
ffmpeg -i input.mp4 -ss 00:01:17 -to 01:05:33 -c:v copy -c:a copy output.mp4
```

## To concatenate several videos into a single video file (without reencoding)
```
ffmpeg -f concat -i files.txt -c copy output.mp4
```
Where `files.txt` contains the list of video files to concatenate, in order:
```
file 'vid1.mp4'
file 'vid2.mp4'
file 'vid3.mp4'
```
More details [here](https://trac.ffmpeg.org/wiki/Concatenate).

# Virtualization

## To create a qcow2 image file (linux.qcow) from a physical bootable disk (e.g. /dev/sda) with `qemu-img`
```
qemu-img.exe convert -f raw -O qcow2 /dev/sda linux.qcow2
```

## To create a vmdk image file (linux.vmdk) from a physical bootable disk (e.g. /dev/sda) with VirtualBox
```
vboxmanage internalcommands createrawvmdk -filename linux.vmdk -rawdisk /dev/sda
```

source [here](http://www.howtogeek.com/187721/how-to-boot-from-a-usb-drive-in-virtualbox/)

## To fix VirtualBox's "UUID already exists" errors
```
vboxmanage internalcommands sethduuid my_vm_disk.vmdk
```
