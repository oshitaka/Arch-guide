---
date: 2024-05-21
links:
  - "[[_MOC Linux]]"
  - "[[Arch. 1-installation]]"
tags:
  - linux
  - workstation
aliases:
  - workstation
section: PKB
subsection: Linux
class: 
status: false
---
# <span style="color:#ffadad">Tweaks</span>
## <span style="color:#ffd6a5">Hotkeys for laptop</span>
in progress

## <span style="color:#ffd6a5">Power management</span>
### <span style="color:#fdffb6">Battery charging mode</span>
- First make sure the `ideapad_laptop` kernel module is loaded, with the `lsmod` command.
- If OK then
```bash
echo 1 > /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode
```
A `0` will disable the feature.

### <span style="color:#fdffb6">Power modes</span>
Install laptop-mode-tools (really need this?)
```bash
paru -S laptop-mode-tools
```

## <span style="color:#ffd6a5">Clean Systemd journal</span>
`Systemd` stores its logs in `/var/log/journal/`. These log files can take up to 10% of your system size by default. 
Manually limit the size of log file
```bash 
sudo journalctl --vacuum-size=200M
```

<span style="color:#f12778">Set permanent limit</span>. Open
```bash
sudo nvim /etc/systemd/journald.conf
```
and set size
```bash
SystemMaxUse=200M
```


---
# <span style="color:#ffadad">Troubleshooting</span>
## <span style="color:#ffd6a5">Bluetooth</span>
>[!done]

<span style="font-weight:bold; color:#a0c4ff">Description:</span> unable to turn on bluetooth. Get an error:
```bash
Failed to set power on: org.bluez.Error.Failed
```

<span style="font-weight:bold; color:#a0c4ff">Solution:</span>`rfkill` blocks bluetooth on every start. To unblock bluetooth
```bash
rfkill unblock bluetooth
```
>[!note]
>Better to make an autostart script for that or find out how to change setting in `rfkill`

https://stackoverflow.com/questions/68728478/failed-to-set-power-on-org-bluez-error-blocked-problem 

## <span style="color:#ffd6a5">Idle</span>
>[!done]

<span style="font-weight:bold;color:#a0c4ff">Description</span>: Using hypridle. Machine suspend when watching youtube, listen to spotify and etc. 

<span style="font-weight:bold; color:#a0c4ff">Solution:</span> added idle_inhibitor in waybar

## <span style="color:#ffd6a5">Booting</span>
>[!done]

<span style="font-weight:bold;color:#a0c4ff">Description</span>: Some errors appears during booting. Seems they affect the system.
Results (only **errors and warnings**) of `journalctl -xb`

>[!info]- List of errors & warnings
>```bash
x86/cpu: SGX disabled by BIOS. #FIXED
>
ACPI BIOS Error (bug): Failure creating named object [\_SB.PCI0.XH
C.RHUB.HS01._UPC], AE_ALREADY_EXISTS (20230628/dswload2-326) # IGNORE
>
ACPI Error: AE_ALREADY_EXISTS, During name lookup/catalog (2023062
8/psobject-220) # IGNORE
>
hpet_acpi_add: no address or irqs in _CRS # IGNORE
>
integrity: Loading X.509 certificate: UEFI:db
integrity: Problem loading X.509 certificate -65 #IGNORE
>
i8042: PNP: PS/2 appears to have AUX port disabled, if this is incorrect please boot with i8042.nopnp
>
usb: port power management may be unreliable
>
nvidia: loading out-of-tree module taints kernel.
>
nvidia: module verification failed: signature and/or required key 
missing - tainting kernel # IGNORE
>
ACPI Warning: \_SB.PCI0.PEG0.PEGP._DSM: Argument \#4 type mismatch - Found [Buffer], ACPI requires [Package] (20230628/nsarguments-61)
>
NVRM: nvAssertOkFailedNoLog: Assertion failed: Invalid
 data passed [NV_ERR_INVALID_DATA] (0x00000025) returned from PlatformRequestHandler failed to get target temp from SBIOS @ 
platform_request_handler_ctrl.c:2146
>
NVRM: nvAssertOkFailedNoLog: Assertion failed: Invalid data passed [NV_ERR_INVALID_DATA] (0x00000025) returned from PlatformRequestHandler failed to get target temp from SBIOS @ 
platform_request_handler_ctrl.c:2146
>
nvidia: Process '/usr/bin/bash -c '/usr/bin/mknod -Z -m 666 /dev/nvidiactl c $(grep nvidia /proc/devices | cut -d \ -f 1) 255'' failed with exit code 1.
>
nvidia: Process '/usr/bin/bash -c 'for i in $(cat /proc/driver/nvidia/gpus/*/information | grep Minor | cut -d \  -f 4); do /usr/bin/mknod -Z -m 666 /dev/nvidia${i} c $(grep nvidia /proc/devices | cut -d \  -f 1) ${i}; done'' failed with exit code 1.
>
Process 'lmt-udev force' failed with exit code 1.
>
Activation request for 'org.freedesktop.home1' failed: The systemd unit 'dbus-org.freedesktop.home1.serv
ice' could not be found.
>
Service file '/usr/share/dbus-1/services/fr.emersion.mako.service' is not named after the D-Bus name 'or
g.freedesktop.Notifications'.
>
file:///usr/share/sddm/themes/catppuccin-frappe/Main.qml:18:3: QML QQuickImage: Cannot open: file:///usr/s
hare/sddm/themes/catppuccin-frappe/backgrounds/wall.jpg # IGNORE
>
Activation request for 'org.freedesktop.resolve1' failed: The systemd unit 'dbus-org.freedesktop.resolve
1.service' could not be found.
>
xdg-desktop-por[2763]: No skeleton to export # IGNORE
>
Failed to get percentage from UPower: org.freedesktop.DBus.Error.NameHasNoOwner
>
warning: `ThreadPoolForeg` uses wireless extensions which will stop working for Wi-Fi 7 hardware; use nl80211
>
bluetoothd[2018]: src/adv_monitor.c:btd_adv_monitor_power_down() Unexpected NULL btd_adv_monitor_manager object upon power down
>
bluetoothd[2018]: Failed to set mode: Failed (0x03)
>```

