在 Ubuntu 中设置静态 IP 地址可以通过多种方式实现，主要包括使用 Netplan（命令行，适合服务器）、NetworkManager（命令行或图形界面，适合桌面版）或直接修改配置文件。以下我会通俗易懂地详细讲解主要方法，适合 Ubuntu 18.04 及以上版本（包括 20.04、22.04、24.04），并提供步骤、示例和注意事项。
为什么要设置静态 IP？
静态 IP 地址是固定的，不会随着重启或网络变化而改变。相比动态 IP（由路由器 DHCP 自动分配），静态 IP 适合以下场景：
服务器（如 Web 服务器、文件服务器）需要固定地址。
端口转发或远程访问需要稳定的 IP。
避免动态 IP 变化导致的服务中断。
方法一：使用 Netplan（命令行，适合 Ubuntu 服务器）
Ubuntu 18.04 及以上版本默认使用 Netplan 管理网络配置，配置文件是 YAML 格式，适合服务器环境。
步骤
查看当前网络信息：
打开终端，运行以下命令查看网络接口名称和当前 IP 配置：
bash
ip a
输出示例：
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic ens33
ens33 是网络接口名称，192.168.1.100 是当前 IP，/24 表示子网掩码（255.255.255.0）。
记录接口名称（例如 ens33 或 eth0）、当前 IP、子网掩码和网关。
查找网关和 DNS：
查看网关：
bash
ip route
输出示例：default via 192.168.1.1 dev ens33
网关是 192.168.1.1。
DNS 通常使用公共服务器（如 Google 的 8.8.8.8 和 8.8.4.4），或查看当前 DNS：
bash
nmcli dev show | grep DNS
备份 Netplan 配置文件：
Netplan 配置文件位于 /etc/netplan/，通常是 01-netcfg.yaml 或 00-installer-config.yaml。
查看文件：
bash
ls /etc/netplan/
备份文件以防出错：
bash
sudo cp /etc/netplan/01-netcfg.yaml /etc/netplan/01-netcfg.yaml.bak
编辑 Netplan 配置文件：
使用编辑器（如 nano）修改配置文件：
bash
sudo nano /etc/netplan/01-netcfg.yaml
修改或添加以下内容（根据你的网络信息调整）：
yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33: # 替换为你的接口名称
      dhcp4: no
      addresses:
        - 192.168.1.100/24 # 静态 IP 和子网掩码
      routes:
        - to: default
          via: 192.168.1.1 # 网关
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4] # DNS 服务器
说明：
dhcp4: no：禁用 DHCP，启用静态 IP。
addresses：设置静态 IP，/24 表示子网掩码 255.255.255.0。
routes：设置默认网关。
nameservers：设置 DNS 服务器。
注意：YAML 文件对缩进敏感，使用 2 个空格，不要用 Tab 键。
验证配置文件：
保存文件（Ctrl+O，Enter，Ctrl+X 退出 nano）。
测试配置是否有语法错误：
bash
sudo netplan try
如果没有错误，会提示确认（按 Enter 接受）。
如果有错误，检查缩进或语法。
应用配置：
应用更改：
bash
sudo netplan apply
验证静态 IP：
检查 IP 是否更新：
bash
ip a
确认 ens33 显示为 192.168.1.100/24 且没有 dynamic 字样。
测试网络连接：
bash
ping 8.8.8.8
如果有响应，说明网络正常。
示例配置文件
假设你的接口是 ens33，想设置 IP 为 192.168.1.100，网关为 192.168.1.1，DNS 为 Google 的服务器：
yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses: [192.168.1.100/24]
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
注意事项
IP 冲突：确保选择的 IP（如 192.168.1.100）不在路由器的 DHCP 分配范围内，避免冲突。
权限：配置文件需正确权限，运行：
bash
sudo chmod 600 /etc/netplan/01-netcfg.yaml
重启测试：重启系统后检查 IP 是否保持：
bash
sudo reboot
方法二：使用 NetworkManager（图形界面，适合 Ubuntu 桌面版）
Ubuntu 桌面版默认使用 NetworkManager，可以通过图形界面（GUI）设置静态 IP，简单直观。
步骤
打开网络设置：
点击屏幕右上角的网络图标（Wi-Fi 或有线图标）。
选择“设置”或“有线设置”（Wired Settings）。
选择网络接口：
在弹出的“设置”窗口中，点击左侧的“网络”（Network）。
找到你的网络连接（有线或 Wi-Fi），点击旁边的齿轮图标。
配置静态 IP：
在新窗口中，切换到 IPv4 标签。
将“IPv4 方法”从“自动（DHCP）”改为手动（Manual）。
输入以下信息：
地址（Address）：静态 IP，如 192.168.1.100。
子网掩码（Netmask）：通常是 255.255.255.0（对应 /24）。
网关（Gateway）：如 192.168.1.1。
DNS：输入 DNS 服务器，如 8.8.8.8,8.8.4.4（用逗号分隔）。
关闭“自动 DNS”开关以确保使用手动 DNS。
应用设置：
点击“应用”（Apply）按钮。
关闭窗口。
重启网络连接：
在网络设置中，关闭网络开关（toggle off），然后重新打开（toggle on）。
或者断开并重新连接网络。
验证：
打开终端，检查 IP：
bash
ip a
确认接口显示为 192.168.1.100/24。
测试网络：
bash
ping 8.8.8.8
示例
假设你设置：
IP 地址：192.168.1.100
子网掩码：255.255.255.0
网关：192.168.1.1
DNS：8.8.8.8,8.8.4.4
完成后，网络会使用这个固定 IP。
注意事项
直观但有限：GUI 适合简单配置，但复杂网络可能需要命令行。
Wi-Fi vs 有线：Wi-Fi 配置类似，选择对应的 Wi-Fi 连接即可。
权限：GUI 操作会自动处理权限，无需额外命令。
方法三：使用 NetworkManager 的命令行工具（nmcli）
如果不想用 GUI 或 Netplan，可以用 nmcli（NetworkManager 的命令行工具）设置静态 IP，适合桌面和服务器。
步骤
检查当前连接：
查看网络连接名称：
bash
nmcli con show
输出示例：
NAME                UUID                                  TYPE      DEVICE
Wired connection 1  0fb27c8a-2e77-31c6-b212-2b3daa5e47e6  ethernet  ens33
记录连接名称（如 Wired connection 1）和设备名称（如 ens33）。
设置静态 IP：
创建或修改连接，设置静态 IP：
bash
sudo nmcli con mod "Wired connection 1" ipv4.addresses "192.168.1.100/24"
sudo nmcli con mod "Wired connection 1" ipv4.gateway "192.168.1.1"
sudo nmcli con mod "Wired connection 1" ipv4.dns "8.8.8.8,8.8.4.4"
sudo nmcli con mod "Wired connection 1" ipv4.method manual
说明：
ipv4.addresses：设置 IP 和子网掩码。
ipv4.gateway：设置网关。
ipv4.dns：设置 DNS。
ipv4.method manual：禁用 DHCP，使用静态 IP。
应用更改：
激活连接：
bash
sudo nmcli con up "Wired connection 1"
验证：
检查 IP：
bash
ip a
测试网络：
bash
ping 8.8.8.8
示例
完整命令：
bash
sudo nmcli con mod "Wired connection 1" ipv4.addresses "192.168.1.100/24" ipv4.gateway "192.168.1.1" ipv4.dns "8.8.8.8,8.8.4.4" ipv4.method manual
sudo nmcli con up "Wired connection 1"
注意事项
连接名称：确保使用正确的连接名称（nmcli con show 查看）。
安装 nmcli：服务器版可能未默认安装，运行：
bash
sudo apt update
sudo apt install network-manager
重启：重启后检查 IP 是否保留。
方法四：通过路由器设置静态 IP（推荐）
与其在 Ubuntu 上手动设置静态 IP，更推荐的方式是通过路由器的 DHCP 保留功能（也叫静态 DHCP 分配）：
步骤：
登录路由器管理界面（通常是 192.168.1.1 或 192.168.0.1）。
找到“DHCP 分配”或“地址保留”选项。
添加设备（用 MAC 地址标识），分配固定 IP（如 192.168.1.100）。
保存并重启路由器或设备。
优点：
集中管理，所有静态 IP 在路由器上配置。
避免 Ubuntu 配置错误导致网络问题。
设备仍通过 DHCP 连接，简化维护。
注意：需要知道设备的 MAC 地址（运行 ip link 查看）。
常见问题与解决
网络不通：
检查 IP 是否与路由器 DHCP 范围冲突。
确认网关和 DNS 是否正确。
尝试重启网络服务：
bash
sudo systemctl restart networking
配置文件错误：
Netplan YAML 缩进错误会导致失败，使用 sudo netplan try 检查。
恢复备份：
bash
sudo cp /etc/netplan/01-netcfg.yaml.bak /etc/netplan/01-netcfg.yaml
接口名称未知：
使用 ip a 或 nmcli dev show 查看接口名称。
常见名称：eth0（旧）、ens33、enp0s3 等。
权限问题：
确保配置文件权限正确：
bash
sudo chmod 600 /etc/netplan/*.yaml
Wi-Fi 设置：
Wi-Fi 配置类似，但接口名称可能是 wlan0 或 wlp2s0。
GUI 或 nmcli 更适合 Wi-Fi。
综合示例
假设你想设置：
接口：ens33
静态 IP：192.168.1.100
子网掩码：255.255.255.0（/24）
网关：192.168.1.1
DNS：8.8.8.8, 8.8.4.4
Netplan 方式
bash
sudo nano /etc/netplan/01-netcfg.yaml
内容：
yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens33:
      dhcp4: no
      addresses: [192.168.1.100/24]
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
应用：
bash
sudo netplan try
sudo netplan apply
nmcli 方式
bash
sudo nmcli con mod "Wired connection 1" ipv4.addresses "192.168.1.100/24" ipv4.gateway "192.168.1.1" ipv4.dns "8.8.8.8,8.8.4.4" ipv4.method manual
sudo nmcli con up "Wired connection 1"
GUI 方式
打开网络设置 > IPv4 > 手动：
地址：192.168.1.100
子网掩码：255.255.255.0
网关：192.168.1.1
DNS：8.8.8.8,8.8.4.4
应用并重启连接。
总结
服务器：推荐 Netplan，配置文件清晰，适合批量管理。
桌面版：推荐 GUI 或 nmcli，操作简单。
路由器：最推荐通过 DHCP 保留设置，管理方便。
验证：设置后用 ip a 和 ping 确认网络正常。
备份：始终备份配置文件，避免错误。
如果你有具体环境（比如 Ubuntu 版本、服务器还是桌面、有线还是 Wi-Fi），可以告诉我，我会进一步优化步骤！
