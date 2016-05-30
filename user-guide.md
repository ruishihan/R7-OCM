R7使用说明
==========================================
#本文分三部分介绍R7的使用方法：
1. 对SD卡进行分区并制作成系统盘
2. 启动ARM，并向R7上传文件
3. 
#本说明默认具备以下环境：
1. 文件夹root.full.function，可从指定网址下载
2. Ubuntu 14.04 LTS 系统，并且装有git、ssh、curl、Vivado_2014.4、gtkterm


# SD卡分区及系统盘制作
> 进入管理员模式：
	
        sudo su
> 通过fdisk -l命令找出SD可对应的设备文件：
	
        fdsik -l
> 对SD对应的设备文件进行操作，这里假设设备文件是/dev/sdb
- 假设SD卡未进行分区，则先对SD卡进行分区
	
        fdisk /dev/sdb
- 然后根据fdisk的命令提示进行操作，用n选项新建分区，第一个分区分配8G空间，剩余空间分给第二个分区(分配空间可自由分配，但不应小于***)
- 在建立两个分区后，用w选项将分区表写入SD卡，分区完成
- 重新进入fdisk /dev/sdb，用t选项改变分区格式，分区一格式化为W95 FAT32格式（Hex code 为 c），分区二保持linux格式不变。
* 如果SD卡已有分区，但分区大小不符合要求，则可以通过 d 命令删除分区。

> 将SD卡挂载后进入分区二（linux分区），执行下述命令：
	
        zcat /.../root.full.function/core-image-my-sdk-r7-zynq7-20150620151425.rootfs.cpio.gz |sudo cpio -i

> 将root.full.function 文件夹中的 BBOOOT.bin  devicetree.dtb  uEnv.txt  uImage uramdisk.image.gz四个文件拷贝到分区一中
* 至此，系统盘制作完成。ARM将先读取分区一中的启动文件，并启动程序，在启动后挂载分区二，并改为运行分区二中的程序。



# 启动ARM，并向R7上传文件


> 首先将R7的USB端口和网口分别同PC的USB端口和网口连接起来
> 通过gtkterm 打开两个终端，两个终端分别对应USB口的两条线路
        sudo -S gtkterm -p /dev/ttyUSB1 -s 115200 &
        sudo -S gtkterm -p /dev/ttyUSB0 -s 115200 
> 在其中一个端口中输入
        BootFromSD
> 另一个端口会显示ARM正在启动linux系统，等待启动完成后，输入用户名 root，进入系统
> 对PC和ARM的IP进行配置，注意一定要让两者在一个网段中，这里假设PC的IP地址为10.0.77.110, ARM的IP为10.0.77.111
> 从github下载R7-OCM库
        git clone https://github.com/RP7/R7-OCM.git
> 在往ARM上传文件之前，先用Vivado 生成FPGA的配置文件R7OCM_top.bit
>> 在PC段的终端中，将R7-OCM/scripts/post-commit文件复制到R-OCM/.git/hooks/目录下,并运行此文件（是一个sh文件）。
>> 打开Vivado,在Tcl Console 窗口中输入命令，先切换到R7-OCM目录下，执行
        source ./scripts/R7OCM.tcl
>> 等待工程生成完毕后，运行generate bitstream，生成R7OCM_top.bit文件
>> 在PC的命令窗口中，切换到R7-OCM目录下，运行
        sh ./scripts/upload.sh 10.0.77.111
* 注意在命令后面要跟ARM的IP地址作为参数
* upload.sh 脚本中主要完成文件压缩（PC端）、文件上传、文件解压（ARM端），FPGA配置（R7OCM_top.bit），并且在ARM端运行axi2s_c.py 和 q7web.py两个程序

# 在PC端接收并查看数据 及 通过curl对R7各参数进行配置

## PC端接收并查看数据

> 进入PC端命令终端，切换到R7-OCM目录下，运行
        sh ./scripts/curlinit.sh

> 打开浏览器，输入10.0.77.111：8080，即可打开数据查看界面。

## 通过curl对R7进行配置
* 通过curl控制R7的方法详见R7-OCM/RESTful.md
