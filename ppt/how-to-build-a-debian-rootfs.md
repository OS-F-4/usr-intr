# 如何构建一个 debian 文件系统

 ## 依赖

```bash
apt-get install -y debootstrap
```

## 指令

```bash
debootstrap --components=main,universe bullseye ./rootfs
```

## 操作

```bash
cd rootfs
chroot .
# Do anything in this new rootfs! Such as install gcc...
```

## 用到 qemu 中

记得在根目录下新建一个可执行文件 `init`，内容如下：

```bash
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
echo -e "\nBoot took $(cut -d' ' -f1 /proc/uptime) seconds\n"
exec /bin/sh
```

