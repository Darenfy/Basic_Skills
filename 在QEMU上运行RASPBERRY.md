我们使用 Ubuntu 系统并且搭建我们所期望的仿真环境

获取最新版的 Ubuntu 系统，并且在虚拟机中运行
- https://www.ubuntu.com/download/desktop

为了搭建 QEMU 仿真环境，我们需要以下文件：
1. 树莓派镜像：http://downloads.raspberrypi.org/raspbian/images/raspbian-2017-04-10/ （其他版本可能会好使，但是推荐 Jessie 
2. 最新的 qemu 内核：https://github.com/dhruvvyas90/qemu-rpi-kernel

进入虚拟机，创建文件夹：
```
$ mkdir ~/qemu_vms/
```

下载树莓派镜像并放在 ~/qemu_vms/ 文件夹中

下载 qemu-kernel 并放在 ~/qemu_vms/ 文件夹中

```
$ sudo apt-get install qemu-system
$ unzip <image-file>.zip
$ fdisk -l <image-file>
```

你将会看到类似的信息：
```
Disk 2017-03-02-raspbian-jessie.img: 4.1 GiB, 4393533440 bytes, 8581120 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x432b3940

Device                          Boot  Start     End Sectors Size Id Type
2017-03-02-raspbian-jessie.img1        8192  137215  129024  63M  c W95 FAT32 (LBA)
2017-03-02-raspbian-jessie.img2      137216 8581119 8443904   4G 83 Linux
```

可以看到文件系统 .img2 在 137216 启动。现在用这个值乘 512 。
然后在下面的命令中使用这个值作为偏移
```
$ sudo mkdir /mnt/raspbian
$ sudo mount -v -o offset=70254592 -t ext4 ~/qemu_vms/<your-img-file.img> /mnt/raspbian
$ sudo nano /mnt/raspbian/etc/ld.so.preload
```
用“＃”注释掉该文件中的每个条目，保存并使用Ctrl-x»Y退出。
```
sudo nano /mnt/raspbian/etc/fstab
```
如果在fstab中看到带有 mmcblk0 的任何内容，则：
1. 用/ dev / sda1替换包含/ dev / mmcblk0p1的第一个条目
2. 用/ dev / sda2替换包含/ dev / mmcblk0p2的第二个条目，保存并退出。
```
$ cd ~
$ sudo umount /mnt/raspbian
```
现在，您可以使用以下命令在 Qemu 上运行树莓派
```
$ qemu-system-arm -kernel ~/qemu_vms/<your-kernel-qemu> -cpu arm1176 -m 256 -M versatilepb -serial stdio -append "root=/dev/sda2 rootfstype=ext4 rw" -hda ~/qemu_vms/<your-jessie-image.img> -redir tcp:5022::22 -no-reboot
```
<!--qemu-system-arm -kernel ~/qemu_vms/kernel-qemu-4.4.34-jessie -cpu arm1176 -m 256 -M versatilepb -serial stdio -append "root=/dev/sda2 rootfstype=ext4 rw" -hda ~/qemu_vms/raspbian-jessie.img -net nic -net user,hostfwd=tcp::5022-:22-->

