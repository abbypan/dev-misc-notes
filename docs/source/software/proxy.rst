proxy
=======

TUN：ap0 经 sing-box 转发到本地 SOCKS5
-----------------------------------------

以下配置用于 Arch Linux 热点或路由器场景，只接管从 ``ap0`` 进入的流量：

.. code-block:: text

        ap0 客户端
        ↓
        ap0（Arch Linux 网关）
        ↓
        sing-box TUN（singbox0）
        ↓
        SOCKS5 127.0.0.1:8008
        ↓
        Internet

前提条件
########

``ap0`` 已经创建并配置好地址、DHCP 等热点功能，客户端将 Arch Linux 主机作为
默认网关；本地 SOCKS5 服务监听 ``127.0.0.1:8008``，并支持需要转发的协议。

安装 sing-box 并确认接口及 SOCKS5 服务：

.. code-block:: bash

        sudo pacman -S sing-box
        ip address show dev ap0
        ss -lntup | grep ':8008'

设置 ap0 Wi-Fi 热点
###################

使用 NetworkManager 创建热点。先确认无线网卡支持 AP 模式，并且接口名称为
``ap0``；如果实际名称是 ``wlan0`` 等，需要同时替换下列命令和 sing-box 配置中的
``include_interface``：

.. code-block:: bash

        sudo pacman -S networkmanager
        sudo systemctl enable --now NetworkManager.service
        iw list | sed -n '/Supported interface modes:/,/Band/p'
        nmcli device status

创建名为 ``ap0-hotspot``、SSID 为 ``ArchProxy`` 的 WPA2 热点。将示例密码替换为
自己的密码；WPA2 密码长度必须为 8 到 63 个字符：

.. code-block:: bash

        sudo nmcli device wifi hotspot \
          ifname ap0 \
          con-name ap0-hotspot \
          ssid ArchProxy \
          password 'change-this-password'

        sudo nmcli connection modify ap0-hotspot \
          802-11-wireless-security.key-mgmt wpa-psk \
          802-11-wireless-security.psk 'change-this-password' \
          ipv4.method shared \
          ipv4.addresses 192.168.50.1/24 \
          connection.autoconnect yes

        sudo nmcli connection down ap0-hotspot
        sudo nmcli connection up ap0-hotspot

``ipv4.method shared`` 让 NetworkManager 为热点客户端提供 DHCP 和 DNS；客户端通常
会获得 ``192.168.50.0/24`` 网段的地址，并以 ``192.168.50.1`` 为默认网关。密码会
出现在 shell 历史记录中，不要直接在共享终端中输入真实密码。

查看热点状态和已连接设备：

.. code-block:: bash

        nmcli connection show --active
        ip address show dev ap0
        ip neigh show dev ap0

修改 SSID 或密码后重新激活连接：

.. code-block:: bash

        sudo nmcli connection modify ap0-hotspot \
          802-11-wireless.ssid NewSSID \
          802-11-wireless-security.psk 'new-password'
        sudo nmcli connection up ap0-hotspot

临时关闭热点使用 ``sudo nmcli connection down ap0-hotspot``。如需禁止开机自动启动，
执行 ``sudo nmcli connection modify ap0-hotspot connection.autoconnect no``。

开启转发
########

创建 ``/etc/sysctl.d/30-ip-forward.conf``：

.. code-block:: ini

        net.ipv4.ip_forward = 1

如果 ``ap0`` 同时提供 IPv6，再添加：

.. code-block:: ini

        net.ipv6.conf.all.forwarding = 1

应用配置：

.. code-block:: bash

        sudo sysctl --system

sing-box 配置
#############

创建 ``/etc/sing-box/config.json``：

