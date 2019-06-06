更多nmcli设置，参考：https://developer.gnome.org/NetworkManager/stable/ref-settings.html

#### 扫描周围可见的WiFi
```bibtex
nmcli device wifi list
```

#### 查看已有的网络连接
```bibtex
nmcli connection show
```

#### 删除不需要的网络连接
```bibtex
sudo nmcli connection delete LINK_NAME
```

#### 复制网络连接
```bibtex
sudo nmcli connection clone LINK_NAME LINK_NAME_NEW
```

#### 关闭指定的网络连接
```bibtex
sudo nmcli connection down LINK_NAME
```
注意：使用这个指令时确保要关闭的连接没有启用自动连接，否则可能会自动重连，达不到要关闭的效果，此时使用‘关闭指定的网卡’指令更加可靠


#### 开启指定的网络连接
```bibtex
sudo nmcli connection up LINK_NAME
```
注意：如果正在使用的连接跟将要开启的连接不是同一个，可以不需要关闭正在使用的连接直接 up，会自动替换；


#### 关闭指定的网卡
```bibtex
sudo nmcli device disconnect wlp58s0
```
注意：关闭网卡后，使用‘开启指定的网络连接’指令将会打开网卡并连接，不需要再执行‘开启指定的网卡’指令


#### 开启指定的网卡
```bibtex
sudo nmcli device connect wlp58s0
```



### 直接连接到扫描到的网络
```bibtex
sudo nmcli device wifi connect TP-LINK_5G_4A49 \
password Lanyinkejiccs \
name LINK_NAME \
hidden no &&\
sudo nmcli connection modify LINK_NAME \
ipv4.method manual \
ipv4.addresses 192.168.1.204/24 \
ipv4.gateway 192.168.1.254 \
ipv4.dns 192.168.1.254 && \
sudo nmcli connection down LINK_NAME && \
sudo nmcli connection up LINK_NAME
```
#### 注意：

1.‘hidden no’用来表示连接到的网络不是隐藏的，此时可以省去这一项，当连接到隐藏的网络时，则必须指明为 yes；
  
2.还存在‘ifname’用来指定使用哪张网卡执行这个连接，不指定时省略掉；
  
3.使用‘nmcli device wifi connect’命令连接已知 WiFi 时，IP将是自动分配，为方便SSH连接，必须使用‘&&’搭配指定 IP 和重启网络连接的命令；
  
4.如果不使用‘name LINK_NAME’来指定连接的名字，将会自动以对应的 SSID 来命名；
  
5.不指定连接名地重复连接同一个 WiFi 时，从第二次开始会自动以 SSID_n 来命名，n 从1开始逐次加1，此时，在指定 IP 和重启网络连接的命令中，LINK_NAME 必须修改为对应的连接名。

### 手动建立新的WiFi连接
```bibtex
sudo nmcli connection add \
con-name LINK_NAME \
ifname '*' \
autoconnect yes \
type wifi \
ssid TP-LINK_5G_4A49 \
-- \
802-11-wireless-security.key-mgmt wpa-psk \
802-11-wireless-security.psk Lanyinkejiccs \
ipv4.method manual \
ipv4.addresses 192.168.1.204/24 \
ipv4.gateway 192.168.1.254 \
ipv4.dns 192.168.1.254
```
#### 说明：

con-name ：连接命名
  
ifname ：用于指定网卡，不指定时必须用 '*' 代替
  
autoconnect ：设置自动连接

### 手动建立新的热点连接
```bibtex
sudo nmcli connection add \
con-name LINK_NAME \
ifname '*' \
autoconnect no \
type wifi \
mode ap \
ssid AP_NAME \
-- \
802-11-wireless.band bg \
802-11-wireless-security.key-mgmt wpa-psk \
802-11-wireless-security.psk PASSWORD \
ipv4.method shared
ipv4.addresses 10.42.0.1/24 \
```
#### 说明：

con-name ：用于给连接命名
  
ifname ：用于指定网卡，不指定时必须用 '*' 代替
  
autoconnect ：设置自动连接

### 启用WiFi并关闭热点
```bibtex
sudo nmcli connection modify WiFi connection.autoconnect yes 802-11-wireless.ssid TP-LINK_5G_4A49 802-11-wireless.hidden no 802-11-wireless-security.key-mgmt wpa-psk 802-11-wireless-security.psk Lanyinkejiccs ipv4.method manual ipv4.addresses 192.168.1.204/24 ipv4.gateway 192.168.1.254 ipv4.dns 192.168.1.254 && sudo nmcli connection modify Hotspot connection.autoconnect no && sudo nmcli connection reload && sudo nmcli connection down Hotspot && sudo nmcli connection up WiFi
```
#### 以上命令的拆解如下：
1.设置WiFi的自动连接、SSID、密码、IP地址
```bibtex
sudo nmcli connection modify WiFi \
connection.autoconnect yes \
802-11-wireless.ssid TP-LINK_5G_4A49 \
802-11-wireless.hidden no \
802-11-wireless-security.key-mgmt wpa-psk \
802-11-wireless-security.psk Lanyinkejiccs \
ipv4.method manual \
ipv4.addresses 192.168.1.204/24 \
ipv4.gateway 192.168.1.254 \
ipv4.dns 192.168.1.254 
```
2.关闭热点的自动连接
```bibtex
sudo nmcli connection modify Hotspot connection.autoconnect no
```
3.刷新修改后关闭热点，并启用WiFi连接
```bibtex
sudo nmcli connection reload && \
sudo nmcli connection down Hotspot && \
sudo nmcli connection up WiFi
```
注意：这里这里开启的网络连接名是 WiFi，关闭的热点连接名是 Hotspot，直接使用这些指令前先确保存在这两个网络连接，或者替换为你要使用的连接名。

### 启用热点并关闭WiFi
```bibtex
sudo nmcli connection modify Hotspot connection.autoconnect yes 802-11-wireless.mode ap 802-11-wireless.band bg 802-11-wireless.ssid AP_TEST 802-11-wireless-security.key-mgmt wpa-psk 802-11-wireless-security.psk 11112222 ipv4.method shared ipv4.addresses 10.42.0.1/24 && sudo nmcli connection modify WiFi connection.autoconnect no && sudo nmcli connection reload && sudo nmcli connection down WiFi && sudo nmcli connection up Hotspot
```
#### 以上命令的拆解如下：
1.设置热点的自动连接、SSID、密码、IP地址
```bibtex
sudo nmcli connection modify Hotspot \
connection.autoconnect yes \
802-11-wireless.mode ap \
802-11-wireless.band bg \
802-11-wireless.ssid AP_TEST \
802-11-wireless.hidden no \
802-11-wireless-security.key-mgmt wpa-psk \
802-11-wireless-security.psk 11112222 \
ipv4.method shared \
ipv4.addresses 10.42.0.1/24
```
2.关闭WiFi的自动连接
```bibtex
sudo nmcli connection modify WiFi connection.autoconnect no
```
3.刷新修改后关闭WiFi，并启用热点连接
```bibtex
sudo nmcli connection reload && \
sudo nmcli connection down WiFi && \
sudo nmcli connection up Hotspot
```
注意：这里这里开启的热点连接名是 Hotspot，关闭的网络连接名是 WiFi，直接使用这些指令前先确保存在这两个网络连接，或者替换为你要使用的连接名。
