# ESPRESSObin U-boot

U-boot源代码：https://github.com/chenhaninformation/u-boot
U-boot分支：u-boot-2017.03-armada-17.10-ch-dev

ATF源代码：https://github.com/chenhaninformation/arm-trusted-firmware
ATF分支：atf-v1.3-armada-17.10-ch-dev

rootfs-extra源代码：https://github.com/chenhaninformation/rootfs-extra
rootfs-extra分支：master

## 编译U-boot

### 编译须知

ESPRESSObin板载ARM双核Cortex-A53 CPU，型号为Marvell Armada 3720。
ESPRESSObin开发板在U-boot中不支持内存初始化，进而初始化工作需要在
ARM-Trusted-Firmware（以下简称ATF）中。不同内存大小，编译选项不一样，生成的二
进制包也不一样，具体可查阅ATF文档。为方便快速出包，维护人员可访问rootfs-extra
项目查阅文档，编译U-boot包。

### 编译流程

将上述rootfs-extra clone到本地，并仔细阅读文档，在项目文件夹下，执行
make u-boot，即可生成适配所有内存型号的U-boot二进制包，访问目录，
${PROJECT-DIR}/build/image/u-boot/ 查阅编译好的U-boot固件，其中根目录文件下有
包含当前固件编译的信息，commit id及gcc版本等。

## 烧录U-boot

### U-boot版本

编译好的U-boot包，在rootfs-extra目录，./build/image/u-boot/目录下。其中，所有
版本的二进制包需要对应上内存版本，内存大小，内存片数，及U-boot烧录位置。
例如：有一片板子，是DDR3内存，1G大小，内存贴了两片，每一片为512M，且U-boot需要
烧录至SPI上，应该使用./ddr3/1g/2cs/spi/\* 下面的二进制包。

每个版本下面有两种二进制包，一种是直接烧录的二进制包flash-image.bin，另一种是
uart-images/\* 下面的UART包。对于裸机而言，应使用UART包。UART包下载方法可访问
http://wiki.espressobin.net/tiki-index.php?page=Bootloader+recovery+via+UART。
对于已有U-boot镜像正常运行在板子上，在主板开机时按键中断自动启动，按任意键中断
自动启动，进入U-boot控制台，可以在控制台里使用bubt对板载U-boot进行更新。访问
http://wiki.espressobin.net/tiki-index.php?page=Update+the+Bootloader查阅烧录
方法及流程。

## 修改内容

原有U-boot不能满足现有需求，针对U-boot的修改将是必须的了，相关修改请查阅U-boot
源代码文档，或commit信息，内部有详细讲述了修改信息。

