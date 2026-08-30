archlinux
###############


制作启动u盘
===========

下载\ `archlinux iso <http://www.archlinux.org/download/>`__

下载\ `ventoy <https://www.ventoy.net/>`__

刻录ventoy后，将ventoy分区格式化为ntfs。

直接将archlinux iso文件拷贝到ventoy分区。

还可以放入 `clonezilla
iso <http://drbl.nchc.org.tw/clonezilla/clonezilla-live/download/>`__,
`gparted iso <https://gparted.org/livecd.php>`__, `winpe
iso <https://www.ibmnb.com/51pe/>`__ 等其他iso。

dd
--

linux 环境直接刻录u盘:

.. code-block::

   dd bs=4M if=archlinux.iso of=/dev/sdx status=progress && sync

windows
-------

windows环境直接刻录u盘:

.. code-block::

   下载LinuxLive USB Creator：http://www.linuxliveusb.com/

初始安装
========

`How to Install Arch Linux <https://itsfoss.com/install-arch-linux/>`__

`Arch Linux and Windows 10 (UEFI + Encrypted) Install
Guide <https://octetz.com/docs/2020/2020-2-16-arch-windows-install/>`__

u盘启动
-------

进入BIOS，从U盘启动，进入archlinux live

设置arch源
----------

编辑/etc/pacman.d/mirrorlist，选择合适的server

例如

http://mirrors.163.com/archlinux/

硬盘分区
--------

假设系统硬盘为/dev/sda

全新分区
~~~~~~~~

   fdisk /dev/sda

将磁盘旧的msdos分区表切换为gpt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

   sgdisk -g /dev/sda

新建系统分区
~~~~~~~~~~~~

mkfs -t ext4 -b 4096 -E stride=128,stripe-width=128 /dev/sda1 

调整系统分区大小
~~~~~~~~~~~~~~~~

.. code-block::

   e2fsck -f /dev/sda1
   resize2fs /dev/sda1

新建efi分区
~~~~~~~~~~~

磁盘新建一个efi分区（假设为 /dev/sda5），类型为vfat，标识为 ``boot, esp`` ，大小例如1GB

/etc/fstab 示例
~~~~~~~~~~~~~~~

.. code-block::

   /dev/sda1           /   ext4        rw,relatime,stripe=128  0 1
   /dev/sda5       /boot           vfat        defaults 0 2

连接无线网络
------------

安装镜像已预装并启用 ``iwd``，可以使用 ``iwctl`` 连接无线网络：

.. code-block:: console

   # iwctl
   [iwd]# device list
   [iwd]# station <device> scan
   [iwd]# station <device> get-networks
   [iwd]# station <device> connect <SSID>

安装系统
--------

.. code-block::

    mount /dev/sda1 /mnt 
    mount /dev/sda5 /mnt/boot
    pacstrap /mnt base base-devel dialog vim networkmanager
    pacstrap /mnt linux linux-headers linux-firmware 
    pacstrap /mnt glibc lib32-glibc 
    arch-chroot /mnt

编辑/etc/pacman.d/mirrorlist，设置arch源，同上节

.. code-block::

    ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
    mkinitcpio -p linux 
    systemctl enable NetworkManager
    exit 
    reboot

系统更新
--------

重启后使用 ``nmtui`` 或 ``nmcli`` 连接网络

.. code-block::

    pacman -Syu 

数据包更新失败
--------------

.. code-block::

    sudo pacman -S archlinux-keyring
    sudo pacman-key –refresh-keys 
    sudo pacman-key –populate archlinux 
    sudo pacman -Scc 
    sudo pacman -Syu 

pacman/yaourt调用aria2多线程下载文件
------------------------------------

假设同时开8个连接

.. code-block::

   pacman -S aria2

在/etc/pacman.conf中指定

.. code-block::

    XferCommand = /usr/bin/aria2c -c -o %o %u 

在/etc/makepkg.conf中指定DLAGENTS

.. code-block::

    http::/usr/bin/aria2c -c -o %o %u
    https::/usr/bin/aria2c -c -o %o %u
    ftp::/usr/bin/aria2c -c -o %o %u

时间
----

设置rtc避免windows/archlinux重启后时间乱跳

.. code-block::

    pacman -S ntp ntpdate asia.pool.ntp.org
    timedatectl set-local-rtc 1 
    timedatectl status 

UEFI 引导
=========

配置 uefi Systemd-boot
----------------------

