# 步骤
1. 更新termux软件
```bash
pkg update -y && pkg upgrade -y
```

2. 安装qemu
```bash
pkg install qemu-utils qemu-common qemu-system-x86_64-headless wget -y
```

3. 创建目录
```bash
mkdir alpine && cd alpine
```

4. 下载alpine镜像
```bash
wget http://dl-cdn.alpinelinux.org/alpine/v3.19/releases/x86_64/alpine-virt-3.19.1-x86_64.iso
```

5. 创建虚拟硬盘镜像
```bash
qemu-img create -f qcow2 alpine.img 60G
```

6. 初次启动
```bash
qemu-system-x86_64 \
    -machine q35 \
    -m 8196 \
    -smp cpus=6 \
    -cpu qemu64 \
    -drive if=pflash,format=raw,read-only=on,file=/data/data/com.termux/files/usr/share/qemu/edk2-x86_64-code.fd \
    -netdev user,id=n1,hostfwd=tcp::2222-:22 \
    -device virtio-net,netdev=n1 \
    -cdrom alpine-virt-3.19.1-x86_64.iso \
    -nographic alpine.img
```

7. 登录用户名：root

8. 设置网络（一路默认）
```bash
localhost:~# setup-interfaces
 Available interfaces are: eth0.
 Enter '?' for help on bridges, bonding and vlans.
 Which one do you want to initialize? (or '?' or 'done') [eth0]
 Ip address for eth0? (or 'dhcp', 'none', '?') [dhcp]
 Do you want to do any manual network configuration? [no]
```

localhost:~# 
```bash
ifup eth0
```

9. 修改DNS：
```bash
echo -e "nameserver 8.8.8.8" > /etc/resolv.conf
```

10. 创建answerfile（键盘为美式，时区为上海）：
```bash
vi answerfile
```
```bash
ANSWERFILE:
KEYMAPOPTS="us us"
HOSTNAMEOPTS="-n AlpineNote10+"
INTERFACESOPTS="auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp
"
DNSOPTS="8.8.8.8"
TIMEZONEOPTS="-z Asia/Shanghai"
PROXYOPTS="none"
APKREPOSOPTS="http://dl-cdn.alpinelinux.org/alpine/latest-stable/main http://dl-cdn.alpinelinux.org/alpine/latest-stable/community"
SSHDOPTS="-c openssh"
NTPOPTS="-c busybox"
DISKOPTS="-v -m sys -s 0 /dev/sda"
```

11. 修改``setup-disk``，以在系统启动时启用串行控制台输出：
sed -i -E 's/(local kernel_opts)=.*/\1="console=ttyS0"/' /sbin/setup-disk

12. 从CDROM用answerfile安装系统
setup-alpine -f answerfile

13. 安装好后用``poweroff``关机

14. 启动系统，这次不需要CDROM。这段命令可以存为.sh，记得要```chmod +x```:
```bash
#!/bin/bash

qemu-system-x86_64 \
    -machine q35 \
    -m 8196 \
    -smp cpus=6 \
    -cpu qemu64 \
    -drive if=pflash,format=raw,read-only=on,file=/data/data/com.termux/files/usr/share/qemu/edk2-x86_64-code.fd \
    -netdev user,id=n1,hostfwd=tcp::2222-:22 \
    -device virtio-net,netdev=n1 \
    -nographic alpine.img
```

15. 再次设定DNS
```bash
echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "nameserver 8.8.4.4" >> /etc/resolv.conf
```

16. 升级并安装docker
```bash
apk update && apk add docker
```

17. 启动docker:
```bash
service docker start
```

18. 将docker设为开机启动:
```bash
rc-update add docker
```

19. 检查docker是否安装成功:
```bash
docker run hello-world
```

20. 为了防止重启后dns又被改掉，重新```setup-interfaces```，配置为静态IP，不要dhcp了

21. 如果在宿主机（即termux）中通过ssh启动qemu，会话终止时虚拟机也会关闭，记得用screen保留会话

# 有用的快捷键
- ``Ctrl+a x``: 退出qemu虚拟机
- ``Ctrl+a h``: 激活qemu控制台

# 可爱的开机画面
```vi /etc/modt```

