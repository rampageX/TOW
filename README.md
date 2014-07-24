# TOW (	Transparent Over the Wall )

TOW 是一个安装在路由器系统上的软件包，安装之后，可以保证连接在这个路由器上的所有客户端透明翻墙。

当前版本：1.0

## 功能

TOW 的设计目标是透明化/自动化，理想情况下客户端用户无需关心哪些网站无法访问，可直连网站也不会因为使用二级代理而降低访问速度。

- 使用 pdnsd 特性防止 DNS 污染，不使用 TCP 查询，保证 CDN 效果
- 支持 GoAgent, SOCKS5, [shadowsocks](https://github.com/clowwindy/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E) 和 Obfuscated ssh 等代理服务器
- 使用 gfwlist 和 ipset 配合 iptables 处理被墙网站，仅对被墙网站使用代理

# 依赖

- 一台支持 [Tomato Shibby MOD](http://http://tomato.groov.pl/) （国内为 [bwq518 增强版](http://www.right.com.cn/forum/thread-126746-1-1.html)，已自带 TOW 功能，推荐使用）或者 [RMerlin](http://www.lostrealm.ca/tower/node/80), [Padavan](http://code.google.com/p/rt-n56u/) 固件的路由器
- 固件内含的 iptables/dnsmasq 必须编译支持 ipset

# 安装：(均以 Tomato 为例）

准备好 Putty 和 WinSCP 工具；


如果 Flash 空间足够，启用 JFFS ，格式化后，SSH 登陆路由器：


```sh
cd /jffs
mkdir opt
mount -o bind /jffs/opt /opt
```
下载软件包，[tow-1.0.tar.gz](https://dl.dropboxusercontent.com/u/200370/Router/%21unique4g/backup/tow-1.0.tar.gz)，用 WinSCP 上传到路由器 tmp 目录，然后执行：


```sh
cd /
tar xvzf /tmp/tow-1.0.tar.gz
```
如果 Flash 空间不够，请挂载 U 盘并且在 U 盘上创建 opt 目录，步骤可 Google，例如：


```sh
mount -o bind /mnt/sda1/opt /opt
cd /
tar xvzf /tmp/tow-1.0.tar.gz
cp /opt/.autorun /mnt/sda1/   ###注意这一步不要漏掉，自启动脚本必须放在挂载点根目录
```
请确保 U 盘挂载的分区格式为 ext2/ext3/ext4，optware 不能安装运行在 ntfs 格式上！

pdnsd 内置 114 DNS 服务器解析本地域名，如果对速度不满意，可以修改 /opt/etc/pdnsd.conf 文件第 27 行：


```
	ip = 114.114.114.114,114.114.115.115;
```
改为你本地 ISP 的 DNS 服务器地址。

路由器Web界面设置：

1.高级设置-DHCP/DNS-自定义设置：


```
conf-dir=/opt/etc/dnsmasq/custom/
```
2. 系统管理-JFFS-挂载后执行：


```
mount -o bind /jffs/opt /opt
```
如果使用 U 盘，则在：USB and NAS-USB 支持-挂载后运行：


```
mount -o bind /mnt/sda1/opt /opt
```
3. 系统管理-定时重启/连接-自定义(每12小时）：


```
/opt/etc/init.d/S10dnsmasqc
```

重启路由器

再次 SSH 登入路由器，运行：


```sh
netstat -lnp
```

应该可以看到 tcp 有 8099 和 7070 端口在监听；tcp/udp 有 5454 在监听；

运行：


```sh
iptables -t nat -nvL
```

应该看到有 REDSOCKS 或者 SHADOWSOCKS 的 Chian 存在。

给出的软件包默认工作在 `ShadowSocks redir` + 全子网模式，也就是在路由器子网内所有客户端透明翻墙。

请修改： `/opt/etc/py/ga/proxy.ini` 填入你自己的 GoAgent 账号。

请修改：`/opt/etc/shadowsocks.json`, `/opt/etc/shadowsocks_redir.json`文件，填入你自己的服务器地址端口和密码，本地监听端口`7070/7272`不可改！

请修改： `/opt/etc/config/*.fire` 文件，填入你自己的 `ssh_server` 和 `ss_server` 域名；

请修改： `/opt/etc/init.d/S27obssh` 文件，填入 SSH 登录信息

# 流程以及文件解析

### 自启动以及防火墙

关键目录和文件：`/opt/.autorun`, `/opt/etc/config`。

`/opt/.autorun` 文件利用了 Tomato Optware 的特点：在任何挂载设备根目录下如果存在 `.autorun` 结尾的文件,则在挂载此设备后执行该文件。`.autorun` 文件代码：
```sh
if [ -f /var/notice/wan ]; then
   for s in /opt/etc/config/*.wanup; do $s; done
   for s in /opt/etc/config/*.fire; do $s; done
fi
```
表示在 WAN 上线后，运行 `/opt/etc/config` 下所有以 `wanup` 和 `fire` 为后缀的文件。

`*.fire` 为防火墙脚本，和 Tomato 图形界面中设置防火墙脚本作用是一样的。这个脚本在 TOW 方案中极为重要，可以仔细阅读一下源代码；

`*.wanup` 为自启动程序列表，所需的主要程序 shadowsocks，redsocks 等都由这个文件调用启动。

### DNS 处理

关键目录和文件： `/opt/etc/dnsmasq`

...待完成...

### 程序的启动/停止/重启以及监控

关键目录和文件： `/opt/etc/init.d` 下 S 开头的文件

...待完成...

### 程序配置文件

关键目录和文件： `/opt/etc` 以及 `/opt/etc/init.d/S27obssh`

...待完成...

### 手动维护 ipset 的 gfwlist 列表

关键文件： `/opt/etc/init.d` 下 dmadd, dmdel, dmcheck 三个文件

...待完成...

# 致谢

贡献代码：

- autogfwlist 维护的黑名单；
- lantern 维护的黑名单；
- n0wa11 维护的白名单；
- panda 维护的白名单；
- semigodking 修改的 redsocks2；
- OpenWRT RA-MOD；
- fqdns/西厢3的 dns 钓鱼脚本；