`用Systemd-boot取代GRUB作為Linux的bootloader <https://ivonblog.com/posts/replace-grub-with-systemd-boot/>`__

安装
~~~~

.. code-block::

   pacman -S efibootmgr

安装/修复 EFI引导
~~~~~~~~~~~~~~~~~

进入archlinux live环境

.. code-block::

   mount /dev/sda1 /mnt 
   mount /dev/sda5 /mnt/boot
   bootctl install --path=/mnt/boot

设置archlinux的EFI引导
----------------------

blkid 查看 archlinux 根分区的PARTUUID。

编辑 /boot/loader/entries/arch.conf

.. code-block::

   title Arch Linux
   linux   /vmlinuz-linux 
   initrd  /initramfs-linux.img
   options root=PARTUUID=xxxxxxxxxx rw quiet splash

添加windows的EFI引导
--------------------

把windows系统分区下的EFI/Microsoft目录直接拷贝到/boot/EFI/目录下。

设置默认引导
------------

编辑 /boot/loader/loader.conf

.. code-block::

   default @saved
   timeout 3
   console-mode max


sudo bootctl set-default @saved


boot目录示例
------------

.. code-block::

   $ tree /boot -L 3
   /boot
   ├── EFI
   │   ├── Microsoft
   │   │   ├── Boot
   │   │   └── Recovery
   │   └── systemd
   │       └── systemd-bootx64.efi
   ├── initramfs-linux-fallback.img
   ├── initramfs-linux.img
   ├── loader
   │   ├── entries
   │   │   └── arch.conf
   │   ├── entries.srel
   │   ├── loader.conf
   │   └── random-seed
   └── vmlinuz-linux

调整efi boot项
--------------

设置boot优先顺序，例如先1再0：efibootmgr -o 1,0

删除某个boot项，例如3: efibootmgr -b 3 -B

图形界面
========

安装X
-----

.. code-block::

    pacman -S xorg xorg-xinit consolekit 

安装XFCE
~~~~~~~~

.. code-block::

    pacman -S xfce4 xfce4-goodies xfce4-notifyd elementary-xfce-icons 

进入X的配置，不然关机键老是灰的：编辑~/.xinitrc

.. code-block::

    exec ck-launch-session dbus-launch startxfce4 

硬件驱动
========

声卡
----

- `ArchWiki:设置ALSA <https://wiki.archlinux.org/index.php/ALSA_%E5%AE%89%E8%A3%85%E8%AE%BE%E7%BD%AE_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29>`__
- `archlinux音量太小的问题解决 <https://bbs.archlinux.org/viewtopic.php?pid=1090109>`__

.. code-block::

     pacman -Sy pavucontrol pulseaudio pulseaudio-alsa
     pacman -S alsa-lib alsa-utils alsa-oss sof-firmware 
     gpasswd -a USERNAME wheel 
     gpasswd -a USERNAME audio 
     alsaconf 
     alsamixer 
     alsactl store 

编辑 ``/etc/rc.conf`` 文件，添加 ``alsa`` 到DAEMONS行。

编辑\ ``/etc/modprobe.d/alsa-base``\ 文件，添加以下两行：

.. code-block::
 options snd-usb-audio index=0 options snd-hda-intel
index=1 


热插拔(xfce4)
-------------

.. code-block::

    pacman -S ntfs-3g thunar-volman udisks2 
    pacman -S gvfs gvfs-afc gvfs-gphoto2 gvfs-mtp 

配置 /etc/fstab，手动挂载磁盘

.. code-block::

    /dev/sdb1 /mnt/usb ntfs-3g noauto,users,permissions 0 0

挂载示例：

.. code-block::

   udisksctl mount -b /dev/sdc1

显卡
----

.. code-block::

   sudo pacman -S nvidia-dkms  nvidia-utils  lib32-nvidia-utils nvidia-prime

触摸屏
------

.. code-block::

   sudo pacman -S xf86-input-wacom

关闭触摸板
----------

.. code-block::

    sudo pacman -S synaptics 
    sudo synclient TouchpadOff=1

输入法
======

.. code-block::

    pacman -S ibus-rime 

在\ ``$HOME/.bashrc``\ 中添加：

.. code-block::

    export LANG=zh_CN.UTF-8
    export LC_CTYPE=zh_CN.UTF-8
    export XIM_PROGRAM=ibus 
    export XMODIFIERS="@im=ibus"
    export GTK_IM_MODULE=ibus 
    export QT_IM_MODULE=ibus
    export XIM=ibus 

