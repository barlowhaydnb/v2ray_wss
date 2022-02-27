# V2ray_VLESS_WebSocket_TLS_CaddyV2
V2ray最新版本，VLESS_WebSocket_TLS模式，CaddyV2前置解除TLS和path

# 一键执行
```
bash <(curl -L https://github.com/crazypeace/V2ray_VLESS_WebSocket_TLS_CaddyV2/raw/main/install.sh)
```

脚本中很大部分都是在校验用户的输入。其实照着下面的内容自己配置就行了。


# 安装V2ray最新版本
```
bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
```
# 安装CaddyV2最新版本

```
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo apt-key add -
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

# 配置 /usr/local/etc/v2ray/config.json
```
{ // VLESS + WebSocket + TLS
    "log": {
        "access": "/var/log/v2ray/access.log",
        "error": "/var/log/v2ray/error.log",
        "loglevel": "warning"
    },
    "inbounds": [
        {
            "port": 你的v2ray内部端口,             // ***
            "protocol": "vless",
            "settings": {
                "clients": [
                    {
                        "id": "你的v2rayID",             // ***
                        "level": 1,
                        "alterId": 0
                    }
                ],
                "decryption": "none"
            },
            "listen": "127.0.0.1",
            "streamSettings": {
                "network": "ws"
            },
            "sniffing": {
                "enabled": true,
                "destOverride": [
                    "http",
                    "tls"
                ]
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "freedom",
            "settings": {
                "domainStrategy": "UseIP"
            },
            "tag": "direct"
        },
        {
            "protocol": "blackhole",
            "settings": {},
            "tag": "blocked"
            },
        {
            "protocol": "mtproto",
            "settings": {},
            "tag": "tg-out"
        }
    ],
    "dns": {
        "servers": [
            "https+local://8.8.8.8/dns-query",
            "8.8.8.8",
            "1.1.1.1",
            "localhost"
        ]
    },
    "routing": {
        "domainStrategy": "IPOnDemand",
        "rules": [
            {
                "type": "field",
                "ip": [
                    "0.0.0.0/8",
                    "10.0.0.0/8",
                    "100.64.0.0/10",
                    "127.0.0.0/8",
                    "169.254.0.0/16",
                    "172.16.0.0/12",
                    "192.0.0.0/24",
                    "192.0.2.0/24",
                    "192.168.0.0/16",
                    "198.18.0.0/15",
                    "198.51.100.0/24",
                    "203.0.113.0/24",
                    "::1/128",
                    "fc00::/7",
                    "fe80::/10"
                ],
                "outboundTag": "blocked"
            },
            {
                "type": "field",
                "inboundTag": ["tg-in"],
                "outboundTag": "tg-out"
            },
            {
                "type": "field",
                "protocol": [
                    "bittorrent"
                ],
                "outboundTag": "blocked"
            }
        ]
    },
    "transport": {
        "kcpSettings": {
            "uplinkCapacity": 100,
            "downlinkCapacity": 100,
            "congestion": true
        }
    }
}
```

# 配置 /etc/caddy/Caddyfile
```
你的域名
{
    tls Y3JhenlwZWFjZQ@gmail.com {
        on_demand
    }
    encode gzip

    handle_path /分流path {
        reverse_proxy localhost:你的v2ray内部端口
    }
    handle {
        reverse_proxy https://你反代伪装的网站 {
            header_up Host {upstream_hostport}
            header_up X-Forwarded-Host {host}
        }
    }
}
```