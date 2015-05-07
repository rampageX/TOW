Based on [Shibby's Tomato Firmware](http://tomato.groov.pl/)

# 安装：

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