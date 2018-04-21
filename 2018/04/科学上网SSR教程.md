本次教程有以下三步：

1. 购买VPS服务器；
2. 部署SSR；
3. 加速SSR ；
4. SSR客户端下载及使用。

## 一：购买VPS服务器

vps服务器优选国外的，首选国际知名的[Vultr](https://www.vultr.com/?ref=7243695)，速度不错、稳定且性价比高，按小时计费，能够随时开通和删除服务器，新服务器即是新IP。

注册并充值后即可购买服务器，充值方式现已支持支付宝，

![](https://subetter.com/images/figures/20180421_01.png)

Vultr实际上是折算成小时来计费的，比如服务器是5美元1个月，那么每小时收费为5/30/24=0.0069美元，会自动从账号中扣费，只要保证账号有钱即可。如果你部署的服务器实测后速度不理想，你可以把它删掉（destroy），重新换个地区的服务器来部署，方便且实用。因为新的服务器就是新的IP，所以当IP被墙时这个方法很有用。当IP被墙时，为了保证新开的服务器IP和原先的IP不一样，先开新服务器，开好后再删除旧服务器即可。

计费从你开通服务器开始算的，不管你有没有使用，即使服务器处于关机状态仍然会计费，如果你没有开通服务器就不算。比如你今天早上开通了服务器，但你有事情，晚上才部署，那么这段时间是会计费的。同理，如果你早上删掉服务器，第二天才开通新的服务器，那么这段时间是不会计费的。在账号的Billing选项里可以看到账户余额。

温馨提醒：同样的服务器位置，不同的宽带类型和地区所搭建的账号的翻墙速度会不同，这与中国电信、中国联通、中国移动国际出口带宽和线路不同有关，所以以实测为准。可以先选定一个服务器位置来按照教程进行搭建，熟悉搭建方法，当账号搭建完成并进行了bbr加速后，测试下速度自己是否满意，如果满意那就用这个服务器位置的服务器。如果速度不太满意，就一次性开几台不同的服务器位置的服务器，然后按照同样的方法来进行搭建并测试，选择最优的，之后把其它的服务器删掉，按小时计费测试成本可以忽略，就是这么任性。

开通服务器，

![](https://subetter.com/images/figures/20180421_02.png)

选择你的服务器位置，美国和日本的都行。

选择vps操作系统时，不要选centos7系统！点击图中的CentOS几个字，会弹出centos6，然后选中centos6！entos7默认的防火墙可能会干扰ssr的正常连接！

价格和流量选择上，如果只是查查资料，偶尔看下视频，那么2.5美元一个月（500G流量）的套餐已经足够了，不过这个套餐很紧俏，很容易卖光，你能否买到得看运气，实在不行的话，就选5美元一个月的，可以和朋友一起分摊费用。

然后选择Deploy Now，生成完毕后就可以看到自己的vps了，

![](https://subetter.com/images/figures/20180421_03.png)

## 二：部署SSR

购买服务器后，需要部署一下。因为你买的是虚拟东西，而且又远在国外，我们需要一个叫[Xshell](https://www.netsarang.com/download/down_form.html?code=522)的软件来远程部署，下载免费版，

![](https://subetter.com/images/figures/20180421_04.png)

下载安装好打开，选择文件，新建，

![](https://subetter.com/images/figures/20180421_05.png)

随便取个名字，然后把你的服务器ip填上，

![](https://subetter.com/images/figures/20180421_06.png)

连接国外ip即服务器时，软件会先后提醒你输入用户名和密码，用户名默认都是root，

![](https://subetter.com/images/figures/20180421_07.png)

密码是你购买的服务器系统的密码，

![](https://subetter.com/images/figures/20180421_08.png)

连接成功后，会出现如上图所示，之后就可以复制粘贴代码部署了。

![](https://subetter.com/images/figures/20180421_09.png)

连接成功后，会出现如上图所示，之后就可以复制粘贴代码部署了。CentOS6/Debian6/Ubuntu14 ShadowsocksR一键部署管理脚本：

```
yum -y install wget

wget -N --no-check-certificate https://softs.fun/Bash/ssr.sh && chmod +x ssr.sh && bash ssr.sh
```

上面代码不行，就用备用脚本：

```
yum -y install wget

wget -N --no-check-certificate https://raw.githubusercontent.com/ToyoDAdoubi/doubi/master/ssr.sh && chmod +x ssr.sh && bash ssr.sh
```

复制上面的代码到VPS服务器里，复制代码用鼠标右键的复制，然后在vps里面右键粘贴进去，因为ctrl+c和ctrl+v无效。接着按回车键，脚本会自动安装，以后只需要运行快捷命令就可以出现下图的界面进行设置，快捷管理命令为：`bash ssr.sh`。

![](https://subetter.com/images/figures/20180421_10.png)

如上图，出现管理界面后，输入数字1来安装SSR服务端。如果输入1后不能进入下一步，那么请退出Xshell，重新连接vps服务器，然后输入快捷管理命令bash ssr.sh再尝试。

![](https://subetter.com/images/figures/20180421_11.png)

根据上图提示，依次输入自己想设置的**端口和密码** (**密码建议用复杂点的字母组合，端口号为40-65535之间的数字**)，回车键用于确认

注：关于端口的设置，总的网络总端口有6万多个，理论上可以任意设置，但不要以0开头！但是有的地区需要设置特殊的端口才有效，一些特殊的端口比如80、143、443、1433、3306、3389、8080。

![](https://subetter.com/images/figures/20180421_12.png)

如上图，选择想设置的**加密方式**，比如10，按回车键确认

接下来是选择**协议插件**，如下图：

![](https://subetter.com/images/figures/20180421_13.png)

选择并确认后，会出现下面的界面，提示你是否选择兼容原版，这里的原版指的是SS客户端（SS客户端没有协议和混淆的选项），可以根据需求进行选择，演示选择y，

![](https://subetter.com/images/figures/20180421_14.PNG)

之后进行混淆插件的设置。

**注意：如果协议是origin，那么混淆也必须是plain；如果协议不是origin，那么混淆可以是任意的。有的地区需要把混淆设置成plain才好用。因为混淆不总是有效果，要看各地区的策略，有时候不混淆（plain）让其看起来像随机数据更好。（特别注意：tls 1.2_ticket_auth容易受到干扰！请选择除tls开头以外的其它混淆！！！）**

![](https://subetter.com/images/figures/20180421_15.png)

进行混淆插件的设置后，会依次提示你对设备数、单线程限速和端口总限速进行设置，默认值是不进行限制，个人使用的话，选择默认即可，即直接敲回车键。

注意：关于限制设备数，这个协议必须是非原版且不兼容原版才有效，也就是必须使用SSR协议的情况下，才有效！

![](https://subetter.com/images/figures/20180421_16.png)

之后代码就正式自动部署了，到下图所示的位置，提示你下载文件，输入：y，

![](https://subetter.com/images/figures/20180421_17.png)

耐心等待一会，出现下面的界面即部署完成，

![](https://subetter.com/images/figures/20180421_18.png)

接着出现，

![](https://subetter.com/images/figures/20180421_19.png)

根据上图就可以看到自己设置的SSR账号信息，包括IP、端口、密码、加密方式、协议插件、混淆插件，这些信息需要填入你的SSR客户端。如果之后想修改账号信息，直接输入快捷管理命令：bash ssr.sh 进入管理界面，选择相应的数字来进行一键修改。例如：

![](https://subetter.com/images/figures/20180421_20.png)

![](https://subetter.com/images/figures/20180421_21.png)

**脚本演示结束。**

此脚本是开机自动启动，部署一次即可。最后可以重启服务器确保部署生效（一般情况不重启也可以）。重启需要在命令栏里输入reboot ，输入命令后稍微等待一会服务器就会自动重启，一般重启过程需要2～5分钟，重启过程中Xshell会自动断开连接，等VPS重启好后才可以用Xshell软件进行连接。如果部署过程中卡在某个位置超过10分钟，可以用xshell软件断开，然后重新连接你的ip，再复制代码进行部署。

## 三：加速SSR

此加速教程为谷歌BBR加速，Vultr的服务器框架可以装BBR加速，加速后对速度的提升很明显，所以推荐部署加速脚本。该加速方法是开机自动启动，部署一次就可以了。

按照第二步的步骤，连接服务器ip，登录成功后，在命令栏里粘贴以下代码：

```
yum -y install wget

wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh

chmod +x bbr.sh

./bbr.sh
```

把上面整个代码复制后粘贴进去，不动的时候按回车，然后耐心等待，最后重启vps服务器即可。

复制并粘贴代码后，按回车键确认，演示开始，如图：

![](https://subetter.com/images/figures/20180421_22.png)

如下图提示，按任意键继续部署，

![](https://subetter.com/images/figures/20180421_23.png)

![](https://subetter.com/images/figures/20180421_24.png)

部署到上图这个位置的时候，等待3～6分钟，

![](https://subetter.com/images/figures/20180421_25.png)

最后输入y重启服务器，如果输入y提示command not found ，接着输入reboot来重启服务器，确保加速生效，bbr加速脚本是开机自动启动，装一次就可以了。

服务器重启成功并重新连接服务器后，输入命令lsmod | grep bbr 如果出现tcp_bbr字样表示bbr已安装并启动成功。如图：

![](https://subetter.com/images/figures/20180421_26.PNG)

## 四：SSR客户端下载

第一次电脑系统使用SSR/SS客户端时，如果提示你需要安装NET Framework 4.0，网上搜一下这个东西，安装一下即可。NET Framework 4.0是SSR/SS的运行库，没有这个SSR/SS客户端无法正常运行。有的电脑系统可能会自带NET Framework 4.0。

>Windows SSR客户端 [下载地址](https://github.com/shadowsocksr-backup/shadowsocksr-csharp/releases) [备用下载地址](https://nofile.io/f/6Jm7WJCyOVv/ShadowsocksR-4.7.0-win.7z)
>MAC SSR客户端 [下载地址](https://github.com/shadowsocksr-backup/ShadowsocksX-NG/releases) [备用下载地址](https://nofile.io/f/jgMWFwCBonU#ab0d3c3b6ac54482)
>[Linux客户端一键安装配置使用脚本(使用方法见注释)](https://github.com/the0demiurge/CharlesScripts/blob/master/charles/bin/ssr) 或者采用图形界面的[linux ssr客户端](https://github.com/erguotou520/electron-ssr/releases)
>安卓SSR客户端 [下载地址](https://github.com/shadowsocksr-backup/shadowsocksr-android/releases/download/3.4.0.8/shadowsocksr-release.apk) [备用下载地址](https://nofile.io/f/rvTJoj0h5GC/shadowsocksr-release.apk)

苹果手机SSR客户端：Potatso Lite、Potatso、shadowrocket都可以作为SSR客户端，但这些软件目前已经在国内的app商店下架，可以用美区的appid账号来下载。但是，如果你配置的SSR账号兼容SS客户端，或者协议选择origin且混淆选择plain，那么你可以选择苹果SS客户端软件（即协议和混淆可以不填），APP商店里面有很多，比如：superwingy、firstwingy、shadowingy、wingy+、banananet、kite-ss proxy、goodshadow、icproyx、shadowrocket等。

打开SSR客户端，填上信息，这里以windows版的SSR客户端为例子，

![](https://subetter.com/images/figures/20180421_27.PNG)

在对应的位置，填上服务器ip、服务器端口、密码、加密方式、协议和混淆，最后将浏览器的代理设置为（http）127.0.0.1和1080即可。账号的端口号就是你自己设置的，而要上网的浏览器的端口号是1080，固定的，谷歌浏览器可以通过 SwitchyOmega 插件来设置，设置方法如下:

进入官网下载[ SwitchyOmega ](https://www.switchyomega.com/)，安装到Google浏览器中，新建情景模式，然后设置成，

![](https://subetter.com/images/figures/20180421_29.png)

启动SSR客户端后，右键SSR客户端图标，选择第一个“系统代理模式”，里面有3个子选项，“直连模式”就是不走代理；“PAC模式”就是国外走代理，国内直连（推荐）；"全局模式“就是国内国外都走代理。

![](https://subetter.com/images/figures/20180421_28.png)

## 五：参考

- [https://github.com/Alvin9999/new-pac/wiki/%E8%87%AA%E5%BB%BAss%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%95%99%E7%A8%8B](https://github.com/Alvin9999/new-pac/wiki/%E8%87%AA%E5%BB%BAss%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%95%99%E7%A8%8B)