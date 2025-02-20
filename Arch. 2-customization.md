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
section: PKM
subsection: Linux
class: note
status: false
---
# Customisation
## WM and additional packages
Main packages
```bash
sudo pacman -S --needed waybar firefox fuse3 imv mako spotify-launcher hyprland xdg-desktop-portal-hyprland hyprpaper hypridle hyprlock polkit-kde-agent qt5-wayland qt6-wayland slurp grim wf-recorder nwg-look noto-fonts otf-font-awesome ttf-jetbrains-mono-nerd ttf-nerd-fonts-symbols ttf-bigblueterminal-nerd noto-fonts-emoji ffmpegthumbnailer jq poppler rofi-wayland
```
Details:
- ttf - main fonts 
- gamescope - for steam games (see troubleshooting)
- wf-recorder - screen recording


```bash
paru -S visual-studio-code-bin obsidian-bin
```
Details:
- visual-studio-code-bin - config from here: https://github.com/Shobhit0109/maxhu08_dotfiles/tree/master/vscode

> [!note]
> Obsidian package was deleted. Get AppImage from official site

## Rice
See dotfiles

## Colour pallets
catppucin https://github.com/catppuccin/catppuccin
gruvbox https://github.com/morhetz/gruvbox
everforest https://github.com/sainnhe/everforest
[[HTB theme]]

## Lockscreen
See [[Hyprlock]]

## Set keybindings in Hyprland
See: https://wiki.hyprland.org/Configuring/Binds/#uncommon-syms--binding-with-a-keycode
and see [[Power management & special keys]]

## GTK themes
Get [catppuccin](https://www.pling.com/p/1715554) and [gruvbox](https://www.pling.com/p/1681313) and place them `/usr/share/themes`