.. code-block:: json

        {
          "log": {
            "level": "info"
          },
          "inbounds": [
            {
              "type": "tun",
              "tag": "tun-in",
              "interface_name": "singbox0",
              "address": [
                "172.19.0.1/30"
              ],
              "auto_route": true,
              "auto_redirect": true,
              "strict_route": true,
              "include_interface": [
                "ap0"
              ],
              "dns_mode": "disabled"
            }
          ],
          "outbounds": [
            {
              "type": "socks",
              "tag": "socks-out",
              "server": "127.0.0.1",
              "server_port": 8008,
              "version": "5"
            }
          ],
          "route": {
            "auto_detect_interface": true,
            "rules": [
              {
                "inbound": [
                  "tun-in"
                ],
                "action": "route",
                "outbound": "socks-out"
              }
            ]
          }
        }

``include_interface`` 将自动路由限制为 ``ap0``，因此主机自身以及其他接口的流量
不会被接管；``auto_detect_interface`` 用于避免 sing-box 的出站连接再次进入 TUN
形成回环。Linux 下建议同时启用 ``auto_route`` 和 ``auto_redirect``。

启动与验证
############

先检查配置，再启动服务：

.. code-block:: bash

        sudo sing-box check -c /etc/sing-box/config.json
        sudo systemctl enable --now sing-box.service
        sudo systemctl status sing-box.service

检查自动生成的 TUN、策略路由和 nftables 规则：

.. code-block:: bash

        ip address show dev singbox0
        ip rule show
        ip route show table 2022
        sudo nft list ruleset | grep -i sing-box
        journalctl -u sing-box.service -b

在连接到 ``ap0`` 的客户端上访问公网 IP 检查出口。若网页可访问但 DNS 失败，先确认
``127.0.0.1:8008`` 的 SOCKS5 服务支持 UDP，并检查热点下发的 DNS 地址。主机启用了
额外的 nftables、firewalld 或 ufw 规则时，还需要允许来自 ``ap0`` 的转发流量以及
DHCP、DNS 流量。此方案由 sing-box 终止连接并转交 SOCKS5，不要再为
``ap0`` 到 ``singbox0`` 添加 MASQUERADE 规则。

hysteria 的 chain proxy
----------------------------

install
########

https://github.com/apernet/hysteria

server:

