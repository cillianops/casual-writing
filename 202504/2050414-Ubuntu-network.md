```
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
```
