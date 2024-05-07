# Steps-for-collecting-csi-data-of-WiFi-using-Raspberry-PI-4B
The Raspberry PI 4B installs the nexmon patch and nexmon_csi and captures the csi data steps for a specific WiFi
## 一、安装nexmon补丁
### (一)安装前树莓派预备事项
1. 硬件准备：键盘、鼠标、充电器、显示器&特殊hdmi线、一个赤子之心  
2. 系统务必开启SSH远程连接服务，以使后续系统、软件包、git库等下载操作快速完成。（在树莓派上下载速度极慢）  
3. 本教程树莓派系统务必更新到内核5.10.103版本，烧录完系统第一次开机启动时进行。若第一次没有更新，则在命令行中输入代码：`sudo apt update`, `sudo apt full-upgrade`, `sudo reboot`。输入`uname -r`查看系统版本。
4. 务必在进行安装前在命令行中输入df -h查看空间使用情况（nexmon固件补丁大约占2~3G内存）。若不够，参照以下博客扩容：[树莓派扩展root分区_树莓派扩展分区-CSDN博客](树莓派扩展root分区_树莓派扩展分区-CSDN博客)
### (二)安装nexmon补丁步骤
参考nexmon的github官方网址：[seemoo-lab/nexmon_csi：各种 Broadcom Wi-Fi 芯片上的信道状态信息提取 (github.com)](https://github.com/seemoo-lab/nexmon?tab=readme-ov-file#build-patches-for-bcm43430a1-on-the-rpi3zero-w-or-bcm434355c0-on-the-rpi3rpi4-or-bcm43436b0-on-the-rpi-zero-2w-using-raspbianraspberry-pi-os-recommended)
备注：  
* 树莓派4B的网卡芯片为BCM43455C0。
* 执行完第6、7步后再次检查会出现报错：“段错误”。具体产生原因不详，可能由野指针、空指针导致，可忽略。
  ![image](https://github.com/Fu0804/Steps-for-collecting-csi-data-of-WiFi-using-Raspberry-PI-4B/assets/151499353/b30aa05c-8b2b-4536-9be5-b8acef647844)
  ![image](https://github.com/Fu0804/Steps-for-collecting-csi-data-of-WiFi-using-Raspberry-PI-4B/assets/151499353/d2fef81a-f2ca-4546-b7e5-42c50e2c4d6d)
* 第12步无需执行。  
  ![image](https://github.com/Fu0804/Steps-for-collecting-csi-data-of-WiFi-using-Raspberry-PI-4B/assets/151499353/ac0a6516-9f8d-432b-b7e1-9eea0e5ef9cf)
* 注意第十步执行在`/home/pi/nexmon/patches/bcm43455c0/7_45_189/nexmon_csi`目录下。
  ![image](https://github.com/Fu0804/Steps-for-collecting-csi-data-of-WiFi-using-Raspberry-PI-4B/assets/151499353/314f1448-5a07-4071-99d5-486ef35d459c)
### (三)检验是否安装良好可以使树莓派网卡进入监听模式
1. 输入`iw dev`查看输出是否有非phy0其他网络名称。若有则初步证明安装良好。
2. 输入`iw phy1 info`（phy1为你刚刚得到的网络名称），查看wlan0是否有monitor模式。若有则安装成功。
## 二、安装nexmon_csi并实现csi的捕获与采集
### (一)下载makecsiparams工具
执行以下命令：`curl -fsSL https://raw.githubusercontent.com/nexmonster/nexmon_csi_bin/main/install.sh | sed '39,43d' | sudo bash`  
**注意：此命令会禁用并删除wpa_supplicant，导致树莓派网卡无法连接WiFi，从而无法联网。**
### (二)查找想要捕获的wifi的MAC地址
1. 首先输入`ifconfig wlan0 up`
2. 输入`iwlist wlan0 scan`
3. 在输出结果中查找复制想要捕获WiFi的MAC地址，形如aa:bb:aa:bb:aa:bb，并记住其频道。
### (三)采集工作
参考网址：[seemoo-lab/nexmon_csi：各种 Broadcom Wi-Fi 芯片上的信道状态信息提取 (github.com)](https://github.com/seemoo-lab/nexmon)
1. 输入`mcp -C 1 -N 1 -c 36/80`（其中36/80分别为你要捕获的WiFi的频道和想要采集的带宽）。输出形如：`m+IBEQGIAgAAESIzRFWqu6q7qrsAAAAAAAAAAAAAAAAAAA==`，复制。
2. 确保接口已启动：`ifconfig wlan0 up`
3. 使用 nexutil 和生成的参数配置提取器（使用参数调整 -v 的参数）：`nexutil -Iwlan0 -s500 -b -l34 -vm+IBEQGIAgAAESIzRFWqu6q7qrsAAAAAAAAAAAAAAAAAAA==`
4. 启用监控模式：``iw phy `iw dev wlan0 info | gawk '/wiphy/ {printf "phy" $2}'` interface add mon0 type monitor``
5. `ifconfig mon0 up`  
   ![image](https://github.com/Fu0804/Steps-for-collecting-csi-data-of-WiFi-using-Raspberry-PI-4B/assets/151499353/b69fd713-44f8-44d2-811a-00a95d1e07ce)
7. 采集CSI数据：`tcpdump -i wlan0 dst port 5500 -vv -w test.pcap -c 60`
## 三、每次开机后务必注意事项
### 步骤
1. `cd /home/pi/nexmon`
2. `source setup_env.sh`
3. `cd /home/pi/nexmon/patches/bcm43455c0/7_45_189/nexmon_csi`
4. `make install-firmware`
  
**如果你已经采集了一种WiFi的csi并想采集其他不同的WiFi务必在切换WiFi前执行上述操作操作**


