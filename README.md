# 红米 ax6000 刷机教程

1. 登录后台 http://192.168.31.1/ 默认后台密码是无线网密码

2. 升级到指定版本固件 redmi-ax6000-1.2.8.bin

3. 获取 sock（登陆小米路由器后台后，浏览器地址栏 stok= 后面的一段内容即是）
   ![](./img/2022-11-19-00-23-06-image.png)

4. 打开浏览器，复制下面的内容到地址栏，并替换下面的{token}为 stok

```
http://192.168.31.1/cgi-bin/luci/;stok={token}/api/misystem/set_sys_time?timezone=%20%27%20%3B%20zz%3D%24%28dd%20if%3D%2Fdev%2Fzero%20bs%3D1%20count%3D2%202%3E%2Fdev%2Fnull%29%20%3B%20printf%20%27%A5%5A%25c%25c%27%20%24zz%20%24zz%20%7C%20mtd%20write%20-%20crash%20%3B%20
```

![](/Users/z/Downloads/ax6000/img/2022-11-19-00-25-21-image.png)

如果显示 {"code":0} 如上返回内容则表示成功

然后重新打开 http://192.168.31.1

通过浏览器执行完这一步路由器会重启，等待重启完成（注意，电脑会自动连上其他网络，需要手工切换回来）。

5. 重启完成后打开路由器后台（注意：路由器重启你需要重新登陆获取下新的 stok ），然后同样打开浏览器，复制下面的内容到地址栏，并替换 {STOK} 。

```
http://192.168.31.1/cgi-bin/luci/;stok={token}/api/misystem/set_sys_time?timezone=%20%27%20%3B%20bdata%20set%20telnet_en%3D1%20%3B%20bdata%20set%20ssh_en%3D1%20%3B%20bdata%20set%20uart_en%3D1%20%3B%20bdata%20commit%20%3B%20
```

如果显示 {"code":0} 如上返回内容则表示成功

然后再粘贴 (记得替换 token)

```
http://192.168.31.1/cgi-bin/luci/;stok={token}/api/misystem/set_sys_time?timezone=%20%27%20%3b%20reboot%20%3b%20
```

通过浏览器执行完这一步路由器会重启，等待重启完成（注意，电脑会自动连上其他网络，需要手工切换回来）。

6. 永久开启并固化 ssh

先 `telnet 192.168.31.1`

```
echo -e 'gooutgfw\ngooutgfw' | passwd root
nvram set ssh_en=1
nvram set telnet_en=1
nvram set uart_en=1
nvram set boot_wait=on
nvram commit
sed -i 's/channel=.*/channel="debug"/g' /etc/init.d/dropbear
/etc/init.d/dropbear restart
mkdir /data/auto_ssh
cd /data/auto_ssh

cat > auto_ssh.sh << EOF
#!/bin/sh
host_key=/etc/dropbear/dropbear_rsa_host_key
host_key_bk=/data/auto_ssh/dropbear_rsa_host_key
if [ -f \$host_key_bk ]; then
    ln -sf \$host_key_bk \$host_key
fi

channel=\`/sbin/uci get /usr/share/xiaoqiang/xiaoqiang_version.version.CHANNEL\`
if [ "\$channel" = "release" ]; then
    sed -i 's/channel=.*/channel="debug"/g' /etc/init.d/dropbear
    /etc/init.d/dropbear restart
fi
if [ ! -s \$host_key_bk ]; then
    i=0
    while [ \$i -le 30 ]
    do
        if [ -s \$host_key ]; then
            cp -f \$host_key \$host_key_bk 2>/dev/null
            break
        fi
        let i++
        sleep 1s
    done
fi
EOF

chmod +x auto_ssh.sh
uci set firewall.auto_ssh=include
uci set firewall.auto_ssh.type='script'
uci set firewall.auto_ssh.path='/data/auto_ssh/auto_ssh.sh'
uci set firewall.auto_ssh.enabled='1'
uci commit firewall
uci set system.@system[0].timezone='CST-8'
uci set system.@system[0].webtimezone='CST-8'
uci set system.@system[0].timezoneindex='2.84'
uci commit
mtd erase crash
reboot
```

复制上面的命令到 termius 终端里执行

这会设置 ssh root 用户的密码为 gooutgfw 
并永久开启 SSH、并从开发模式修改回正常的模式，并重启。

等待重启完成后就能连上 ssh 了。

如果重置了路由器，密码会回复为默认密码。

可以通过 SN 获取默认 root 账号的密码（SN 码在路由器底部或路由器管理页面能查看到）

