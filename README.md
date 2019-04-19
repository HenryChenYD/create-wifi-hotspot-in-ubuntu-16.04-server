# wifi_hotspot_ubuntu_16.04
Create wifi hotspot in ubuntu 16.04 server on the command line.

## 相关知识简单说明

本设置wifi热点的方法是针对 server 版的，但其实是参考 desktop 版的配置方法（桌面版有 UI，不用这么麻烦）。

 Ubuntu 系统配置网络一般有两种方法：
* 通过修改配置文件 /etc/network/interfaces
* 通过 NetworkManager 管理
 
在 ubuntu 16.04 desktop 版中，默认安装有 NetworkManager，提供带 UI 的网络配置，而 server 版因为没有桌面 GUI，所以没有安装 NetworkManager，一般只能通过 /etc/network/interfaces 配置文件来配置网络，但我没找到利用该配置文件来设置 WiFi 热点的方法。
    
所以本方法其实就是通过安装 NetworkManager 来实现热点设置。╮(﹀_﹀")╭ 

## 准备工作

安装 NetworkManager:

    sudo apt-get install network-manager

通过这个命令其实也会自动安装 UI 相关的一些程序包，但无所谓啦。

安装完后，可以查看 NetworkManager 的配置文件

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

我们通过修改 /etc/network/interfaces 配置文件来管理网络连接，其实就是 ifupdown 在管理。

当我们将 managed 设置为 false 时，NetworkManager 将会管理配置所有的有线和无线网卡，interfaces 配置文件中的设置将无效，
当 managed 设置为 true 时，NetworkManager 只会管理没有在 interfaces 配置文件中列出的网卡。

#### 因为我已经在 interfaces 配置文件中设置好有线的局域网连接，不想重复配置，所以这里把 managed 修改为 true。

## 添加新的热点 Hotspot

在命令行中，我们是通过 nmcli 命令来管理维护 NetworkManager 的，这个工具在安装 NetworkManager 时会安装。

##### 以下命令中的 hotspot-name 和 hotspot-ssid 分别代表的是这个 WiFi 热点在本地保存的连接名称和在无线网络中的热点名称，自己定义。

```bibtex
# 添加新的热点 Create a connection
nmcli connection add type wifi ifname '*' con-name hotspot-name autoconnect yes ssid hotspot-ssid
```

autoconnect 设置为 yes 将会在服务启动后自动连接， 因为是在服务器运行，所以设置为 yes。

通过上述命令，生成了一个新的 WiFi 热点，它的配置文件保存在 /etc/NetworkManager/system-connections/ 文件夹里，名称为自定义的 hotspot-name。

继续设置：

```bibtex
# 设置工作模式为 AP，并且共享互联网连接 Put it in Access Point
nmcli connection modify hotspot-name 802-11-wireless.mode ap 802-11-wireless.band bg ipv4.method shared
```

```bibtex
# 设置加密方式和密码，密码 password 自己重新定义 Set a WPA password (you should change it)
nmcli connection modify hotspot-name 802-11-wireless-security.key-mgmt wpa-psk 802-11-wireless-security.psk password
```

启用热点的命令：

```bibtex
# 启用热点 Enable it (run this command each time you want to enable the access point)
nmcli connection up hotspot-name
```

如果在之前的步骤中，将 autoconnect 设置为 no，那么每次要启用热点，就是用这个命令。

## 最后一步

正式启用 NetworkManager 服务

```bibtex
# 启用服务
sudo systemctl start NetworkManager.service
# 设置开机启动
sudo systemctl enable NetworkManager.service
```

之所以没在安装 NetworkManager 后就启用服务，是因为我是通过 SSH 进去设置的，为了避免 NetworkManager 还没配置好就启用，可能会踢了连接。

最后，因为我使用的是 interfaces 配置文件管理有线网络连接，NetworkManager 管理无线网络连接，所以设置后上网出现问题，这时候重启一下 networking 服务就好：

    sudo systemctl restart networking.service

## 题外话

先前提到 NetworkManager 管理的网络连接配置文件保存在 /etc/NetworkManager/system-connections/ 文件夹里，所以修改配置的方式不是只有上面的命令，可以直接修改配置文件。

另外，nmcli 配置网络连接也不是只有 nmcli connection modify 这个用法，也有其它用法如：nmcli connection edit，这个略带交互的用法更高大上些，可以自行了解下。

## 参考链接

其实这个方法是网上找来的，自己重新整理下，方便以后使用。<(￣︶￣)>

https://askubuntu.com/questions/762846/how-to-create-wifi-hotspot-in-ubuntu-16-04-since-ap-hotspot-is-no-longer-working/872558

https://help.ubuntu.com/community/NetworkManager

https://blog.csdn.net/u012336923/article/details/50639425
