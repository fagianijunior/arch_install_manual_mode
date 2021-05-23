### Install arch Linux
  # Do you will need:
    # - PC, notebook
    # - Pendrive >= 1GB
    # - 
## Pre-install
  # Download the iso image, I will use a mirror get from https://archlinux.org/download/ 
    ```wget http://mirrors.evowise.com/archlinux/iso/2021.05.01/archlinux-2021.05.01-x86_64.iso```
  # Burn the iso image on pendrive
    ```dd if=archlinux-2021.05.01-x86_64.iso of=/dev/sdb bs=1024k status=progress```

## Configure installation
  # Insert the pendrive on USB, check the boot ordem on your BIOS and if it is on UEFI mode start your machine
  # On grub screen, select the first option, to join to Arch Linux live system
  
  # Set the keyboard layout, you can list all layout types with this command `ls /usr/share/kbd/keymaps/**/*.map.gz`
    ```loadkeys br-abnt2```
    
  # Use tmux to has the ability to split terminal  (optional)
    tmux
    `ctrl+b "` # split horizontal
    # `ctrl+b %` # split vertival
    `ctrl+b arrow keys` # Change terminal
  # Verify the boot mode is UEFI, if this folder exist it is UEFI
    ```ls /sys/firmware/efi/efivars```
  # Connect to wifi (if not using wired)
    ```iwctl
    help
    device list
    station device scan
    station device get-networks
    station device connect SSID
    quit```
  # Check internet connection
    ```ping archlinux.org```
  # System time date
    ```timedatectl set-ntp true```
  # Particionar o disco (/mnt/efi(300M), /, SWAP)
    ```cfdisk /dev/sda```
  # Write the filetype
    ```mkfs.fat -F32 /dev/sda1
    mkfs.brtfs /dev/sda2
    mkswap /dev/sda3```
  # Mount the partitions
    ```mount /dev/sda2 /mnt
    mkdir /mnt/efi
    mount /dev/sda1 /mnt/efi
    swapon /dev/sda3```

## Install
  # Instalar pacotes (Separar por sessões para melhor entendimento)
    # Base do systema
      pacstrap /mnt base linux linux-firmware grub efibootmgr
    # Desenvolvimento
      pacstrap /mnt git vim terminator 
    # Outros
      pacstrap /mnt ranger zsh sudo
    # WM
      pacstrap /mnt i3-wm dmenu i3lock xss-lock i3status perl-anyevent-i3 perl-json-xs xorg-server xorg-xinit xorg-fonts-100dpi xorg-fonts-75dpi xorg-fonts-alias-100dpi xorg-fonts-alias-75dpi ttf-roboto ttf-roboto-mono
    # VirtualBox
      pacstrap /mnt xf86-video-vmware
  # Gerar o fstab
    genfstab -U /mnt >> /mnt/etc/fstab
  # Entrar no systema
    arch-chroot /mnt
  # Seta o timezone
    ln -sf /usr/share/zoneinfo/America/Fortaleza /etc/localtime
    hwclock --systohc
  # Idioma do sistema
    vi /etc/locale.gen # selecione o que achar melhor # en_US-UTF-8 pt_BR-UTF-8
    locale-gen
    echo 'LANG=en_US.UTF-8' > /etc/locale.conf
    echo 'KEYMAP=br-abnt2' > /etc/vconsole.conf
  # Configurar o nome do computador na rede
    echo 'doraemonwm' > /etc/hostname
    cat << EOF >> /etc/hosts
    127.0.0.1	localhost
    ::1		localhost
    127.0.1.1	doraemonwm.localdomain doraemonwm
    EOF
  # Gerar senha do usuário root
    passwd
  # Instalar gerenciador de boot
    grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
    grub-mkconfig -o /boot/grub/grub.cfg
  # reboot
## Pós instalação
  # Abilitar o gerenciamento de rede com fio e a resolução DNS
    su
    ip address
    cat << EOF > [Match]
    Name=enp0s3

    [Network]
    DHCP=yes
    EOF
    systemctl enable systemd-networkd.service
    systemctl start systemd-networkd.service
    systemctl enable systemd-resolved.service
    systemctl start systemd-resolved.service
  # Criar novousuário
    useradd -m -G audio,disk,kvm,optical,scanner,storage,video,network,wheel -s /bin/zsh terabytes
    passwd terabytes
  # Adicionar usuário ao sudoers
    sed --in-place 's/^#\s*\(%wheel\s\+ALL=(ALL)\s\+ALL\)/\1/' /etc/sudoers
  # ctrl+d para fazer logoff do root e logar com o novo usuário
  # Ao logar vai precisar configurar o zshell 
  # Instalar plugins do zsh 
    # Oh My Zsh
      sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
    # Oh My Zsh plugins
      git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
      git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
  # Configurar o X e i3
    cp /etc/X11/xinit/xinitrc ~/.xinitrc
    vi ~.xinitrc
    # comentar twm... adicionar i3
