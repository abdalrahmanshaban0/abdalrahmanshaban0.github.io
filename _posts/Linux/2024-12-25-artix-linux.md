---
title: Artix Linux
image: /assets/posts/Linux/artix.png
categories: [Linux]
---
# Choosing distro
- It depends on your experience with Linux
	- Beginner ? Mint : Arch
- If you know what you're doing, Arch is very very stable.
- Arch with no systemd == Artix
- Why Arch ? 
	- Rolling release (No versions, No major upgrades, Bleeding edge)
	- Good {Wiki, Community, AUR}
- Why not systemd ? 
	- I noticed it uses more memory and causing some problems.
	- It's in the sucks page in suckless.
	- Use runit instead.

## Little tip for troubleshooting 
- Open programs in terminal (Ex. Davinci Resolve)

## Artix Linux Installation Guide
First, You can login as 
-> root
-> artix
## Connecting to WIFI
```shell
#Enabling WIFI
$ rfkill unblock wifi

$ wpa_cli
$ scan
$ scan_results
$ add network
0, 1, 2, ...
$ set_network 0 ssid "NETWORK NAME"
$ set_network 0 psk "NETWORK PASS"
$ enable_network 0
$ save config
```
## Partitioning
```shell
$ lsblk
# choose the drive you want to install the system in like /dev/sda
$ cfdisk
# make 512M with type -> EFI System
# make for example 4G swap with type -> Linux Swap
# make the rest of the size -> Linux File System (for the root partition)
```
## Mount Points
```shell
#Assume the output of $lsblk like this:
# sda |
#     -> sda1 512M
#     -> sda2 111.3G
$ mkfs.fat -F32 /dev/sda1
$ mkfs.ext4 /dev/sda2
$ mount /dev/sda2 /mnt
$ mkdir -p /mnt/boot/efi
$ mount /dev/sda1 /mnt/boot/efi
```
## Install base packages
```shell
$ basestrap /mnt base base-devel runit elogind-runit linux linux-firmware vim intel-ucode grub efibootmgr linux-headers
```
## fstab, clock, locale, hostname
```shell
$ fstabgen -U /mnt >> /mnt/etc/fstab
$ artix-chroot /mnt
$ ln -sf /usr/share/zoneinfo/Africa/Cairo /etc/localtime
$ hwclock --systohc
$ vim /etc/locale.gen #uncomment en_US.UTF-8
$ locale-gen
$ vim /etc/locale.conf
	-> LANG=en_US.UTF-8
$ vim /etc/hostname
	-> HOSTNAME
```
## Install Grub
```shell
$ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB --removable
$ grub-mkconfig -o /boot/grub/grub.cfg
```
## Adding users
```shell
$ useradd -mG wheel abdalrahman
$ passwd abdalrahman
$ vim /etc/sudoers
	-> uncomment %wheel ALL=(ALL:ALL) ALL
```

## Installing some packages
```shell
pacman -s xf86-video-intel xorg-server networkmanager networkmanager-runit network-manager-applet git xdg-utils xdg-user-dirs
ln -s /etc/runit/sv/NetworkManager/ /run/runit/service/NetworkManager
```
## Reboot

## Connect to Internet using NetworkManager
```shell
$ nmtui
```
## Post Installation
```shell
echo "Installing xorg needed stuff.."
sudo pacman -Syu libxft libxinerama xorg-xinit libxrandr xorg-xrandr

echo "Installing some fonts.."
sudo pacman -S noto-fonts noto-fonts-emoji noto-fonts-cjk ttf-dejavu ttf-liberation ttf-jetbrains-mono ttf-hack-nerd ttf-nerd-fonts-symbols

echo "Installing my packages.."
sudo pacman -S dunst feh pulseaudio firefox mpv yt-dlp pavucontrol pamixer

echo "Installing suckless tools.."
mkdir ~/.src
cd ~/.src
git clone https://git.suckless.org/dwm
git clone https://git.suckless.org/st
git clone https://git.suckless.org/dmenu
git clone https://git.suckless.org/slstatus
cd dwm/
echo "Installing dwm.."
sudo make clean install
cd ../st
echo "Installing st.."
sudo make clean install
cd ../slstatus
echo "Installing slstatus.."
sudo make clean install
cd ../dmenu
echo "Installing dmenu.."
sudo make clean install
cd
echo dwm >> .xinitrc
```
## Fix Screen tearing
```
--> /etc/X11/xorg.conf.d/20-intel.conf
Section "Device"
	Identifier    "Intel Graphics"
	Driver        "intel"
	Option        "AccelMethod"    "sna"
	Option        "DRI"            "3"
	Option        "TearFree"       "true"
	Option        "TripleBuffer"   "true"
EndSection
```