在\ ``$HOME/.xinitrc``\ 中添加：

.. code-block::

   ibus-daemon -drx

郑码
----

.. code-block::

   git clone https://github.com/rime/plum
   cd plum 
   bash rime-install lotem/rime-zhengma

编辑 ``~/.config/ibus/rime/default.custom.yaml``\ ：

.. code-block::

   patch:
     schema_list:
       - schema: zhengma
       - schema: terra_pinyin

中文环境
========

- ``vim /etc/locale.gen``\ ，指定zh_CN.UTF-8
- ``vim /etc/local.conf``

.. code-block::

    LANG=zh_CN.UTF-8 
    LC_MESSAGES=zh_CN.UTF-8 

- 执行locale-gen
- vim /etc/rc.conf

.. code-block::

    LOCALE=zh_CN.UTF-8

locale-gen时找不到character map file
------------------------------------

``pacman -Syu``\ 升级的时候出的问题

执行locale-gen提示：\ ``character map file "en_US" not found``

结果locale就变成”C”

``pacman -S glibc``\ 重装一遍，还是出错，不过提示空间不足

``pacman -Scc`` 清理空间

``pacman -S glibc -f``

网络
====

根据MAC地址固定网卡名称
-----------------------

编辑文件/etc/udev/rules.d/10-network.rules

.. code-block::
    SUBSYSTEM==“net”,
    ATTR{address}==“00:26:2d:f6:ad:43”, NAME=“eth0” SUBSYSTEM==“net”,
    ATTR{address}==“70:f1:a1:28:5a:ad”, NAME=“wlan0” 

NetworkManager
--------------

安装并启用
~~~~~~~~~~

.. code-block:: console

   # pacman -S networkmanager
   # systemctl enable --now NetworkManager.service

NetworkManager 自带命令行工具 ``nmcli`` 和文本界面 ``nmtui``。使用桌面环境时，
GNOME 可以直接使用系统网络设置；KDE Plasma 可以安装 ``plasma-nm``；其他桌面环境
可以安装 ``network-manager-applet``。

不要同时启用其他独立管理同一网络接口的服务。安装前已经配置过网络服务时，先检查：

.. code-block:: console

   $ systemctl list-unit-files --state=enabled | grep -E 'NetworkManager|dhcpcd|systemd-networkd|wpa_supplicant'

常用命令
~~~~~~~~

.. code-block:: console

   $ nmcli general status                 # 查看 NetworkManager 状态
   $ nmcli device status                  # 查看网络设备及连接状态
   $ nmcli connection show                # 查看所有连接配置
   $ nmcli connection show --active       # 查看活动连接
   $ nmcli connection up "<连接名称>"       # 启用已有连接
   $ nmcli connection down "<连接名称>"     # 停用连接
   $ nmcli connection delete "<连接名称>"   # 删除连接配置

也可以运行 ``nmtui``，在文本界面中新增、编辑和启用连接。

连接无线网络
~~~~~~~~~~~~

.. code-block:: console

   $ nmcli radio wifi on
   $ nmcli device wifi list
   $ nmcli --ask device wifi connect "<SSID>"

``--ask`` 会交互式询问密码，避免密码出现在 shell 历史记录中。连接隐藏的无线网络：

.. code-block:: console

   $ nmcli --ask device wifi connect "<SSID>" hidden yes

成功连接后会保存连接配置，并默认在以后自动连接。可以手工控制自动连接：

.. code-block:: console

   $ nmcli connection modify "<连接名称>" connection.autoconnect yes

配置静态 IPv4 地址
~~~~~~~~~~~~~~~~~~

先用 ``nmcli connection show`` 找到连接名称，然后执行：

.. code-block:: console

   $ nmcli connection modify "<连接名称>" \
       ipv4.method manual \
       ipv4.addresses 192.168.1.10/24 \
       ipv4.gateway 192.168.1.1 \
       ipv4.dns "1.1.1.1 8.8.8.8"
   $ nmcli connection up "<连接名称>"

恢复使用 DHCP：

.. code-block:: console

   $ nmcli connection modify "<连接名称>" \
       ipv4.method auto ipv4.addresses "" ipv4.gateway "" ipv4.dns ""
   $ nmcli connection up "<连接名称>"

故障排查
~~~~~~~~

.. code-block:: console

   $ systemctl status NetworkManager.service
   $ journalctl -b -u NetworkManager.service
   $ nmcli device show
   $ nmcli networking connectivity check

