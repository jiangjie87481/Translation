# 编译px４软件

PX4可以在控制台或者图形界面/IDE开发

## 在控制台编译
在去到图形界面或者IDE前，验证系统设置的正确性非常重要，因此打开控制台。在 OS X, 敲击 ⌘-space ，并搜索'terminal'。在Ubuntu，单击启动栏，搜索“terminal”（或者trl+alt+T）。在windows平台，在开始菜菜单找到px4文件夹，单击'PX4 Console'

![building](../pictures/toolchain/terminal.png)

终端在家目录启动，我们默认去到'~/src/Firmware' 然后，克隆顶层资源库。有经验的开发者可以克隆自己的复制的[资源库](https://help.github.com/articles/fork-a-repo/) 

<div class="host-code"></div>

```sh
mkdir -p ~/src
cd ~/src
git clone https://github.com/PX4/Firmware.git
cd Firmware
git submodule update --init --recursive
cd ..
```

现在可以由源代码编译二进制代码了。但在直接使用硬件前，推荐先[进行仿真](../4_Simulation/basic_simulation.md)。喜欢在图形界面开发环境工作的用户也应该继续完成下面部分。


###基于NuttX / Pixhawk的硬件板

<div class="host-code"></div>

```sh
cd Firmware
make px4fmu-v2_default
```

注意到“make”是一个字符命令编译工具，“px4fmu-v2”是硬件/ardupilot版本，“default”是默认配置，所有的PX4编译目标遵循这个规则。 

成功编译的最后输出是这样的：

<div class="host-code"></div>

```sh
[100%] Linking CXX executable firmware_nuttx
[100%] Built target firmware_nuttx
Scanning dependencies of target build_firmware_px4fmu-v2
[100%] Generating nuttx-px4fmu-v2-default.px4
[100%] Built target build_firmware_px4fmu-v2
```

通过在命令后面添加‘upload’，编译的二进制程序就会通过USB上传到飞控硬件:

<div class="host-code"></div>

```sh
make px4fmu-v2_default upload
```

成功运行后将会输出:

<div class="host-code"></div>

```sh
Erase  : [====================] 100.0%
Program: [====================] 100.0%
Verify : [====================] 100.0%
Rebooting.

[100%] Built target upload
```

### Raspberry Pi 2 boards

下面命令将生成为Raspbia(posix_pi2发行版)的目标。

<div class="host-code"></div>

```sh
cd Firmware
make posix_rpi2_release # for cross-compiler build
```

The "mainapp" executable file is in the directory build_posix_rpi2_release/src/firmware/posix.
Copy it over to the RPi (replace YOUR_PI with the IP or hostname of your RPi, [instructions how to access your RPi](../5_Autopilot-Hardware/raspeberry_pi2.md#developer-quick-start))
'mainapp'可执行文件所在目录为：build_posix_rpi2_release/src/firmware/posix。将其拷到RPi（用IP或你RPi的主机名替换YOU_PI）。 如何得到你的RPi的说明在../5_Autopilot-Hardware/raspeberry_pi2.md#developer-quick-start中

```sh
scp build_posix_rpi2_release/src/firmware/posix/mainapp pi@YOUR_PI:/home/pi/
```

接着运行
<div class="host-code"></div>

```sh
./mainapp
```

If you're building *directly* on the Pi, you will want the native build target (posix_pi2_default).
如果你就在pi上进行编译链接，你将得到本机目标（默认的poxix_pi2_default）。

<div class="host-code"></div>

```sh
cd Firmware
make posix_rpi2_default # for native build
```

mainapp可执行文件所在目录为，按如下命令运行。
<div class="host-code"></div>

```sh
./build_posix_rpi2_default/src/firmware/posix/mainapp
```

成功编译后再运行mainapp你将得到以下结果：
```sh
[init] shell id: 1996021760
[init] task name: mainapp

______  __   __    ___
| ___ \ \ \ / /   /   |
| |_/ /  \ V /   / /| |
|  __/   /   \  / /_| |
| |     / /^\ \ \___  |
\_|     \/   \/     |_/

Ready to fly.


pxh>
```

QuRT / Snapdragon 开发板

编译

下面面命令编译了能在linux与DSP上使用的目标.所有的可执行文件通信通过
[muORB](../6_Middleware-and-Architecture/uorb_messaging.md).

<div class="host-code"></div>

```sh
cd Firmware
make eagle_default
```

为了将SW下载到设备上，通过usb线连上然后确信设备已经boot启动了。在新的终端运行如下命令：

<div class="host-code"></div>

```sh
adb shell
```

回到之前的终端然后上传：

<div class="host-code"></div>

```sh
make eagle_default upload
```

<aside class="note">
注：这样的操作会拷贝或覆盖两个配置文件： [mainapp.config](https://github.com/PX4/Firmware/blob/master/posix-configs/eagle/flight/mainapp.config) and [px4.config](https://github.com/PX4/Firmware/blob/master/posix-configs/eagle/flight/px4.config) to the device. 文件存储目录： /usr/share/data/adsp/px4.config and /home/linaro/mainapp.config 你可以为你的设备（飞行器）修改它们.
</aside>

目前mixer需要手动复制：
<div class="host-code"></div>

```
adb push ROMFS/px4fmu_common/mixers/quad_x.main.mix  /usr/share/data/adsp
```

运行：

运行DSP监视:

<div class="host-code"></div>

```sh
${HEXAGON_SDK_ROOT}/tools/mini-dm/Linux_Debug/mini-dm
```

返回到ADB shell然后运行mainapp:
```sh
cd /home/linaro
./mainapp mainapp.config
```
注意，mainapp将会停止一但你断开usb连接线（或者你的ssh端断开连接）。为飞行，你可以将mainapp设置为boot启动后自启

自动运行mainapp

为使得mainapp能够在Snapdragon boot后能马上运行，你可以在将启动加到‘rc.local’文件中：

```sh
adb shell
vim /etc/rc.local
```

或者将文件拷到你的电脑中，编辑完后再拷回去：

```sh
adb pull /etc/rc.local
gedit rc.local
adb push rc.local /etc/rc.local
```

为了能够自启动，在'exit 0 '列前，加上下列命令：

```
(cd /home/linaro && ./mainapp mainapp.config > mainapp.log)

exit 0
```

确信‘rc.local’文件是可执行的：

```
adb shell
chmod +x /etc/rc.local
```

重启 Snapdragon:

```
adb reboot
```

在图形界面的集成开发环境IDE中进行编译

PX4系统支持QT creator, Eclipse以及Sublime Text等集成发环境。QT Creator      是一种用起来比较好的一款，因此也是官方唯一支持的IDE。除非你是Eclipse及Sublime的专家，否则他们用起来一点也不爽。集成开发环境的下载如下：
 [Eclipse project](https://github.com/PX4/Firmware/blob/master/.project) and a [Sublime project](https://github.com/PX4/Firmware/blob/master/Firmware.sublime-project) in the source tree.

{% youtube %}https://www.youtube.com/watch?v=Bkk8zttWxEI&rel=0&vq=hd720{% endyoutube %}

Qt Creator功能

Qt creator提供可查询的符号，自动完成代码库的更新及完成固件的生成。

![qtcreator](../pictures/toolchain/qtcreator.png)

在Linux 上使用 Qt Creator

在启动 Qt Creator,需要将工程 [project file](https://cmake.org/Wiki/CMake_Generator_Specific_Information#Code::Blocks_Generator) 建立起来:

<div class="host-code"></div>

```sh
cd ~/src/Firmware
mkdir ../Firmware-build
cd ../Firmware-build
cmake ../Firmware -G "CodeBlocks - Unix Makefiles"
```

Then load the CMakeLists.txt in the root firmware folder via File -> Open File or Project -> Select the CMakeLists.txt file.

After loading, the 'play' button can be configured to run the project by selecting 'custom executable' in the run target configuration and entering 'make' as executable and 'upload' as argument.

在Window下运行Qt Creator

<aside class="todo">
还未进行过测试。
</aside>

在Mac OS运行Qt Creator

在启动Qt Creator之前, 工程[project file](https://cmake.org/Wiki/CMake_Generator_Specific_Information#Code::Blocks_Generator) 需要先建立起来:

<div class="host-code"></div>

```sh
cd ~/src/Firmware
mkdir build_creator
cd build_creator
cmake .. -G "CodeBlocks - Unix Makefiles"
```

正是如此，启动Qt Creator，然后完成视频中的后续步骤设置好需要编译的工程。

{% youtube %}https://www.youtube.com/watch?v=0pa0gS30zNw&rel=0&vq=hd720{% endyoutube %}