## sddm 
```shell
sudo pacman -S sddm sddm-runit
sudo ln -s /etc/runit/sv/sddm /run/runit/service/
sudo mkdir /usr/share/xsessions
```
```
#/usr/share/xsessions/dwm.desktop
[Desktop Entry]
Encoding=UTF-8
Name=dwm
Comment=Dynamic window manager
Exec=dwm
Icon=dwm
Type=XSession
```

## Enable arch-support
```
https://wiki.artixlinux.org/Main/Repositories
```

## sxhkd & screenlock & SRS (Shutdown, Reboot, Suspend) | demnu
```shell
sudo pacman -S sxhkd slock xss-lock
mkdir ~/.config/sxhkd
#Auto start this command:
	xss-lock --transfer-sleep-lock -- slock
## Add your first script!
mkdir ~/.scripts
vim ~/.scripts/SRS.sh
echo -e "poweroff\nreboot\nsuspend" | dmenu | xargs loginctl
chmod +x ~/.scripts/SRS.sh

#Add your first keybinding!
#~/.config/sxhkd/sxhkdrc
super + shift + m
	~/.scripts/SRS.sh
super + shift + l
	slock
```

## File manager (thunar)
```shell
sudo pacman -S thunar gvfs gvfs-mtp thunar-volman ffmpegthumbnailer tumbler man-db lsd zathura zathura-pdf-mupdf 
```
## Archiving
```shell
sudo pacman -S bzip2 gzip xztar p7zip unrar zip unzip
```
## ScreenKey & ScreenShots & compositor
```shell
sudo pacman -S slop screenkey maim xclip picom
#For screenkey
screenkey -g $(slop -n -f '%g')
#for screenshot
maim -s | tee ~/Pictures/$(date +%s).png | xclip -selection clipboard -t image/png
#Auto start picom
#Add keybinding for screenshot
#Add keybinding for toggle script for screenkey
```
## Bluetooth
```shell
sudo pacman -S bluez bluez-runit bluez-utils bluez-obex pulseaudio-bluetooth blueman
ln -s /etc/runit/sv/bluetoothd /run/runit/service
#autostart blueman-applet
```
## nvim
```shell
sudo pacman -S lua luarocks repgrep
#https://www.youtube.com/watch?v=6pAG3BHurdM

#install LazyVim (plugin manager for nvim)
https://github.com/LazyVim/LazyVim
#install gruvbox theme (or any theme you like)
https://github.com/ellisonleao/gruvbox.nvim
#install telescope
https://github.com/nvim-telescope/telescope.nvim
#install treesitter
https://github.com/nvim-treesitter/nvim-treesitter
#install neotree
https://github.com/nvim-neo-tree/neo-tree.nvim
#install mason
https://github.com/williamboman/mason.nvim
#install mason-lsp-config
https://github.com/williamboman/mason-lspconfig.nvim
#install nvim-lspconfig
https://github.com/neovim/nvim-lspconfig
#install telescope-ui-select
https://github.com/nvim-telescope/telescope-ui-select.nvim
#install none-ls
https://github.com/nvimtools/none-ls.nvim
#install nvim.cmp
https://github.com/hrsh7th/nvim-cmp
#LuaSnip
https://github.com/L3MON4D3/LuaSnip
#cmp-nvim-lsp
https://github.com/hrsh7th/cmp-nvim-lsp
#friendly snippets
https://github.com/rafamadriz/friendly-snippets
#indent-blankline
https://github.com/lukas-reineke/indent-blankline.nvim
#nvim.autopairs
https://github.com/windwp/nvim-autopairs
#greeter
https://github.com/goolord/alpha-nvim
#bufferline
https://github.com/akinsho/bufferline.nvim
#dressing 
https://github.com/stevearc/dressing.nvim
#whitch-key
https://github.com/folke/which-key.nvim
#maxmizer
https://github.com/szw/vim-maximizer
#lualine
https://github.com/nvim-lualine/lualine.nvim
```