如果看到 Raspbian OS 的GUI，则需要进入终端。 使用 Win 键获取菜单，然后使用箭头键导航，直到找到如下所示的 Terminal 应用程序
![](https://azeria-labs.com/wp-content/uploads/2017/03/raspbian-terminal.png.pagespeed.ce.xjhvHXkYgm.png)

从终端，您需要启动 SSH 服务，以便可以从主机系统（启动 qemu 的主机系统）访问它
![](https://azeria-labs.com/wp-content/uploads/2017/03/raspbian-ssh.png.pagespeed.ce.zHzvnRc16E.png)

现在，您可以使用主机名（默认密码– raspberry）从主机系统 SSH 进行访问

```
$ ssh pi@127.0.0.1 -p 5022
```

**故障排除**
如果默认情况下在启动时未在 Emulator 中启动 SSH ，则可以使用以下命令在 Pi 终端中更改它：
```
$ sudo update-rc.d ssh enable
```

如果您的仿真Pi启动了GUI，并且您想使其在启动时以控制台模式启动，请在Pi终端内使用以下命令：
```
$ sudo raspi-config
>Select 3 – Boot Options
>Select B1 – Desktop / CLI
>Select B2 – Console Autologin
```

如果您的鼠标在模拟的Pi中没有移动，请单击<Windows>，向下箭头至 Accessories ，向至“终端”，然后输入

**扩容树莓派镜像**

```
//创建镜像副本
$ cp <your-raspbian-jessie>.img rasbian.img
//扩容
$ qemu-img resize raspbian.img +6G
//重新启动
$ sudo qemu-system-arm -kernel ~/qemu_vms/<kernel-qemu> -cpu arm1176 -m 256 -M versatilepb -serial stdio -append "root=/dev/sda2 rootfstype=ext4 rw" -hda ~/qemu_vms/<your-original-raspbian-jessie>.img -redir tcp:5022::22 -no-reboot -hdb raspbian.img

//删掉 sdb2  建立新分区  pri - write
$ sudo cfdisk /dev/sdb 

//调整大小 检查旧分区并关机
$ sudo resize2fs /dev/sdb2
$ sudo fsck -f /dev/sdb2
$ sudo halt

//用新的 img 启动
sudo qemu-system-arm -kernel ~/qemu_vms/kernel-qemu-4.4.34-jessie -cpu arm1176 -m 256 -M versatilepb -serial stdio -append "root=/dev/sda2 rootfstype=ext4 rw" -hda ~/qemu_vms/raspi.img -net nic -net user,hostfwd=tcp::5022-:22 -no-reboot
```
<!--qemu-system-arm -kernel ~/qemu_vms/kernel-qemu-4.4.34-jessie -cpu arm1176 -m 256 -M versatilepb -serial stdio -append "root=/dev/sda2 rootfstype=ext4 rw" -hda ~/qemu_vms/raspbian-jessie.img -net nic -net user,hostfwd=tcp::5022-:22 -no-reboot -hdb raspi.img-->
**高级网络**
Host:
```
azeria@labs:~ $ sudo apt-get install uml-utilities
azeria@labs:~ $ sudo tunctl -t tap0 -u azeria
azeria@labs:~ $ sudo ifconfig tap0 172.16.0.1/24
```
你可以在 ifconfig 的输出里看到 tap0 接口
```
azeria@labs:~ $ ifconfig tap0
tap0: flags=4099<UP,BROADCAST,MULTICAST> mtu 1500
inet 172.16.0.1 netmask 255.255.255.0 broadcast 172.16.0.255
ether 22:a8:a9:d3:95:f1 txqueuelen 1000 (Ethernet)
RX packets 0 bytes 0 (0.0 B)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 0 bytes 0 (0.0 B)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
```
然后可以再次启动
```
$ sudo qemu-system-arm -kernel ~/qemu_vms/kernel-qemu-4.4.34-jessie -cpu arm1176 -m 256 -M versatilepb -serial stdio -append "root=/dev/sda2 rootfstype=ext4 rw" -hda ~/qemu_vms/raspbian.img -net nic -net tap,ifname=tap0,script=no,downscript=no -no-reboot
```
在树莓派里更改 eth0 的配置
```
pi@labs:~ $ sudo ifconfig eth0 172.16.0.2/24
```
如果一切顺利，可以从主机（Ubuntu）系统访问GUEST（Raspbian）上的开放端口。 您可以使用netcat（nc）工具对此进行测试（请参见下面的示例）
![](https://azeria-labs.com/wp-content/uploads/2017/03/qemu_advanced_networking.png.pagespeed.ce.oPfykYICtN.png)



参考资料：
https://www.jianshu.com/p/da00aea5d666
https://api-caller.com/2019/04/30/ARM-note/

ssh Host key verification Failed:
https://blog.csdn.net/wd2014610/article/details/85639741