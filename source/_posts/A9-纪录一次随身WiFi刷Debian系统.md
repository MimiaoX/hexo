---

title: A9-纪录一次随身WiFi刷Debian系统
date: 2022-11-28 16:16:55
updated: 
top: false
categories: 学习资料
description: 高通骁龙410的随身WiFi，刷入Debian之后可以变成小型服务器，拿来跑脚本美滋滋。

---

## 9008模式读取配置并备份原系统
1. 使用的工具是Miko_Pro
<img src="https://sky.x-gap.ml/d/Config/A9-1.png"/>

2. 很遗憾抽到的是4G的机子，也不是不能用就不退了
<img src="https://sky.x-gap.ml/d/Config/A9-2.png"/>

3. 如下图备份一个后缀名为bin的全量包，方便日后回安卓
<img src="https://sky.x-gap.ml/d/Config/A9-3.png"/>

4. 回安卓方法
<img src="https://sky.x-gap.ml/d/Config/A9-4.jpg"/>

## 9008模式备份基带文件
1. 用9008Tool备份基带文件(不要插卡），先扫描出分区再备份，有以下几个文件
fsc
fsg
modemst1
modemst2
modem（备份后为NON-HLOS.bin）
<img src="https://sky.x-gap.ml/d/Config/A9-5.png"/>
2. 分别重命名为备用
fsc.bin
fsg.bin
modemst1.bin
modemst2.bin
NON-HLOS.img
<img src="https://sky.x-gap.ml/d/Config/A9-6.png"/>

## 备份QCN
1. 将随身WiFi恢复出厂设置开启adb，顺便切卡看看能不能用自己卡上网（切卡密码UFIadmin1234）
<img src="https://sky.x-gap.ml/d/Config/A9-7.jpg"/>
2. 这一步需要获取root权限，自己装个面具修补一下boot即可，开端口时在投屏上允许shell获取root
<img src="https://sky.x-gap.ml/d/Config/A9-8.jpg"/>
<img src="https://sky.x-gap.ml/d/Config/A9-9.jpg"/>

## 拆开棒子看板号找相关资源
1. 可见我的随身WiFi型号是ufi003，寻找相关刷机资源
<img src="https://sky.x-gap.ml/d/Config/A9-10.jpg"/>

## 刷入Debian
1. 安卓下重启到fastboot，将备份的基带fsc.bin、fsg.bin、modemst1.bin、modemst2.bin替换掉Debian文件夹里的文件
<img src="https://sky.x-gap.ml/d/Config/A9-11.png"/>

2. 双击一键刷入，按提示刷入即可
<img src="https://sky.x-gap.ml/d/Config/A9-12.png"/>

3. 刷入成功后重启，看Windows托盘会变为HandsomeMod
<img src="https://sky.x-gap.ml/d/Config/A9-13.png"/>

4.设备管理器里应有 基于远程的NDIS的Internet的共享设备
<img src="https://sky.x-gap.ml/d/Config/A9-14.png"/>

没有的话在其它设备-未知设备里找到它并更新为基于远程的NDIS的Internet的共享设备
<img src="https://sky.x-gap.ml/d/Config/A9-15.png"/>
<img src="https://sky.x-gap.ml/d/Config/A9-15.jpg"/>
<img src="https://sky.x-gap.ml/d/Config/A9-16.jpg"/>
<img src="https://sky.x-gap.ml/d/Config/A9-17.jpg"/>
<img src="https://sky.x-gap.ml/d/Config/A9-18.jpg"/>
未知设备里也没有就把Android Device驱动更新成Composite USB Device驱动就会出现未知设备了，再重复上述步骤

## ssh连接
1. 使用finalshell连接随身WiFi
主机：10.42.0.1
用户名：root
密码：1313144

2. 电脑安装DiskGenius
依次点击：磁盘-打开虚拟磁盘文件-NON-HLOS.img-主分区，右键，把image文件夹复制到桌面。
进入debian系统，将image文件夹里面的文件，复制到debian的/usr/lib/firmware/文件夹里，重启
<img src="https://sky.x-gap.ml/d/Config/A9-19.png"/>
<img src="https://sky.x-gap.ml/d/Config/A9-20.jpg"/>

3. 若还是不读卡，可执行一下命令

```
sleep 3
systemctl stop ModemManager
sleep 3
qmicli -d /dev/wwan0qmi0 --uim-sim-power-off=1
sleep 3
qmicli -d /dev/wwan0qmi0 --uim-sim-power-on=1
sleep 3
systemctl start ModemManager
sleep 3
```
若成功联网，则把上述命令放入/etc/rc.local中让其自启
<img src="https://sky.x-gap.ml/d/Config/A9-21.png"/>

4. 用卡会很热，需要改散热，还是建议连接WiFi使用
输入命令
```
nmtui
```
用键盘选择编辑链接，把wifi删除，esc回到上级启用连接，连接自家WiFi，WiFi不能有中文，否则扫描不到

## 更换Debian软件源
先安装vim编辑器
```
sudo apt-get install vim
```
再一键执行以下命令
```
echo -e '\n\n\n\n\n\n\n\n\n\n####################################\n'
sudo rm /etc/apt/sources.list
sudo touch /etc/apt/sources.list
sudo echo -e "# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释\n\ndeb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free\n# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free\n\ndeb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free\n# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free\n\ndeb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free\n# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free\n\ndeb https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free\n# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security bullseye-security main contrib non-free" >> /etc/apt/sources.list
echo -e '1、默认软件源修改完成！\n\n'
sudo sed -i '1c deb http://mirrors.tuna.tsinghua.edu.cn/Adoptium/deb buster main' /etc/apt/sources.list.d/AdoptOpenJDK.list
gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 843C48A565F8F04B
sudo gpg --armor --export 843C48A565F8F04B | sudo apt-key add -
echo -e '\n\n2、AdoptOpenJDK报错修复完成！\n\n'
sudo sed -i '1c #deb http://repo.mobian-project.org/ bullseye main non-free'  /etc/apt/sources.list.d/mobian.list
echo -e '3、Mobian源报错已屏蔽！'
echo -e '\n\n####################################\n\n即将开始更新软件源list......\n'
sleep 2
sudo apt-get update
echo -e '\n\n4、更新软件源list更新完成！'
echo -e '\n\n####################################\n\n即将开始升级系统程序至最新版......'
sleep 2
sudo apt-mark hold openssh-server
sudo apt-get -y upgrade
sudo apt-mark unhold openssh-server
echo -e '\n\n5、系统程序更新完成！\n\n####################################\n\n\n\n'
```

## 安装docker及管理面板
1. 切换root：
```
sudo -i
```

2. 更新源：
```
sudo apt-get update
```

3. 安装工具：
```
sudo apt-get install curl wget apt-transport-https  ca-certificates gnupg2 software-properties-common
```

4. 添加 Docker 的官方 GPG 密钥：
```
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/debian/gpg | sudo apt-key add -
```

5. 先卸载残留：
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

6. 自动安装docker：
```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

7. docker切换为国内源：

```
sudo echo '{"registry-mirrors": ["https://alzgoonw.mirror.aliyuncs.com"]}' > /etc/docker/daemon.json
```

8. 查看下是否添加成功，成功会有一行源的信息
```
cat /etc/docker/daemon.json
```

9. 更新下配置并且重启：
```
systemctl restart docker && systemctl status docker && reboot
```

10. 重启后再次连接ssh，输入：sudo docker version，查看当前的docker安装状态

11. 安装面板：
```shell
sudo docker run -d \
--name FAST_OS_DOCKER \
--restart always \
-p 8081:8081 \
-p 8082:8082 \
-e TZ="Asia/Shanghai" \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /etc/docker/:/etc/docker/ \
wangbinxingkong/fast:latest
```

12. 安装好之后，访问地址进入系统：
http://服务器IP地址或域名:8081

13. 初始登录名和密码为:
登录名：admin
密码：888888/123456

## 安装青龙面板
1. 一键执行以下命令
```shell
mkdir /Docker
docker run -dit \
--name QingLong \
--network=host \
--restart always \
-v /Docker/QingLong/config:/ql/config \
-v /Docker/QingLong/log:/ql/log \
-v /Docker/QingLong/db:/ql/db \
-v /Docker/QingLong/scripts:/ql/scripts \
-v /Docker/QingLong/jbot:/ql/jbot \
-e ENABLE_HANCUP=true \
-e ENABLE_WEB_PANEL=true \
 --label com.centurylinklabs.watchtower.enable=false \
 whyour/qinglong:latest
```

2. 登录青龙面板
登录地址：http://你的ip地址:5700

3. 安装依赖
进入docker面板 FAST OS DOCKER，在容器中找到QingLong，点击控制台并连接
执行：
```shell
curl -fsSL https://ghproxy.com/https://raw.githubusercontent.com/bean661/utils/main/QLOneKeyDependency.sh | sh
```
自动安装时间比较长，耐心等待依赖安装完成