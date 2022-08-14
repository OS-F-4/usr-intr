# 其他编译帮助



## 如何安装gcc-11

### ubuntu: 

```shell
# 安装gcc-11
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt install build-essential manpages-dev software-properties-common
sudo apt update && sudo apt install gcc-11 
```

切换默认的gcc到gcc-11

```shell
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 100
```

其中最后的数字100表示优先级, 只要比其他gcc高, 该gcc就会成为默认的gcc。



### centos:

```shell
yum install gcc-toolset-11
scl enable gcc-toolset-11 bash
```

centos 暗转工具链后可以通过`as -v`查看版本, 只要版本大于2.36就可以编译uintr程序, 否则需要手动编译安装或者通过更新软件包的方式升级汇编器。



## 如何安装binutils-2.38

从网上下载`binutils-2.38.tar.bz2`

```shell
tar -jxvf binutils-2.38.tar.bz2
cd binutils-2.38
./configure
make -j
sudo make install
```



