# wifi_hotspot_ubuntu_16.04
Create wifi hotspot in ubuntu 16.04 server on the command line.

## 相关知识简单说明

本设置wifi热点的方法是针对server版的，但其实是参考desktop版的配置方法（桌面版有UI，不用这么麻烦）。

 Ubuntu系统配置网络一般有两种方法：
* 通过修改配置文件 /etc/network/interfaces
* 通过 NetworkManager 管理
 
在ubuntu 16.04 desktop版中，默认安装有NetworkManager，提供带UI的网络配置，而server版因为没有桌面GUI，所以没有安装NetworkManager，一般只能通过/etc/network/interfaces配置文件来配置网络，但我没找到利用该配置文件来设置WiFi热点的方法。
    
所以本方法其实就是通过安装 NetworkManager 来实现热点设置。╮(﹀_﹀")╭ 

## 准备工作

安装 NetworkManager:

    sudo apt-get install network-manager

通过这个命令其实也会自动安装UI相关的一些程序包，但无所谓啦。

安装完后，可以查看NetworkManager的配置文件

    sudo cat /etc/NetworkManger/NetworkManager.conf

内容大概是这样：

```bibtex
[main]
plugins=ifupdown,keyfile,ofono
dns=dnsmasq

[ifupdown]
managed=false
```

这里要注意的是最后一项 manager 的设置。

我们通过修改/etc/network/interfaces配置文件来管理网络连接，其实就是ifupdown在管理。

当我们将managed设置为false时，NetworkManager将会管理配置所有的有线和无线网卡，interfaces配置文件中的设置将无效，
当managed设置为true时，NetworkManager只会管理没有在interfaces配置文件中列出的网卡。

#### 因为我已经在interfaces配置文件中设置好有线的局域网连接，不想重复配置，所以managed修改为true。


Start

    sudo systemctl start NetworkManager.service
    sudo systemctl enable NetworkManager.service
