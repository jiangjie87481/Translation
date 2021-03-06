
# Arch和CentOS下的linux安装指南

###USB设备的配置

linux 用户需要支持USB—JTAG仿真器
在linux架构下，用下列命令完成uucp对plugdev组的替代
用sudo模式运行ls命令保证其正常执行
<div class="host-code"></div>

```sh
sudo ls
```

在有sudo权限下运行下列命令：
<div class="host-code"></div>

```sh
cat > $HOME/rule.tmp <<_EOF
# All 3D Robotics (includes PX4) devices
SUBSYSTEM=="usb", ATTR{idVendor}=="26AC", GROUP="plugdev"
# FTDI (and Black Magic Probe) Devices
SUBSYSTEM=="usb", ATTR{idVendor}=="0483", GROUP="plugdev"
# Olimex Devices
SUBSYSTEM=="usb",  ATTR{idVendor}=="15ba", GROUP="plugdev"
_EOF
sudo mv $HOME/rule.tmp /etc/udev/rules.d/10-px4.rules
sudo /etc/init.d/udev restart
```

将user加入到plugdev组中：
<div class="host-code"></div>

```sh
sudo usermod -a -G plugdev $USER
```

# 非标linux系统下的安装指南
CentOs 版linux操作系统
因为编译需要安装Python2.7.5版本，因此选择Centos7版本（因为早期的Centos发行版中自带安装了phython v2.7.5， 但是不建议这样做，因为可能后期通过yum来安装）
打开libftdi-devel 和 libftdi-python 需要EPEL库的支持。下面是安装的操作指令

<div class="host-code"></div>

```sh
wget https://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
sudo yum install epel-release-7-5.noarch.rpm
yum update
yum groupinstall “Development Tools”
yum install python-setuptools
easy_install pyserial
easy_install pexpect
yum install openocd libftdi-devel libftdi-python python-argparse flex bison-devel ncurses-devel ncurses-libs autoconf texinfo libtool zlib-devel cmake
```
注意：你也可能安装python-pip

#一些需要的32位库
当arm的开发工具链安装好时，可以按如下办法进行测试

<div class="host-code"></div>

```sh
arm-none-eabi-gcc --version
```
如果你收到了如下信息

<div class="host-code"></div>

```sh
bash: gcc-arm-none-eabi-4_7-2014q2/bin/arm-none-eabi-gcc: /lib/ld-linux.so.2: bad ELF interpreter: No such file or directory
```
那么你将需要另外一个32位的库：glibc.i686 ncurses-lib.i686

<div class="host-code"></div>

```sh
sudo yum install glibc.i686 ncurses-libs.i686 
```


安装ncurses-lib.i686 将会自带的安装其他所需要的32位库。Centos7会安装大部分px4相关的设备而不需要添加任何设备管理规则。设备能够在预告定义好的'dialou'组中查看到。因此任何其他添加设备管理规则的行为都会被忽视。这样做的唯要要求就是你的使用帐户在'dial out'组中。

# Arch 版 linux

<div class="host-code"></div>

```sh
sudo pacman -S base-devel lib32-glibc git-core python-pyserial zip python-empy
```

安装yaourt，一种包管理软件。下载地址。https://wiki.archlinux.org/index.php/Yaourt#Installation。下完后按以下步骤编译安装

<div class="host-code"></div>

```sh
yaourt -S genromfs
```
权限:用户需要加入到uucp组：

<div class="host-code"></div>

```sh
sudo usermod -a -G uucp $USER
```
然后登出再登入获得权限


<aside class="note">
Log out and log in for changes to take effect! Also remove the device and plug it back in!**
</aside>


#工具链安装

运行下面的脚本安装 gcc4.8.4或4.9.2

<div class="host-code"></div>

```sh
pushd .
cd ~
wget https://launchpadlibrarian.net/186124160/gcc-arm-none-eabi-4_8-2014q3-20140805-linux.tar.bz2
tar -jxf gcc-arm-none-eabi-4_8-2014q3-20140805-linux.tar.bz2
exportline="export PATH=$HOME/gcc-arm-none-eabi-4_8-2014q3/bin:\$PATH"
if grep -Fxq "$exportline" ~/.profile; then echo nothing to do ; else echo $exportline >> ~/.profile; fi
. ~/.profile
popd
```

GCC 4.9:

<div class="host-code"></div>

```sh
pushd .
cd ~
wget https://launchpad.net/gcc-arm-embedded/4.9/4.9-2014-q4-major/+download/gcc-arm-none-eabi-4_9-2014q4-20141203-linux.tar.bz2
tar -jxf gcc-arm-none-eabi-4_9-2014q4-20141203-linux.tar.bz2
exportline="export PATH=$HOME/gcc-arm-none-eabi-4_9-2014q4/bin:\$PATH"
if grep -Fxq "$exportline" ~/.profile; then echo nothing to do ; else echo $exportline >> ~/.profile; fi
. ~/.profile
popd
```

<aside class="note">
如果是Debian Linux，则运行以下命令
</aside>

<div class="host-code"></div>

```sh
sudo dpkg --add-architecture i386
sudo apt-get update
```

安装32位的支持库

<div class="host-code"></div>

```sh
sudo apt-get install libc6:i386 libgcc1:i386 gcc-4.6-base:i386 libstdc++5:i386 libstdc++6:i386
```

# Ninja 编译系统

Ninja运行起来比Make要快，而且px4的cmake产生器支持它。不幸的是ubuntu所带的版本很老。最近在版本在[Ninja]“(https://github.com/martine/ninja)“中，下载下来然后将其二进制运行文件加入到你的运行路径中即可

<div class="host-code"></div>

```sh
mkdir -p $HOME/ninja
cd $HOME/ninja
wget https://github.com/martine/ninja/releases/download/v1.6.0/ninja-linux.zip
unzip ninja-linux.zip
rm ninja-linux.zip
exportline="export PATH=$HOME/ninja:\$PATH"
if grep -Fxq "$exportline" ~/.profile; then echo nothing to do ; else echo $exportline >> ~/.profile; fi
. ~/.profile
```
版本测试

Enter:

<div class="host-code"></div>

```sh
arm-none-eabi-gcc --version
```

输出应当类似于：

<div class="host-code"></div>

```sh
arm-none-eabi-gcc (GNU Tools for ARM Embedded Processors) 4.7.4 20140401 (release) [ARM/embedded-4_7-branch revision 209195]
Copyright (C) 2012 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```
如果得到的是：

<div class="host-code"></div>

```sh
arm-none-eabi-gcc --version
arm-none-eabi-gcc: No such file or directory
```
确认你的32位库在前述的说明步骤下已正常安装