修改 ``/etc/NetworkManager/system-connections/`` 中的连接文件后，需要重新加载配置：

.. code-block:: console

   # nmcli connection reload

vpn
---

- 参考：\ `archlinux pptp
  vpn拨号连接 <http://blog.vkill.net/read.php?97>`__
- 没看到arch下有/etc/ppp/ip-up.d目录，用以下脚本来启动vpn，我没有把它加为开机启动项，嗯。

.. code-block::
 #!/bin/zsh

#取网关地址 gateway=\ ``route|grep default|grep eth0|awk '{print $2;}'``
vpn_gateway=“192.168.6.253”

echo “拨号…” sudo poff -a sleep 2 sudo pon lab sleep 3

echo “修改路由…” #科大的地址不从vpn走 sudo route add -net 202.38.0.0/16
gw $gateway eth0 sudo route add -net 210.45.0.0/16 gw $gateway eth0 sudo
route add -net 211.86.0.0/16 gw $gateway eth0

#默认从vpn走 sudo route del default sudo route add default gw
$vpn_gateway dev ppp0

#看路由 sudo route -n 

无线速度慢 wireless slow
------------------------

参考\ `Slow Wireless Intel 6235 (iwlwifi
module) <https://bbs.archlinux.org/viewtopic.php?id=146518>`__

.. code-block::
 echo options iwlwifi 11n_disable=1 \| sudo tee
/etc/modprobe.d/51-disable-6235-11n.conf 

禁用 wifi 键盘灯 LED 闪烁
-------------------------

见：\ `wireless led
blink <https://wiki.archlinux.org/index.php/Wireless_Setup_%28%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87%29#.E7.A6.81.E7.94.A8_LED_.E9.97.AA.E7.83.81>`__

.. code-block::
 # echo ‘options iwlwifi led_mode=1’ >>
/etc/modprobe.d/wlan.conf # modprobe -r iwlwifi && modprobe iwlwifi {%
endhighlight %} 或者 .. code-block::
 # echo ‘w
/sys/class/leds/phy0-led/trigger - - - - phy0radio’ >
/etc/tmpfiles.d/phy0-led.conf # systemd-tmpfiles –create phy0-led.conf


常用软件
========

.. code-block::
 pacman -S rsync curl lftp wget axel pacman -S
wqy-bitmapfont wqy-zenhei ttf-monaco pacman -S smplayer ffmpeg
flashplayer pacman -S libreoffice-zh-CN libreoffice-impress
libreoffice-writer libreoffice-calc pacman -S zip unzip unrar p7zip
thunar-archive-plugin xarchiver arj cpio lzop pacman -S firefox
firefox-i18n-zh-cn freshplayerplugin pepper-flash chromium pacman -S
dnsutils traceroute wireshark-gtk 

yay
---

.. code-block::

   cd /tmp
   git clone https://aur.archlinux.org/yay-bin.git
   cd yay-bin
   makepkg -si

安装deb包
---------

.. code-block::

   yay -S debtap
   sudo debtap -u
   debtap xxx.deb
   sudo pacman -U xxx.zst

Web: NGINX+PHP
--------------

.. code-block::
 pacman -S nginx spawn-fcgi php-cgi {% endhighlight
%}

- 以http(用户):http(组)启动fastcgi :

.. code-block::
 spawn-fcgi -a 127.0.0.1 -p 9000 -C 5 -u http -g
http -f /usr/bin/php-cgi 

- 配置/etc/nginx/conf/nginx.conf

.. code-block::
 location / { root /var/www; index index.php
index.html index.htm; }

location ~ .php$ { root /var/www; fastcgi_pass 127.0.0.1:9000;
fastcgi_index index.php; fastcgi_param SCRIPT_FILENAME
:math:`document_root`\ fastcgi_script_name; include fastcgi_params; } {%
endhighlight %}

- 启动nginx: /etc/rc.d/nginx start

music
-----

`cue_splitting <https://wiki.archlinux.org/index.php/CUE_Splitting>`__

.. code-block::
 pacman -S cuetools mp3info wavpack flac mac shntool
bchunk 

firefox 安装 flash 插件
-----------------------

- 在 adobe 网站下载 flash player
- 将其中的libflashplayer.so 复制到 ~/.mozilla/plugins/目录下
- 执行 ldd ~/.mozilla/plugins/libflashplayer.so

