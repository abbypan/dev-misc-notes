proxy
=======

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


xray chain proxy
-------------------

https://github.com/Hidden-Node/proxy-builder

https://github.com/XTLS/Xray-core

xray client -- proxy a -- proxy b -- internet

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