.. code-block:: bash

        bash <(curl -fsSL https://get.hy2.sh/)


chain
#######

.. code-block:: text

        应用
        ↓ SOCKS5
        本地 Hysteria 2 Client
        ↓ HY2
        Hysteria Server 1
        ↓ SOCKS5
        Server 1 上的 Hysteria 2 Client
        ↓ HY2
        Hysteria Server 2
        ↓
        Internet


Local  -> Server 1
#################################

client-server1.yaml

.. code-block:: yaml

        server: server1.example.com:443

        auth: server1-password

        tls:
          sni: server1.example.com
          insecure: false

        socks5:
          listen: 127.0.0.1:1080
          disableUDP: false

        lazy: true


启动

.. code-block:: bash

        hysteria client -c client-server1.yaml


测试：

.. code-block:: bash

        curl --proxy socks5h://127.0.0.1:1080 https://ipinfo.json


Server 1 client ->  Server 2
################################


/etc/hysteria/server2-client.yaml：

.. code-block:: yaml

        server: server2.example.com:443

        auth: server2-password

        tls:
          sni: server2.example.com
          insecure: false

        socks5:
          listen: 127.0.0.1:10808
          disableUDP: false

        lazy: false

启动

.. code-block:: bash

        hysteria client -c /etc/hysteria/server2-client.yaml

测试

.. code-block:: bash

        curl --proxy socks5h://127.0.0.1:10808 https://api.ipify.org


Server 1 
##############

/etc/hysteria/server1.yaml

.. code-block:: yaml

        listen: :443

        tls:
          cert: /etc/hysteria/server.crt
          key: /etc/hysteria/server.key

        auth:
          type: password
          password: server1-password

        outbounds:
        - name: server2
          type: socks5
          socks5:
          addr: 127.0.0.1:10808


启动

.. code-block:: bash

        hysteria server -c /etc/hysteria/server1.yaml

Server 2
##################################


/etc/hysteria/server2.yaml

.. code-block:: yaml

        listen: :443

        tls:
          cert: /etc/hysteria/server.crt
          key: /etc/hysteria/server.key

        auth:
          type: password
          password: server2-password

        outbounds:
        - name: direct
          type: direct


overtls
----------

https://github.com/ShadowsocksR-Live/overtls

准备公私钥对，申请证书，例如letsencrypt

.. code-block::

        server domain: xxx.example.com
        server host: xxx.xxx.xxx.xxx
        server port: 443
        server 私钥：/home/someusr/.cert/privkey.pem
        server 证书链: /home/someusr/.cert/fullchain.pem
        tunnel path: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
        client port: 8888

生成 config.json

.. code-block::

        {
            "tunnel_path": "/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/",

            "server_settings": {
                "certfile": "/home/someusr/.cert/fullchain.pem",
                "keyfile": "/home/someusr/.cert/privkey.pem",
                "forward_addr": "http://127.0.0.1:80",
                "listen_host": "0.0.0.0",
                "listen_port": 443
            },

            "client_settings": {
                "server_host": "xxx.xxx.xxx.xxx",
                "server_port": 443,
                "server_domain": "xxx.example.com",
                "listen_host": "127.0.0.1",
                "listen_port": 8888
            }
        }


运行

.. code-block::

        overtls -r server -c config.json

        overtls -r client -c config.json

xray chain proxy
-------------------

https://github.com/Hidden-Node/proxy-builder

https://github.com/XTLS/Xray-core

xray client -- proxy a -- proxy b -- internet


v2ray
--------

apt install v2ray

.. code-block::

        host: xxx.example.com
        port: 443
        私钥：/home/someusr/.cert/privkey.pem
        证书链: /home/someusr/.cert/fullchain.pem
        uuid: 66666666-6666-6666-6666-666666666666

配置/usr/local/etc/v2ray/config.json:

.. code-block::

        {
            "log": {
                "loglevel": "info"
            },
            "inbounds": [
                {
                    "port": 443, 
                    "protocol": "vless",
                    "settings": {
                        "clients": [
                            {
                                "id": "66666666-6666-6666-6666-666666666666", 
                    "flow": "xtls-rprx-origin", 
                                "level": 0
                            }
                        ],
                        "decryption": "none",
                        "fallbacks": [
                            {
                                "dest": 80 
                            }
                        ]
                    },
                    "streamSettings": {
                        "network": "tcp",
                        "security": "xtls",
                        "xtlsSettings": {
                            "alpn": [
                                "http/1.2"
                            ],
                            "certificates": [
                                {
                                    "certificateFile": "/home/someusr/.cert/fullchain.pem", 
                                    "keyFile": "/home/someusr/.cert/privkey.pem" 
                                }
                            ]
                        }
                    }
                }
            ],
            "outbounds": [
                {
                    "protocol": "freedom"
                }
            ]
        }

server启动

 systemctl start v2ray  

添加开机启动

 systemctl enable v2ray

client
############

路径:/home/someclient/share/v2ray，

local port: 8888

配置/home/someclient/share/v2ray/config.json

.. code-block::

        {
          "log": {
            "access": "",
            "error": "",
            "loglevel": "warning"
          },
          "inbounds": [
            {
              "port": 8888,
              "listen": "127.0.0.1",
              "protocol": "socks",
              "sniffing": {
                "enabled": true,
                "destOverride": [
                  "http",
                  "tls"
                ]
              },
              "settings": {
                "auth": "noauth",
                "udp": true,
                "ip": null,
                "clients": null
              },
              "streamSettings": null
            }
          ],
          "outbounds": [
            {
              "tag": "proxy",
              "protocol": "vless",
              "settings": {
                "vnext": [
                  {
                    "address": "xxx.example.com",
                    "port": 443,
                    "users": [
                      {
                          "id": "66666666-6666-6666-6666-666666666666",
                          "flow": "xtls-rprx-origin",
                          "level": 0, 
                    "encryption": "none"
                      }
                    ]
                  }
                ],
                "servers": null,
                "response": null
              },
              "streamSettings": {
                "network": "tcp",
                "security": "",
                "tlsSettings": null,
                "tcpSettings": null,
                "kcpSettings": null,
                "wsSettings": null,
                "httpSettings": null,
                "quicSettings": null
              },
              "mux": {
                "enabled": true
              }
            },
            {
              "tag": "direct",
              "protocol": "freedom",
              "settings": {
                "vnext": null,
                "servers": null,
                "response": null
              },
              "streamSettings": null,
              "mux": null
            },
            {
              "tag": "block",
              "protocol": "blackhole",
              "settings": {
                "vnext": null,
                "servers": null,
                "response": {
                  "type": "http"
                }
              },
              "streamSettings": null,
              "mux": null
            }
          ],
          "dns": null,
          "routing": {
            "domainStrategy": "IPIfNonMatch",
            "rules": []
          }
        }


client运行

.. code-block::

        cd /home/someclient/share/v2ray
        v2ray run

trojan
----------

apt install trojan

server
#############

.. code-block::

        host: xxx.example.com
        port: 443
        私钥：/home/someusr/.cert/privkey.pem
        证书链: /home/someusr/.cert/fullchain.pem

配置/usr/local/etc/trojan/config.json

.. code-block::

        {
            "run_type": "server",
            "local_addr": "0.0.0.0",
            "local_port": 443,
            "remote_addr": "127.0.0.1",
            "remote_port": 80,
            "password": [
                "mypasswd"
            ],
            "log_level": 1,
            "ssl": {
                "cert": "/home/someusr/.cert/fullchain.pem",
                "key": "/home/someusr/.cert/privkey.pem",
                "key_password": "",
            "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305",
                "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
                "prefer_server_cipher": true,
                "alpn": [
                    "http/1.2",
                    "http/1.3",
                    "h2"
                ],
                "reuse_session": true,
                "session_ticket": false,
                "session_timeout": 600,
                "plain_http_response": "",
                "curves": "",
                "dhparam": ""
            },
            "tcp": {
                "prefer_ipv4": false,
                "no_delay": true,
                "keep_alive": true,
                "reuse_port": false,
                "fast_open": false,
                "fast_open_qlen": 20
            },
            "mysql": {
                "enabled": false,
                "server_addr": "127.0.0.1",
                "server_port": 3306,
                "database": "trojan",
                "username": "trojan",
                "password": "",
                "key": "",
                "cert": "",
                "ca": ""
            }
        }

启动

        systemctl start trojan

添加开机启动

        systemctl enable trojan

client
############

.. code-block::

        路径:/home/someclient/share/trojan，
        local port: 8888
        cert chain:/home/someclient/share/trojan/fullchain.pem
        priv key:/home/someclient/share/trojan/privkey.pem

配置/home/someclient/share/trojan/config.json

.. code-block::

        {
            "run_type": "client",
            "local_addr": "127.0.0.1",
            "local_port": 8888,
            "remote_addr": "xxx.example.com",
            "remote_port": 443,
            "password": [
                "mypasswd"
            ],
            "log_level": 1,
            "ssl": {
                "cert": "fullchain.pem",
                "key": "privkey.pem",
                "key_password": "",
                "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305",
                "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
                "prefer_server_cipher": true,
                "alpn": [
                    "http/1.2",
                    "http/1.3",
                    "h2"
                ],
                "reuse_session": true,
                "session_ticket": false,
                "session_timeout": 600,
                "plain_http_response": "",
                "curves": "",
                "dhparam": ""
            },
            "tcp": {
                "prefer_ipv4": false,
                "no_delay": true,
                "keep_alive": true,
                "reuse_port": false,
                "fast_open": false,
                "fast_open_qlen": 20
            },
            "mysql": {
                "enabled": false,
                "server_addr": "127.0.0.1",
                "server_port": 3306,
                "database": "trojan",
                "username": "trojan",
                "password": "",
                "key": "",
                "cert": "",
                "ca": ""
            }
        }

启动

.. code-block::

        cd /home/someclient/share/trojan
        trojan -c config.json