```bash
                            
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣀⣀⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣤⠤⠖⠋⠉⠉⠉⠉⠙⠓⠶⣄⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⡞⠉⠀⠀⠀⠀⣠⡤⠤⠤⣤⣄⠀⠈⠻⣦⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣰⠏⠀⠀⠀⠀⣠⠎⠁⠀⠀⠀⠀⠈⠙⢦⡀⠘⢷⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣸⠃⠀⠀⠀⠀⢰⠇⠀⠀⠀⠀⠀⠀⣀⡀⢄⣙⢦⡈⣧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⡟⠀⠀⠀⠀⠀⡿⠀⠀⠀⠀⠀⠀⠈⠁⠀⠀⠀⠈⢳⠸⣆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⡇⠀⢠⠀⠀⢰⡧⠊⠉⠀⠀⠀⠀⠀⣠⣾⣛⡷⢦⢸⡄⢻⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⡇⠀⣿⠀⠀⢸⠁⣠⣴⣦⡀⠀⠀⠀⠉⣿⣹⣧⠀⢸⠇⠘⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⣧⠀⣿⠀⠀⢸⣾⠋⣿⢿⣧⠀⠀⠀⠀⠻⠧⠿⠀⠘⣷⠀⣷⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢻⠀⢹⠀⠀⢸⣇⠀⢿⣿⣻⠇⠀⠀⠀⠀⠀⠀⠀⠀⣸⠂⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣾⠀⢸⡀⠀⢸⣿⡆⠀⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⣴⠋⠀⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⡟⠄⠸⡇⠀⠈⡟⣷⣀⠀⠀⠀⠀⠀⠈⢀⣠⠴⢺⡇⠀⠀⡿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣼⠁⠀⠀⣷⠀⠀⣷⠀⠉⠛⠒⠒⠒⣺⠿⠛⣷⢀⣸⡇⠀⣸⡃⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢰⡇⠀⠀⠀⠸⣇⠀⢸⡏⠉⠉⠹⡟⠚⠋⠀⠀⠉⠉⣿⠃⣠⠏⠹⡆⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⠀⠀⠀⠀⠀⢻⣆⠀⢻⣄⠀⠀⢿⠀⠀⠀⠀⠀⠒⠗⣎⠁⠀⠀⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⡆⠀⠀⠀⠀⠈⣟⠳⣄⣙⣷⠶⣾⡀⠀⠀⠀⠀⠀⠀⠘⣶⠀⠀⡿⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢷⡀⢸⡄⠀⠀⢿⡀⠀⠀⠀⠀⠹⡇⠀⠀⠀⢦⠀⠀⠀⠹⣦⢰⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠻⣼⣟⣦⡀⠉⣷⡀⠀⠀⠀⠀⢿⠀⠀⠀⠈⢷⠀⠀⠀⣿⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣻⠀⢸⣧⠀⠀⠀⠀⠘⣧⠀⠀⠀⢸⡄⠀⣰⣿⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠾⣏⣀⣸⠘⣧⡀⠀⠀⠀⣿⣷⣄⣀⣼⣁⣴⣿⣿⡏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣀⣼⠀⠸⣷⠀⠀⠀⠸⡿⠿⠛⠛⣻⠛⠉⠀⣷⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⡴⠖⠋⠉⠉⠉⠀⠀⠀⠀⠘⣧⠀⠀⠀⠹⡄⠀⣴⠿⣇⠀⠀⢹⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⣾⣿⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠸⣆⠀⠀⠀⢻⡾⠃⠀⣿⠀⠀⠈⣧⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⣼⣿⣿⣿⣧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣹⡄⠀⠀⠀⣿⠉⠉⢹⡆⠀⠀⢹⡦⠤⣄⡀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⣸⠋⠁⠀⠉⠙⢷⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠁⣿⡀⠀⠀⢸⡄⠀⠀⣷⠀⠀⠸⡇⠀⠀⠙⢢⣄⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⡟⠀⠀⠀⠀⠀⠀⠛⠓⠒⠒⠶⠤⣄⣀⣀⣠⣴⠏⢷⡀⠀⠀⢷⠀⠀⠸⣆⠀⠀⣷⠀⠀⠀⠀⠹⡆⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⣇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠙⠻⢥⣀⠈⢷⡀⠀⠸⡆⠀⠀⠻⡆⠀⢹⠀⠀⠀⠀⢠⡟⠀⠀
⠀⠀⠀⠀⠀⢀⡴⠖⠻⣆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠳⡌⢷⡀⠀⢻⡄⠀⠀⢻⠀⠸⢧⣀⠀⣠⡿⠁⠀⠀
⠀⠀⣀⣠⠴⠋⠀⠀⠀⠘⢷⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢹⡶⢷⠀⠀⠻⣶⠶⢾⡄⠀⢀⣈⠙⠿⠷⢖⣦⣄
⣠⡾⠉⠀⠀⠀⠀⠀⠀⠀⠀⠙⠳⢦⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⡇⢸⡆⠀⠀⣙⢧⡈⠛⠒⠒⠻⠽⠟⠉⠉⠉⠀
⢿⣞⡤⠴⠖⠚⠛⠛⠛⠛⠓⠢⠤⢄⣉⠛⠓⠦⠤⢤⣄⣀⣀⣀⣀⡀⠀⠀⣾⠃⡼⡥⡤⣠⠸⡷⠽⠷⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠉⠙⠒⠶⠤⣤⣀⣈⣉⣁⣀⣤⠞⠁⢮⣺⣼⣱⢻⣇⡿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠉⠉⠁⠀⠀⠀⠀⢀⡉⠀⠀⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀   
                                                       
                                                       
```

# Acknowledgment
This article inspired from:
- https://gist.github.com/oofnikj/e79aef095cd08756f7f26ed244355d62
- https://github.com/cyberkernelofficial/docker-in-termux
- https://github.com/AntonyZ89/docker-qemu-arm
- https://stageguard.top/2019/08/15/run-docker-on-qemu-alpine/#%C2%B7-%E9%85%8D%E7%BD%AE%E7%BD%91%E7%BB%9C%EF%BC%9A
