<span style="font-weight:bold;color:#a0c4ff">Solution</span>: ignore all this notifications, they doesn't effect on system

## <span style="color:#ffd6a5">Hibernation</span>
>[!error]

<span style="font-weight:bold;color:#a0c4ff">Description</span>: When resuming from hibernation, system unable to mount root partition
```bash
EXT4-fs error (device dm-0): ext4_orphan_get: 1420: comm mount: bad orphan inode10621386
EXT4-fs error (device dm-0): ext4_mark_recovery_comlpete:6245: comm mount Orphan file not empty on read-only fs
EXT4-fs (dm0): mount failed

```

<span style="font-weight:bold; color:#a0c4ff">Solution</span>: 

By trial and error found that it's because of `options nvidia NVreg_PreserveVideoMemoryAllocations=1` in `/etc/modprobe.d/nvidia.conf`
So this appears only with 2 monitors.
<span style="color:#f12778">Yet no solution</span>


```
options nvidia NVreg_PreserveVideoMemoryAllocations=1 NVreg_TemporaryFilePath=/var/temp
```

possible solution: https://archlinux.org.ru/forum/topic/22107/?page=2#post-261722

In addition. `systemd-hibernate-resume.service loaded failed failedResume from hibernation`

## <span style="color:#ffd6a5">Firefox</span>
>[!done]

<span style="font-weight:bold;color:#a0c4ff">Description</span>: crashes when open certain sites or files
. E.g.: draculatheme.com, help.obsidian.md/Editing+and+formatting/Callouts 

<span style="font-weight:bold; color:#a0c4ff">Solution</span>: Disabled hardware acceleration 

## <span style="color:#ffd6a5">Mouse in games</span>
>[!error]

<span style="font-weight:bold;color:#a0c4ff">Description</span>: cursor runs out of game to another monitor

WORKS
```bash
hyprctl keyword monitor HDMI-A-1, dpms off
```
hyprReturn monitor setings
```bash
hyprland reload
```

Also 
```bash
hyprctl dispatch dpms off; sleep 5s; hyprctl dispatch dpms on
```


1. https://github.com/hyprwm/Hyprland/issues/1732
This is currently my solution, not a perfect one through.  
You can set the position of your monitor not equal to `resolution / scale` to keep your cursor in that monitor(e.g. +100 in x position for the right monitor). The problem is you can't move your cursor to another monitor when you want to. Another thing is that you can edit the keyword `monitor` using `hyprctl`. I have a script to toggle this screen edge locking:
```bash
#!/bin/bash

# set the expected x position of the monitor by your preference
expect_x=3072
# index 0 is the first monitor in your hyprland.conf
curr_x=$(hyprctl -j monitors | jq '.[0].x') 
echo $curr_x
if [ $curr_x -ne $expect_x ]
then
    echo "Hi"
    hyprctl keyword monitor DP-2, 3840x2160@144, ${expect_x}x0, 1.25
else
    expect_x=$(($expect_x + 100))
    echo "Bye ${expect_x}"
    hyprctl keyword monitor DP-2, 3840x2160@144, ${expect_x}x0, 1.25
fi
```
You can edit the command with your own monitor's resolution, refresh rate and signal input. Finally you can bind this script in your `hyprland.conf`. I suggest you to test the script before binding it.

2. https://git.mgoder.com/mg/hyprlock 
Hyprlock is a script that allows for the mouse cursor to be locked to a monitor in Hyprland Window Manager.

Setup:

- Copy `hyprlock.sh` into `~/.config/hypr`
- Modify the monitor positions at the top of `~/.config/hypr/hyprlock.sh`. Execute `hyprctl -j monitors` to gather information about the monitor configuration.  
Example:
```bash
unlockedMonitorYPositions=(0 0 0)
lockedMonitorYPositions=(1152 1152 1152)
```
- Execute `chmox +x ~/.config/hypr/hyprlock.sh`
- Execute `sudo ln -s ~/.config/hypr/hyprlock.sh /usr/bin/hyprlock`
- Add a bind to Hyprlock in `~/.config/hypr/hyprland.conf`.
Example:
```bash
bind = $mainMod, Q, exec, ~/.config/hypr/hyprlock.sh
```

Script:
```bash
#!/bin/bash

# Use "hyprctl -j monitors" to determine the parameters for the lock and unlock positions.
unlockedMonitorYPositions=(0 0 0)
lockedMonitorYPositions=(1152 1152 1152)

monitors=$(hyprctl -j monitors)
tailMonitorIndex=$(($(echo "$monitors" | jq length)-1))
monitorIndices=$(seq 0 $tailMonitorIndex)

if [ "$1" == "" ]; then
	monitorUnlocked=0
	for monitorIndex in $monitorIndices; do
		monitor=$(echo "$monitors" | jq -c ".[$monitorIndex]")
		
		unlockedMonitorYPosition=${unlockedMonitorYPositions[$monitorIndex]}
	
		if [ $(echo "$monitor" | jq -j ".y") -eq $unlockedMonitorYPosition ]; then continue; fi
	
		hyprctl keyword monitor $(echo "$monitor" | jq -j ".name"),$(echo "$monitor" | jq -j ".width")x$(echo "$monitor" | jq -j ".height")@$(echo "$monitor" | jq -j ".refreshRate"),$(echo "$monitor" | jq -j ".x")x$unlockedMonitorYPosition,$(echo "$monitor" | jq -j ".scale") > /dev/null
		
		monitorUnlocked=1
		
		echo Unlocked monitor $monitorIndex
	done

	if [ $monitorUnlocked -eq 1 ]; then exit; fi

	for monitorIndex in $monitorIndices; do
		monitor=$(echo "$monitors" | jq -c ".[$monitorIndex]")
		
		if [ $(echo "$monitor" | jq -j ".focused") == "false" ]; then continue; fi
		
		hyprctl keyword monitor $(echo "$monitor" | jq -j ".name"),$(echo "$monitor" | jq -j ".width")x$(echo "$monitor" | jq -j ".height")@$(echo "$monitor" | jq -j ".refreshRate"),$(echo "$monitor" | jq -j ".x")x${lockedMonitorYPositions[$monitorIndex]},$(echo "$monitor" | jq -j ".scale") > /dev/null
		
		echo Locked monitor $monitorIndex
		exit
	done
elif [ "$1" == "status" ]; then
	for monitorIndex in $monitorIndices; do
		if [ $(echo "$monitors" | jq -j ".[$monitorIndex].y") -eq ${unlockedMonitorYPositions[$monitorIndex]} ];
			then echo Monitor $monitorIndex: Unlocked
			else echo Monitor $monitorIndex: Locked;
		fi
	done
	exit
elif [ "$1" == "help" ] || [ "$1" == "-h" ] || [ "$1" == "-help" ] || [ "$1" == "--help" ]; then
	echo Usage:
	echo -e "\thyprlock status"
	echo -e "\thyprlock [monitor_id]"
	exit
fi

if ! [[ $1 =~ ^[0-9]+$ ]]; then
	echo "Error: Invalid parameter value was provided."
	echo "[monitor_id] must be an integer."
	exit 1
fi

if [ $1 -lt 0 ] || [ $tailMonitorIndex -lt $1 ]; then
	echo "Error: The provided [monitor_id] is out of range."
	echo "[monitor_id] must be an integer from 0 to $tailMonitorIndex."
	exit 2
fi

monitor=$(echo "$monitors" | jq -c ".[$1]")

unlockedMonitorYPosition=${unlockedMonitorYPositions[$1]}

if [ $(echo "$monitor" | jq -j ".y") -eq $unlockedMonitorYPosition ]; then
	hyprctl keyword monitor $(echo "$monitor" | jq -j ".name"),$(echo "$monitor" | jq -j ".width")x$(echo "$monitor" | jq -j ".height")@$(echo "$monitor" | jq -j ".refreshRate"),$(echo "$monitor" | jq -j ".x")x${lockedMonitorYPositions[$monitorIndex]},$(echo "$monitor" | jq -j ".scale") > /dev/null
		
	echo Locked monitor $1
	exit
fi
	
hyprctl keyword monitor $(echo "$monitor" | jq -j ".name"),$(echo "$monitor" | jq -j ".width")x$(echo "$monitor" | jq -j ".height")@$(echo "$monitor" | jq -j ".refreshRate"),$(echo "$monitor" | jq -j ".x")x$unlockedMonitorYPosition,$(echo "$monitor" | jq -j ".scale") > /dev/null
	
echo Unlocked monitor $1
```