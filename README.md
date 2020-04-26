# 1.硬件系统的搭建
## 树莓派参数

先来看一下树莓派和当前主流的入门级别NAS的对比

<img src="/pic/DS218j.jpg" width="20%" title="DS218j"><img src="/pic/TS-428.jpg" width="26%" title="TS-428"><img src="/pic/rasp4.jpg" width="26%" title="树莓派4B">



   设备  | CPU  | 内存  | 盘位  | 接口  | 参考价格 
  :------: | :------: | :------: |:------: |:------: | :------: 
 群晖DS218j  | Armada 385 双核1.3Ghz  | 512MB DDR3  | 2  | 千兆网口  | 1300 
 威联通TS-428  | Cortex A53 4核心1.4GHz  | 1G  | 4  | 千兆网口x2支持链路聚合  | 1800 
 树莓派3B  | Cortex-A53 4核心1.4GHz  | 1G LPDDR2  | 无  | 千兆网口overUSB2.0  | 270左右 
 树莓派4B  | Cortex A72 4核心1.5Ghz  | 1-4G可选LPDDR4  | 无  | 千兆网口  | 300左右 

### 课程制作的NAS速度预测
课程使用的是树莓派4B 配备了USB3.0(USB3.1 Gen1)带宽为5Gbps.
假设USB硬盘HUB的速度损耗不计算。采用的西部数据1T单盘和某杂牌120G固态作为NAS盘,连续读写速度均远高于1000Mbps的带宽。
那么NAS的读取速度应该在125MB/s（未考虑损耗）

实际使用过程中，ext4文件系统的固态硬盘和机械硬盘读写速度都在100MB/s左右符合预计。（注，如果是使用windows下的NTFS格式读取速度不受影响，而写入会降低到30MB/s左右）

<img src="/pic/1.png" width="30%" title="机械硬盘"><img src="/pic/2.png" width="30%" title="固态硬盘">

## USB硬盘HUB的选择


<img src="/pic/CM249.jpg" width="30%" title="绿联CM249"><img src="/pic/CM198.jpg" width="30%" title="绿联CM198">

上面两个设备由左到右分别为绿联CM249,绿联CM198。课程选择后者（不带脱机拷贝功能）。


 设备 | 盘位 |单盘最大容量|传输带宽| 特别功能 |参考价格
:------:|:------:|:------:|:------:|:------:|:------:
 CM249 | 2 | 16T |5Gbps| 硬件RAID0/1 |569
 CM198 | 2 | 12T |5Gbps| 可选脱机拷贝 |149

设备的推荐原因
>CM249,由于OMV不支持USB接入硬盘的RAID设置。所以如果有RAID0/1的需求的用户可以选择这款设备

>CM198,便于摆放，价格合适。

如果大家需要购买其他型号的USB硬盘HUB，购买时参考一下单盘最大容量，盘位，以及传输带宽。由于USB3.0 改名为USB3.1 Gen1。需要与USB3.1 Gen2区分开了，后者有10Gbps的带宽。另外，尽量购买带外接电源的设备，树莓派的USB供电能力很弱。

## 交换机

交换机往往是NAS系统中容易忽视的一环。一般连接NAS的方式，是电脑和NAS都连接到路由器中。如果路由器的lan口是千兆的话，那么速度上面就没有什么问题。如果路由器不是千兆LAN口，那么我们就需要加入一个千兆的交换机。

<img src="/pic/switch.jpg" width="30%" title="switch">

千兆交换机的选择可以随意。试验环境中使用的是水星的5口千兆交换机，价格为45元。

## 连接线材与其他

网线选择六类线即可。树莓派使用SD卡class10 16G以上。

## 视频中设备价格

树莓派4B-2G 289+绿联CM198 128+水星交换机 45+ 内存卡16G 24=486元（不含网线和硬盘）


------------------------------


# 2.树莓派安装OMV
## 在SD卡中烧录系统（演示包括下载网站，烧录工具）
### 下载镜像和烧录工具

镜像地址: https://www.raspberrypi.org/downloads/raspbian/  
烧录工具地址： https://www.balena.io/etcher/

### 烧录镜像

在SD卡中创建wpa_supplicant.conf(内容如下)来连接wifi（不推荐，建议使用有线连接），另外创建空的名为SSH的文件来开启SSH。

	country=CN
	ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
	update_config=1
	network={
	ssid="连接的wifi名称"
	psk="wifi的密码"
	key_mgmt=WPA-PSK
	priority=1
	}

## 通过putty来连接树莓派,并开始安装OMV

>初始用户名为pi，密码为raspberry

### 修改树莓派apt的源地址，添加OMV的源地址


>A.修改源地址

>>     sudo nano /etc/apt/sources.list

>>将原有地址注释掉，同时添加下面的地址


>>     deb http://mirrors.aliyun.com/raspbian/raspbian/ buster main contrib non-free rpi
>>     deb-src http://mirrors.aliyun.com/raspbian/raspbian/ buster main contrib non-free rpi

>B.添加OMV地址

>>     sudo nano /etc/apt/sources.list.d/openmediavault.list

>>将下列内容写入文件中