virtualbox
----------

参考： `Install VirtualBox on Arch
Linux <https://linuxhint.com/install-virtualbox-arch-linux/>`__

.. code-block::

   bios -> cpu 设置 -> 打开 amd-v

   pacman -S virtualbox  选择 virtualbox-host-modules-arch

   reboot

   modprobe vboxdrv

   yay -S virtualbox-ext-oracle

   usermod -G vboxusers -a [username]

java
----

查看支持的java版本：

.. code-block::

   archlinux-java status

切换java的版本：

.. code-block::

   archlinux-java set <JAVA_ENV_NAME>

steam
-----

install
~~~~~~~

`How to Play Games on Arch Linux Using Steam: A Complete
Guide <https://linuxiac.com/how-to-play-games-on-arch-linux-using-steam/>`__

在 /etc/pacman.conf 开启 multilib

.. code-block::

   pacman -S steam

overlay
~~~~~~~

`SteamDeck上SteamOS+Windows双系统共用原神游戏文件的方法 <https://www.bilibili.com/read/cv20278885/>`__

双系统环境，linux下复用windows已下已安装的游戏文件，避免重复下载，同时避免修改原windows下的游戏文件。

windows下steamapps路径已挂载为 ``/mnt/d/game/Steam/steamapps``

linux下steamapps路径为 ``/home/xxx/.local/share/Steam/steamapps``

linux下steamapps的overlay upperdir路径为
``/home/xxx/game/steamapps_upper``

linux下steamapps的overlay workdir路径为
``/home/xxx/game/steamapps_work``

在/etc/fstab中添加配置：

.. code-block::

   overlay    /home/xxx/.local/share/Steam/steamapps overlay    defaults,nofail,index=off,lowerdir=/mnt/d/game/Steam/steamapps,upperdir=/home/xxx/game/steamapps_upper,workdir=/home/xxx/game/steamapps_work    0   2

挂载目录

.. code-block::

   sudo mount /home/xxx/.local/share/Steam/steamapps

play
~~~~

启动steam，在设置项的兼容性中选择proton版本，一般可默认为proton
experimental。

直接游玩相关游戏。

witcher 3
~~~~~~~~~

`steam
witcher3 <https://steamcommunity.com/app/221410/discussions/0/3731827719870175864/?ctp=2>`__

Witcher 3需要特殊配置proton版本选8.0左右，右键属性中设置启动选项：

.. code-block::

   WINEDLLOVERRIDES="winmm,version=n,b" %command% --launcher-skip 

效果一般，容易僵住

也可以尝试使用第三方的GE-Proton：

.. code-block::

   yay -S protonup-qt

系统备份
--------

使用
`clonezilla <http://drbl.nchc.org.tw/clonezilla/clonezilla-live/download/>`__
备份/恢复 磁盘或分区

故障处理
========

KERNEL PANIC 恢复
-----------------

- 系统升级失败，重启提示kernel panic，switch_root : fail to …
- 从live cd启动，将原来系统的根分区挂载到/mnt下，再用旧版glibc恢复

.. code-block::

    mount /dev/sda1 /mnt 
    pacman -U glibc-2.16.0-1-x86_64.pkg.tar.xz -r /mnt 

/bin/plymouth: No such file or directory
----------------------------------------

archlinux , thinkpad x61t，开机出错

journalctl -xn显示 /bin/plymouth: No such file or directory

删掉 /etc/fstab 中不存在的介质

闪屏
----

thinkpad x12 detachable, intel cpu, 闪屏

`Screen_flickering <https://wiki.archlinux.org/title/Intel_graphics#Screen_flickering>`__

arch.conf开机引导的options项中添加\ ``i915.enable_psr=0``

no sound card
-------------

thinkpad x12 detachable, intel tiger cpu, 没有声音

.. code-block::

   lspci -knn | grep -A2 audio

arch.conf开机引导的options项中添加\ ``snd_hda_intel.dmic_detect=0``

pacman segment fault
--------------------

将 /etc/pacman.conf 中的 XferCommand 注释掉

yay 提示 libalpm.so.13 库文件丢失\`
-----------------------------------

.. code-block::

   sudo ln -sf /usr/lib/libalpm.so.14 /usr/lib/libalpm.so.13

nvidia闪屏
-----------

.. code-block::

    xfconf-query -c xfwm4 -p /general/vblank_mode

    xfconf-query -c xfwm4 -p /general/vblank_mode -s off