![](https://raw.githubusercontent.com/gcxfd/img/gh-pages/qu0JKj.png)

7. 在 上网设置-> 工作模式切换 中 改为无线中续

然后就可以上网了

找到新的 ip,ssh 上去

`cat /proc/cmdline`

查看 firmware= 多少

如果是 0 执行

```
nvram set boot_wait=on
nvram set uart_en=1
nvram set flag_boot_rootfs=1
nvram set flag_last_success=1
nvram set flag_boot_success=1
nvram set flag_try_sys1_failed=0
nvram set flag_try_sys2_failed=0
nvram commit
cd /tmp
curl -L http://sebs.oss-cn-shanghai.aliyuncs.com/initramfs-factory.ubi -o initramfs-factory.ubi
ubiformat /dev/mtd9 -y -f /tmp/initramfs-factory.ubi
reboot -f
```

如果是 1 执行

```
nvram set boot_wait=on
nvram set uart_en=1
nvram set flag_boot_rootfs=0
nvram set flag_last_success=0
nvram set flag_boot_success=1
nvram set flag_try_sys1_failed=0
nvram set flag_try_sys2_failed=0
nvram commit
cd /tmp
curl -L http://sebs.oss-cn-shanghai.aliyuncs.com/initramfs-factory.ubi -o initramfs-factory.ubi
ubiformat /dev/mtd8 -y -f /tmp/initramfs-factory.ubi
reboot -f
```

复制执行完就会重启进入过渡固件，过渡固件的

无线网名称 : OpenWrt
无线网密码 : password
管理后台 : http://192.168.5.1
用户名和密码 : root/password 或者 admin/password

进去管理后台，点 服务-> 终端

输入 root 回车
输入 passoword 回车

然后再粘贴下面的代码

```
fw_setenv boot_wait on
fw_setenv uart_en 1
fw_setenv flag_boot_rootfs 0
fw_setenv flag_last_success 1
fw_setenv flag_boot_success 1
fw_setenv flag_try_sys1_failed 8
fw_setenv flag_try_sys2_failed 8
```

然后打开系统-备份与升级，选择刷写固件。浏览里找到下载好的 openwrt 固件，选择上传。上传好后选择取消勾选保留当前配置然后选择继续。等待刷写完成自动重启 (5 分钟左右)，就会进入新的 openwrt 固件。

新的无线网络名称是 Redmi_AX6000_5G
固件后台地址是 192.168.6.1 用户名和密码依然是 root 和 password，默认的网口 1 是 wan 口，剩下的都是 lan 口。

为了安全性访问 http://192.168.6.1/cgi-bin/luci/admin/system/admin 修改密码

访问 http://192.168.6.1/cgi-bin/luci/admin/network/wifi 点基本配置，修改网络名称和密码，加密方式选 WPA2PSK

另外可以配置无线中续

访问 http://192.168.6.1/cgi-bin/luci/admin/services/openclash/config-subscribe

添加 clashx 订阅，勾上自动更新，更新模式选择循环

点击 http://192.168.6.1/cgi-bin/luci/admin/services/openclash 底部的启动 OpenClash

然后点击 Dashboard 控制面板 旁边的打开控制面板

访问 http://192.168.6.1/cgi-bin/luci/admin/services/openclash/settings
可以选上 实验性：绕过中国大陆 IP
可以取消 禁用 QUIC

然后到页面最底部，点击保存配置，再点应用配置

在 http://192.168.6.1/cgi-bin/luci/admin/network/network 中，有个名为 WAN6 的接口，删除掉

在 http://192.168.6.1/cgi-bin/luci/admin/network/network/wan ，取消使用内置的 ipv6 管理，然后点保存 & 应用

在 http://192.168.6.1/cgi-bin/luci/admin/network/network/lan ，拉到页面底部，点 ipv6 设置，全部禁用，然后点保存 & 应用

在 http://192.168.6.1/cgi-bin/luci/admin/network/turboacc ，勾上设置 BBR 拥塞控制

## 参考教程

1. [红米 ax6000 刷 openwrt 教程，终于有完善好用的 openwrt 了](https://qust.me/post/ax6000-openwrt) 
2. [刷机视频步骤 1](https://www.youtube.com/watch?v=AkBrLBjpc_k)
3. [刷机视频步骤 2](https://www.youtube.com/watch?v=FCvCTyjoeq4)
4. [固件新版本下载](https://www.right.com.cn/forum/thread-8261104-1-1.html)
