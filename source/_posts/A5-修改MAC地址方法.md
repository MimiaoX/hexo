---

title: 修改MAC地址方法
date: 2022-09-06 16:16:55
updated: 2022-09-06 16:16:55
top: false
categories: 技巧攻略
description: 由于某校的校园网存在一个账号只能一台手机和一台电脑同时在线的限制，出于好奇想将手机伪装成电脑，最终发现校园网是通过识别MAC地址来作为识别是什么设备的。修改方法来自酷安@OLX_盐_NaCl，方法仅在高通855平台的MIUI12和MIUI12.5测试通过。

---

### 起因
&emsp;&emsp;由于本校校园网的限制是最多同时支持一台手机和一台电脑上网，于是我便猜想是通过IP地址或MAC地址来识别设备的。大多数情况下我们随身携带手机，但并非会携带电脑，这样就会有闲置资源可以利用，即不使用电脑时可供另一台手机或平板电脑上网。总而言之将“一台手机和一台电脑”的限制变为“两台设备”。
&emsp;&emsp;当限制变为“两台设备”后，除闲置资源利用外，还可以解锁更多玩法，如这个设备可以是路由器或智能音箱等等，当然路由器就可以使更多的设备上网（但总带宽不变）。或是你设备被校园网拉黑后可以更换一个MAC地址。

### 什么是MAC地址
起初我猜想是不是通过IP地址限制了一台电脑和一台手机的使用，但是很遗憾并不是，IP地址并不能识别设备。
<img src="https://https://sky.x-gap.ml/d/Config/IPpng" width="80%"/>
而这个物理地址（MAC地址）才是重点。
<img src="https://https://sky.x-gap.ml/d/Config/MAC.png" width="80%"/>

### 如何修改MAC地址
用【MT管理器】 打开/mnt/vendor/persist 打开/mnt/vendor/persist，在这里新建一个文件夹名字为wlan，如图
<img src="https://sky.x-gap.ml/d/Config/A5-1.jpg" width="80%"/>
然后点进去，新建一个文件叫wlan_mac.bin
<img src="https://sky.x-gap.ml/d/Config/A5-1.jpg" width="80%"/>
接下来我们用编辑文本方式打开它，输入wlan0=你的MAC地址，例如wlan0=6A518AC70A1C，用字母和数字的组合，MAC地址一般是12个字符
<img src="https://sky.x-gap.ml/d/Config/A5-1.jpg" width="80%"/>
然后返回保存，重启手机，你的MAC地址就修改/恢复成功了，打开WLAN，去设置-我的设备-全部参数-状态信息中找到设备WLAN MAC地址看看是否修改成功
<img src="https://sky.x-gap.ml/d/Config/A5-1.jpg" width="80%"/>
在MIUI12（Android10）中，不用新建wlan文件夹，直接在/mnt/vendor/persist中进行上述步骤即可，若还是不成功可以将MAC写为两行，如图
<img src="https://sky.x-gap.ml/d/Config/A5-1.jpg" width="80%"/>

### 个人总结
&emsp;&emsp;根据以上操作之后本人MAC地址修改成功，并且可以模拟电脑上网，但是手机修改MAC地址难度高（主要是指Root门槛），不同手机还不尽相同，而且仅通过MAC地址是如何识别是手机和电脑的呢，手机型号写在System下的bulid.prop内，但实际上又是工具MAC地址来限制上网设备的。
&emsp;&emsp;所以在不能修改手机MAC的情况下，只是为了多一个手机/平板上网的话，我们可以将Windows的MAC修改成手机的，连上校园网上网之后再将电脑MAC恢复，那么你的手机将会被识别为电脑设备，在你不使用电脑时，就可以用这台手机（识别为电脑）和另一台手机/平板（识别为手机）上网了。
&emsp;&emsp;修改电脑MAC地址来让手机被识别为电脑的方法本人并没有尝试过是否可行，这只是我的一个猜想，毕竟修改Windows的MAC相对容易，百度上就有教程。
&emsp;&emsp;不过现在可以证明本校校园网（卓智）依靠的是识别MAC地址来限制设备上网的，将手机模拟成电脑（使用过校园网的电脑）MAC地址即可让校园网把手机识别为电脑。