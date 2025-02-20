---
date: 2024-05-25
links:
  - "[[_MOC Linux]]"
  - "[[Arch. 2-customization]]"
tags:
  - linux
  - workstation
aliases:
  - workstation
status: false
section: PKB
subsection: Linux
class:
---
# <span style="color:#ffadad">Tips</span>
## <span style="color:#ffd6a5">Open LUKS partition</span>
If you need to examine and?or repair root partition with LUKS encryption here is how you can to open it: https://wiki.archlinux.org/title/dm-crypt/Device_encryption#Unlocking/Mapping_LUKS_partitions_with_the_device_mapper or useLuksOpen (instead open) like when was installing


Shows titles and classes of windows to configer them in hypr.conf
```bash
hyprctl clients
```


cursor not run out of display
```bash
gamescope -W 1920 -H 1080 -g --adaptive-sync --expose-wayland  --force-grab-cursor --rt -f -- %command% 
```


https://github.com/mingo99/ouch.yazi - плагин для yazi
https://github.com/m4xshen/dotfiles/tree/main - как из rofi сделать лаунчер, пауерменю и прочее / настройки nvim / пример kitty /

https://github.com/sameemul-haque/dotfiles/tree/mocha/.config скрипты ррофи, и упревление через него wifi bloototh

https://github.com/pimutils/khal -календарь CLI

https://github.com/bilelmoussaoui/Hardcode-Tray - иконки для трея

https://github.com/demingongo/awesomewm-dotfiles/tree/main/config/awesome/icons/layouts - Есть иконки и пример оформления

https://wiki.hyprland.org/Useful-Utilities/Clipboard-Managers/ - менеждеры копипасты

https://github.com/loqusion/hyprshade - смена температуры экрана
https://github.com/Alexays/Waybar/wiki/Module:-Cava - cava in waybar

https://man.archlinux.org/man/cmus-tutorial.7

Images:
- krita
- gimp

Photo
- darktable

Video
- shotcut
- Kdenlive

pomotroid


## <span style="color:#ffd6a5">Данные клавиатуры и мониторов</span>
С помощью команды hyprctl monitors узнаем названия мониторов
hyprctl devices = название клавиатуры для внесения в конфин waybar



# NVIM

use lazy

https://github.com/akinsho/bufferline.nvim - tabs (lazy, plug) ☑️

https://github.com/adelarsq/image_preview.nvim - image preview (lazy, plug)

https://github.com/nvim-telescope/telescope-media-files.nvim - file preview using telescope (plug) !!!<span style="color:#ffadad"> ImageMAgik</span> !!!

https://github.com/nvim-tree/nvim-tree.lua/wiki/Installation - tree (plug, lazy)
https://github.com/b0o/nvim-tree-preview.lua - file preview(lazy)

https://github.com/stevearc/oil.nvim -file explorer (plug, lazy) ☑️

https://github.com/echasnovski/mini.nvim - lots of plugins

Autoclosing braces and html tags with [nvim-autopairs](https://github.com/windwp/nvim-autopairs)

NeoVim Lsp configuration with [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) and [mason.nvim](https://github.com/williamboman/mason.nvim)

Autocompletion with [nvim-cmp](https://github.com/hrsh7th/nvim-cmp)

Useful snippets with [friendly snippets](https://github.com/rafamadriz/friendly-snippets) + [LuaSnip](https://github.com/L3MON4D3/LuaSnip).

Syntax highlighting with [nvim-treesitter](https://github.com/nvim-treesitter/nvim-treesitter)

https://github.com/folke/which-key.nvim - is a lua plugin for Neovim 0.5 that displays a popup with possible key bindings of the command you started typing.




Nvim-Tree Alternatives

- [ChadTree](https://github.com/ms-jpq/chadtree)
- [Neo-Tree](https://github.com/nvim-neo-tree/neo-tree.nvim) ☑️
- [Tree](https://github.com/zgpio/tree.nvim)
- [NerdTree](https://github.com/preservim/nerdtree)
- [Defx](https://github.com/Shougo/defx.nvim)

Lua in nvim
https://github.com/kuator/nvim-lua-guide-ru

config: https://github.com/alexey-goloburdin/nvim-config/blob/main/init.vim

videos
- https://www.youtube.com/watch?v=zHTeCSVAFNY&list=PLsz00TDipIffreIaUNk64KxTIkQaGguqn
- https://www.youtube.com/watch?v=PA7zZNJXJEk

https://vimawesome.com/ - list of plugins

image preview
- [chafa](https://github.com/hpjansson/chafa) - terminal image previewer (recommended, supports most file formats)
- [viu](https://github.com/atanunq/viu) - terminal image previewer
- [ueberzugpp](https://github.com/jstkdng/ueberzugpp) - terminal image previewer using X11/Wayland child windows, sixels, kitty and iterm2

# How do I screenshot?[](https://wiki.hyprland.org/FAQ/#how-do-i-screenshot)

Install `grim` and `slurp`.

Use a keybind (or execute) `grim -g "$(slurp)"`, and select a region. A screenshot will pop into your `~/Pictures/` (You can configure grim and slurp, see their GitHub pages).

If you want those screenshots to go directly to your clipboard, consider using `wl-copy`, from [`wl-clipboard`](https://github.com/bugaevc/wl-clipboard). Here’s an example binding: `bind = , Print, exec, grim -g "$(slurp -d)" - | wl-copy`

Save screenshots to specific folder
```bash
grim -g "$(slurp)" $HOME/Pictures/screenshots/$(date "+%y%m%d_%H-%M-%S").png
```

# <span style="color:#fdffb6">header</span>
Power profile deamon to manage the battery life
https://archlinux.org/packages/extra/x86_64/power-profiles-daemon/

Battery Conservation Mode is a feature that limits battery charging to 55-60% of its capacity to improve battery life, being most useful when the laptop tends to run on external power much of the time. This works on many Lenovo laptops like IdeaPad and Thinkbook series. To check if your laptop is supported, try to set the battery conservation mode in the Vantage app on Windows. If it works on Windows, it can be enabled or disabled on Linux in the following manner:

- First make sure the `ideapad_laptop` kernel module is loaded, with the `lsmod` command.
- If it is, run the following command as root to enable Battery Conservation Mode:
```bash
echo 1 > /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode
```

- A `0` will in turn disable the feature.