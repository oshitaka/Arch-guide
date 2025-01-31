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
section: PKB
subsection: Linux
class: 
status: false
---
# Customization
## WM and additional packages
Main packages for GUI interfaces
```bash
sudo pacman -S --needed waybar firefox fuse3 ntfs-3g imv vlc mako brightnessctl spotify-launcher hyprland xdg-desktop-portal-hyprland hyprpaper hypridle hyprlock polkit-kde-agent qt5-wayland qt6-wayland slurp grim wf-recorder nwg-look noto-fonts otf-font-awesome ttf-jetbrains-mono-nerd ttf-nerd-fonts-symbols ttf-bigblueterminal-nerd noto-fonts-emoji ffmpegthumbnailer jq poppler rofi-wayland cmus usbutils less
```
Details:
- ttf - main fonts 
- fuse3 - need for Obsidian 
- ntfs-3g - for mounting NTFS disks
- cmus - music player
- gamescope - for steam games (see troubleshooting)
- wf-recorder - screen recording
- usbutils - collection of USB tools to query connected USB devices
- less - for git, to view info e.g. config or branch

```bash
paru -S visual-studio-code-bin obsidian-bin
```
Details:
- visual-studio-code-bin - config from here: https://github.com/Shobhit0109/maxhu08_dotfiles/tree/master/vscode

> [!note]
> Obsidian package was deleted. Get AppImage from official site

## GUI for utils (optional)
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

## Rice
See dotfiles

## Colour pallets
catppucin https://github.com/catppuccin/catppuccin
gruvbox https://github.com/morhetz/gruvbox
everforest https://github.com/sainnhe/everforest