# TOW (	Transparent Over the Wall )

TOW 是一个安装在路由器系统上的软件包，安装之后，可以保证连接在这个路由器上的所有客户端透明翻墙。

当前版本：1.1

## 功能

TOW 的设计目标是透明化/自动化，理想情况下客户端用户无需关心哪些网站无法访问，可直连网站也不会因为使用二级代理而降低访问速度。


# 依赖

- 一台支持 [Tomato Shibby MOD](http://http://tomato.groov.pl/) （国内为 [bwq518 增强版](http://www.right.com.cn/forum/thread-126746-1-1.html)，已自带 TOW 功能，推荐使用）或者 [RMerlin](http://www.lostrealm.ca/tower/node/80), [Padavan](http://code.google.com/p/rt-n56u/) 固件的路由器
- 固件内含的 iptables/dnsmasq 必须编译支持 ipset


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