>>     deb https://packages.openmediavault.org/public usul main
>>     deb https://downloads.sourceforge.net/project/openmediavault/packages usul main
>>     ##Uncomment the following line to add software from the proposed repository.
>>     deb https://packages.openmediavault.org/public usul-proposed main
>>     #deb https://downloads.sourceforge.net/project/openmediavault/packages usul-proposed main
>>     ##This software is not part of OpenMediaVault, but is offered by third-party
>>     ##developers as a service to OpenMediaVault users.
>>     #deb https://packages.openmediavault.org/public usul partner
>>     #deb https://downloads.sourceforge.net/project/openmediavault/packages usul partner

>最后更新一下apt

>>     sudo apt-get update

### 创建安装脚本、下载安装OMV


>A.在~目录下，创建OMV文件

>>     sudo nano OMV

>将下面的内容加入到其中

>>     export LANG=C.UTF-8
>>     export DEBIAN_FRONTEND=noninteractive
>>     export APT_LISTCHANGES_FRONTEND=none
>>     wget -O "/etc/apt/trusted.gpg.d/openmediavault-archive-keyring.asc" 	https://packages.openmediavault.org/public/archive.key
>>      apt-key add "/etc/apt/trusted.gpg.d/openmediavault-archive-keyring.asc"
>>      apt-get update
>>      apt-get --yes --auto-remove --show-upgraded \
>>      		--allow-downgrades --allow-change-held-packages \
>>      		--no-install-recommends \
>>      		--option Dpkg::Options::="--force-confdef" \
>>      		--option DPkg::Options::="--force-confold" \
>>      		install openmediavault-keyring openmediavault

>文件编辑完毕后，输入<u>sudo sh OMV</u>运行，等待下载安装完毕。   
## 至此OMV的安装已经完成了，浏览器输入树莓派IP进入UI界面

openmediavault初始用户名密码为admin、openmediavault。

-----------------------------


# 3.OMV的基本配置
 > 在配置OMV的过程中如果webUI上方弹出了下面的黄色标语。可以在所有的内容都修改完毕后点击应用。
 > <img src="E:/pic/5.png" width="60%" title="biaoyu">
## 挂载磁盘

在第一次使用OMV的时候，需要挂载磁盘并创建文件系统
 > 具体操作如下
 
 > 1.连接USB硬盘HUB的电源与USB线
 
 > 2.进入到OMV的webUI，在[储存器]目录，硬盘菜单中，查看硬盘是否连接上来。
 
 > 3.进入到文件系统中。如果没有设备则创建。如果有，则选择设备点击挂载。（一般全新硬盘是没有文件系统的，而如果是使用过的硬盘，WIN下是NTFS的文件系统，也可直接挂载，但是对于缺点不再阐述，这里的设备相当于windows下的盘的概念，一块硬盘可以分为多个设备（盘）。）
 
注意:***在硬盘菜单中，选择对应的设备，可以对其设置电源管理，以及写入缓存的启用与否。建议机械硬盘启用写入缓存，固态硬盘不启用。
 ><img src="/pic/3.png" width="60%" title="WEBUI">

## 用户创建与权限修改

前面说到的OMV初始密码指WEBUI的管理登陆密码，登陆上去以后我们还需要进一步的管理用户。

Pi用户依旧存在，但是它的ssh权限被关闭了，我们需要打开这一个权限，才能继续使用。另外我们可以创建一个专门用来文件传输的用户。操作方式如下:
 > 1.在WEBUI中进入到[访问权限管理]目录下，进入到用户菜单中
 
 > 2.点击添加，输入用户名和密码，确定。
 
 > 3.这样的话就创建了一个用户。默认属于的用户组为users。可以实现例如samba、ftp等服务的使用。
 
 > 4.还是在用户菜单下，选择pi用户进行编辑，将用户组ssh勾选上，这样就实现了SSH权限的分配。
 

## 创建共享文件夹

共享文件夹就是最终用户直接接触到的内容。下图中就是两个共享文件夹。
 ><img src="/pic/4.png" width="60%" title="SAMBA">
共享文件夹的创建方式如下:
 > 1.在WEBUI中进入到[访问权限管理]目录下，进入到共享文件夹菜单中
 
 > 2.点击添加，输入文件夹名称和相对路径(不填写会根据名称自动生成)，选择文件夹所在的设备（即前面提到的文件系统，如果每个硬盘只有一个文件系统（推荐这样设置），那么可以认为是选择文件夹所在的硬盘。）
 
 >3.另外创建完毕以后，可以选择对应的文件夹，进行用户的读写权限编辑，可设置可读写、只读、禁止读写。（例如设置共享文件夹只可被A用户读写，B用户无法读写）
 
## Samba服务

Samba服务对于windows和MAC用户来说都可以使用，一些手机软件也可以直接访问。开启方式:
 > 1.在服务目录下，SMB/CIFS菜单中，将 {启用} 项， {可浏览} 项都调整为开启状态。调整完毕，点击保存。
 
 > 2.点击 该菜单下，共享页签。点击添加，选择需要使用samba服务的共享文件夹。点击保存即可。
 
## 至此OMV的基础配置已经完成了，可以进行文件的传输了


 



 

