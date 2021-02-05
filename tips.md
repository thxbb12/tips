# Shell

## To repeat the last command
```
!!
```

## To recall the argument of the last command (bash)
```
ls yoyo
vi !$
```

## Replace "ls" in the last command by "tree" 
```
^ls^tree^
```

## To ignore a command's alias, e.g. rm here
```
\rm
```

## Clear bash history
```
history -c
history -w
```

## Check whether a file exists in bash
The -a argument means "AND"
```
if [ -f file1 -a -f file2 ]; then
  ...
```

## bash: filename manipulation
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

# Networking

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

# Archives/file Synchronization

## To create a tar.gz archive and preserve permissions and symlinks
sudo tar -p -z -chf archive.tar.gz files

### To restore archive above with original links
sudo tar -p -xhf archive.tar.gz

## To rsync movies from PC to tablet
Mount the tablet and open a terminal in its Cartoon folder (e.g. `/run/user/1000/gvfs/mtp:host=%5Busb%3A003%2C033%5D/Card/Vidéos/Cartoons`),
then type:
```
rsync -rav --delete  ~/Videos/Cartoons/ .
```

# System Config

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

# Find

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

# File Systems

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

# Keyboard

## Set keyboard delay (in ms) and repetition rate (char/sec)
```
xset r rate 200 30
```

## Force an update of the keyboard config
```
dpkg-reconfigure keyboard-configuration
systemctl restart keyboard-setup.service
```

# Touchpad

## Command to disable/enable the touchpad
```
synclient TouchpadOff=1
synclient TouchpadOff=0
```

# SSH

## If you get an error authenticating using a ssh keypair
For instance github would output the following error message: "Permission denied (publickey).".
This is because the ssh-agent cannot find the right private key on your system.
Add the private key manually with `ssh-add PATH` where PATH is the path to the private key (e.g. `/home/joe/.ssh/id_rsa_github`)

# Screen

## Change to default resolution
```
xrandr -s 0
```

## To turn off, suspend or standby the screen
```
xset dpms force off
xset dpms force suspend
xset dpms force standby
```

# Programming

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

## make: how to autogenerate the help, example of makefile:
```
help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

pdf: $(SRCS)  ## Generate pdf files
	pandoc ....

clean:  ## Delete generated pdf files
	rm -f *.pdf
```

# Virtualization

## Create a vmdk from a physical disk
- Create a vmdk from the `/dev/sdb` device:
  ```
  vboxmanage internalcommands createrawvmdk -filename usb.vmdk -rawdisk /dev/sdb
  ```
- Source: [http://www.howtogeek.com/187721/how-to-boot-from-a-usb-drive-in-virtualbox/](http://www.howtogeek.com/187721/how-to-boot-from-a-usb-drive-in-virtualbox/)

## Virtualbox
- How to fix VirtualBox "UUID already exists" errors
  ```
  vboxmanage internalcommands sethduuid my_vm_disk.vmdk
  ```

# File Manipulation

## To undelete files (note: works better than photorec for jpegs)
```
magicrescue -r jpeg-jfif -r jpeg-exif -d ~/output /dev/hdb1
```

## To recover photos/videos
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

# Video Manipulation

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

# Image Manipulation

## Crop a 1000x400 pixels area at coordinates 1700,900 (uses imagemagick)
```
convert -crop 1000x400+1700+900 src.jpg dest.jpg
```

## To rotate or flip a jpeg losslessly
```
jpegtran -rotate 90 image.jpg
jpegtran -flip horizontal image.jpg
```

# Audio Manipulation

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
