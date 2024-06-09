---
date: 2024-05-21
links:
  - "[[_MOC Linux]]"
  - "[[Arch. 3-tweaks & troubleshooting]]"
tags:
  - workstation
  - linux
aliases:
  - workstation
cssclasses: 
status: false
---
# <span style="color:#ffadad">Customization</span>
## <span style="color:#ffd6a5">WM and additional packages</span>
Main packages for GUI interfaces
```bash
sudo pacman -S --needed waybar firefox fuse2 ntfs-3g imv mpv mako brightnessctl spotify-launcher hyprland xdg-desktop-portal-hyprland hyprpaper hypridle hyprlock sddm polkit-kde-agent qt5-wayland qt6-wayland slurp grim nwg-look noto-fonts otf-font-awesome ttf-jetbrains-mono-nerd ttf-nerd-fonts-symbols ttf-bigblueterminal-nerd noto-fonts-emoji ffmpegthumbnailer jq poppler rofi-wayland cmus
```
Details:
- <span style="color:#fdffb6">ttf</span> - main fonts 
- <span style="color:#fdffb6">fuse2</span> - need for Obsidian 
- <span style="color:#fdffb6">ntfs-3g</span> - for mounting NTFS disks
- <span style="color:#fdffb6">cmus</span> - music player
- <span style="color:#fdffb6">gamescope</span> - for steam games (see troubleshooting)

```bash
paru -S visual-studio-code-bin obsidian-bin
```
Details:
- <span style="color:#fdffb6">visual-studio-code-bin</span> - config from here: https://github.com/Shobhit0109/maxhu08_dotfiles/tree/master/vscode

## <span style="color:#ffd6a5">GUI for utils (optional)</span>
USBGuard, bluetooth, ClamAV, Network manager
```bash
paru -S usbguard-qt clamtk blueman nm-connection-editor
```

>[!tip] Usege of network manager
>I prefer to use script for rofi
>```bash
>paru -S networkmanager-dmenu-git
>```


And for GUI usbguard need to start service. Otherwise some futures won't work
```bash
sudo systemctl enable --now usbguard-dbus.service
```

## <span style="color:#ffd6a5">Rice</span>
See dotfiles