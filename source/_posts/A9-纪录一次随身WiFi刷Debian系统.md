---

title: 记录一次随身WiFi刷Debian系统
date: 2023-02-23 16:16:55
updated: 
top: false
categories: 学习资料
description: 高通骁龙410的随身WiFi，刷入Debian之后可以变成小型服务器，装Docker和青龙面板拿来跑脚本，搭建下载站和博客等等功能简直是美滋滋，另外还可以内网穿透让外部网络访问。

---

## 刷入Debian系统
### 9008模式读取配置
1.使用的工具是Miko_Pro
<img src="https://sky.x-gap.ml/d/Config/A9-1.png"/>

2.很遗憾抽到的是4G的机子，也不是不能用就不退了
<img src="https://sky.x-gap.ml/d/Config/A9-2.png"/>

### 备份原系统
3.如下图备份一个后缀名为bin的全量包，方便日后回安卓
<img src="https://sky.x-gap.ml/d/Config/A9-3.png"/>

4.回安卓方法
<img src="https://sky.x-gap.ml/d/Config/A9-4.jpg"/>

### 备份基带分区文件
1.用9008Tool备份基带文件(不要插卡），先扫描出分区再备份，有以下几个文件fsc、fsg、modemst1、modemst2、modem（备份后为NON-HLOS.bin）
<img src="https://sky.x-gap.ml/d/Config/A9-5.png"/>
2.分别重命名为fsc.bin、fsg.bin、modemst1.bin、modemst2.bin、NON-HLOS.img备用
<img src="https://sky.x-gap.ml/d/Config/A9-6.png"/>
3.顺便备份boot分区，重命名为boot.img，方便root

### 备份QCN基带（高通串号）
1.将随身WiFi恢复出厂设置开启adb，顺便切卡看看能不能用自己卡上网，每个棒子切卡密码不一样，我的是UFIadmin1234
<img src="https://sky.x-gap.ml/d/Config/A9-7.jpg"/>
2.这一步需要获取root权限，自己装个面具修补一下boot即可，开端口时在投屏上允许shell获取root权限
<img src="https://sky.x-gap.ml/d/Config/A9-8.jpg"/>
<img src="https://sky.x-gap.ml/d/Config/A9-9.jpg"/>

### 拆开棒子看板号
1.我的随身WiFi型号是ufi003，寻找对应刷机资源
<img src="https://sky.x-gap.ml/d/Config/A9-10.jpg"/>

### 刷入Debian
1.安卓下重启到fastboot，将备份的基带fsc.bin、fsg.bin、modemst1.bin、modemst2.bin替换掉刷Debian系统文件夹里的文件
<img src="https://sky.x-gap.ml/d/Config/A9-11.png"/>

2.双击一键刷入，按提示刷入即可
<img src="https://sky.x-gap.ml/d/Config/A9-12.png"/>

3.刷入成功后重启，看Windows托盘会变为HandsomeMod
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

### 恢复剩余基带文件
1.使用finalshell连接随身WiFi，不同的包参数不一样，自己看作者介绍
主机：10.42.0.1
用户名：root
密码：1313144

2.电脑安装DiskGenius
依次点击：磁盘-打开虚拟磁盘文件-NON-HLOS.img-主分区，右键，把image文件夹复制到桌面，进入Debian系统，将image文件夹里面的文件，复制到debian的/usr/lib/firmware/文件夹里
<img src="https://sky.x-gap.ml/d/Config/A9-19.png"/>
<img src="https://sky.x-gap.ml/d/Config/A9-20.jpg"/>

3.插入SIM卡重启，等待两分钟左右看随身WiFi是否可以提供网络给电脑，可以的话说明已经可以使用SIM卡上网了

4.若还是不读卡（即没有网络），可执行一下命令

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

5.用卡会很热，需要改散热，所以还是建议连接WiFi使用
输入命令
```
nmtui
```
用键盘选择编辑链接，把wifi删除（即关掉随身WiFi的热点），ESC回到上级启用连接，连接自家WiFi，WiFi最好没有中文，否则可能扫描不到

## 安装Docker
### 更换Debian软件源
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

### 安装docker及管理面板
1.切换root：
```
sudo -i
```

2.更新源：
```
sudo apt-get update
```

3.安装工具：
```
sudo apt-get install curl wget apt-transport-https  ca-certificates gnupg2 software-properties-common
```

4.添加 Docker 的官方 GPG 密钥：
```
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/debian/gpg | sudo apt-key add -
```

5.先卸载残留：
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

6.自动安装docker：
```
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

7.docker切换为国内源：

```
sudo echo '{"registry-mirrors": ["https://alzgoonw.mirror.aliyuncs.com"]}' > /etc/docker/daemon.json
```

8.查看下是否添加成功，成功会有一行源的信息
```
cat /etc/docker/daemon.json
```

9.更新下配置并且重启：
```
systemctl restart docker && systemctl status docker && reboot
```

10.重启后再次连接ssh，输入：`sudo docker version`，查看当前的docker安装状态

11.安装面板：
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

12.安装好之后，访问地址进入系统：
http://服务器IP地址或域名:8081

13.初始登录名和密码为:
登录名：admin
密码：888888/123456

## Docker安装青龙面板
1.一键执行以下命令
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

2.登录青龙面板
登录地址：http://你的ip地址:5700

3.安装依赖
建议不要自动安装依赖，容易卡死棒子，手动安装需要的依赖即可

## Docker搭建Alist
Alist发行版本稳定版，端口5244
```
docker run -d --restart=always -v /etc/alist:/opt/alist/data -p 5244:5244 -e PUID=0 -e PGID=0 -e UMASK=022 --name="alist" xhofe/alist:latest
```

## 内网穿透
1.下载程序
```
wget https://git.histb.com/cloudflare/cloudflared/releases/download/2022.12.1/cloudflared-linux-arm -O /usr/bin/cloudflared
```
2.登录 Cloudflared
```
cloudflared tunnel login
```
3.这时会弹出来一个URL，用浏览器打开，登录成功后关闭浏览器，再次打开URL，这时候会出现授权   页面，然后选择你想用来做内网穿透的域名授权即可

4.建立隧道
```
cloudflared tunnel create <隧道名称>
```

举例：
```
cloudflared tunnel create skylist #skylist就是隧道名
```

成功后会提示，同时得到一个隧道ID，复制下来备用

如：2e5c689a-****-****-****-94786210ff72

相关凭证已放置于~/.cloudflared/<Tunnel-UUID>.json中

5.新建 Tunnel 对应的 DNS 记录
```
cloudflared tunnel route dns <隧道名称> <域名>
```
举例：
```
cloudflared tunnel route dns skylist sky.sauh.com
```

6.写配置文件
新建配置文件（i编辑，ESC退出编辑，:q退出，:wq!保存并强制退出）
```
vim config.yml
```
新建配置文件的存放文件夹
```
mkdir -p /etc/cloudflared/
```
把配置文件移动到该文件夹内
```
cp config.yml /etc/cloudflared/
```
编辑配置文件并保存
```
tunnel: <隧道ID>
credentials-file: /root/.cloudflared/<隧道ID>.json
protocol: http2
originRequest:
 connectTimeout: 30s
 noTLSVerify: false
ingress:
  - hostname: <域名>
    service: http://localhost:<端口号>
  - service: http_status:404
```
举例：
```
tunnel: 2e5c689a-****-****-****-94786210ff72
credentials-file: /root/.cloudflared/2e5c689a-****-****-****-94786210ff72.json
protocol: http2
originRequest:
 connectTimeout: 30s
 noTLSVerify: false
ingress:
  - hostname: sky.sauh.com
    service: http://10.42.0.1:5244
  - service: http_status:404
```
7.安装服务
```
cloudflared service install
```
8.设置自启动
```
systemctl start cloudflared
```
9.现在任意网络访问sky.sauh.com就可以访问到我们搭建在本地5244端口的Alist了

10.实现不同二级域名对应不同端口的内网穿透
配置文件这么写
```
tunnel: 2e5c689a-****-****-****-94786210ff72
credentials-file: /root/.cloudflared/2e5c689a-****-****-****-94786210ff72.json
protocol: http2
originRequest:
 connectTimeout: 30s
 noTLSVerify: false
ingress:

  - hostname: sky.sauh.com
    service: http://10.42.0.1:5244

 - hostname: qinglong.sauh.com
    service: http://10.42.0.1:5700

  - service: http_status:404
```
在同一隧道记录新的dns
```
cloudflared tunnel route dns skylist qinglong.sauh.com
```
这样就实现了不同二级域名访问不同端口

## 随身WiFi清理
折腾到这基本结束了，还能再搭建个博客，以后弄了再更新教程，系统内有一些缓存垃圾，使用如下命令删除
```
apt-get clean
journalctl --vacuum-size=5M
echo > /var/log/syslog
echo > /var/log/syslog.1
echo > /var/log/mail.log.1
echo > /var/log/mail.info.1
echo > /var/log/mail.warn.1
echo > /var/log/mail.err.1
echo > /var/log/mail.log
echo > /var/log/mail.info
echo > /var/log/mail.warn
echo > /var/log/mail.err
```
可以制作定时任务定期清理

## 资源下载及说明
[点击查看](https://sky.x-gap.ml/安卓资源盘/随身WiFi)