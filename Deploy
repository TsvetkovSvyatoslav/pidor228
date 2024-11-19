#!/bin/bash

# Cleaning
systemctl stop shadowsocks-libev
apt purge -y shadowsocks-libev
apt autoremove -y
rm -rf /usr/bin/v2ray

# Installing
apt update -y
apt install -y curl jq psmisc tar wget xz-utils
cat << EOF > /etc/resolv.conf
nameserver 1.1.1.1
nameserver 1.0.0.1
EOF
systemctl stop ssrxray.service
systemctl disable ssrxray.service
fuser -k 9090/tcp
mkdir /tmp/sstmp
LATEST=$(curl -s https://api.github.com/repos/shadowsocks/shadowsocks-rust/releases/latest | jq -r '.tag_name')
wget -O /tmp/sstmp/ss.tar.gz \
https://github.com/shadowsocks/shadowsocks-rust/releases/download/$LATEST/shadowsocks-$LATEST.x86_64-unknown-linux-gnu.tar.xz
tar -xf /tmp/sstmp/ss.tar.gz -C /tmp/sstmp
install -m 755 /tmp/sstmp/ssserver /usr/bin/ssserver
LATEST=$(curl -s https://api.github.com/repos/teddysun/xray-plugin/releases/latest | jq -r '.tag_name')
wget -O /tmp/sstmp/xray.tar.gz \
https://github.com/teddysun/xray-plugin/releases/download/$LATEST/xray-plugin-linux-amd64-$LATEST.tar.gz
tar -xf /tmp/sstmp/xray.tar.gz -C /tmp/sstmp
install -m 755 /tmp/sstmp/xray-plugin_linux_amd64 /usr/bin/xray
rm -rf /tmp/sstmp
IPADDR=$(ip addr show |grep 'inet '|grep -v 127.0.0.1 |awk '{print $2}'| cut -d/ -f1)
PASSWORD=$(cat /proc/sys/kernel/random/uuid)
ENCRYPTION=chacha20-ietf-poly1305
mkdir /etc/shadowsocks-rust
cat << EOF > /etc/shadowsocks-rust/config.json
{
    "server":"$IPADDR",
    "server_port":9090,
    "password":"$PASSWORD",
    "method":"$ENCRYPTION",
    "timeout":60,
    "mode": "tcp_and_udp",
    "fast_open":true,
    "reuse_port":true,
    "no_delay":true,
    "plugin": "/usr/bin/xray",
    "plugin_opts":"server;mode=grpc;loglevel=none"
}
EOF
cat << EOF > /etc/systemd/system/ssrxray.service
[Unit]
Description=Shadowsocks-Xray
After=network.target nss-lookup.target
[Service]
Type=simple
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks-rust/config.json
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
systemctl enable ssrxray.service
systemctl start ssrxray.service
systemctl restart ssrxray.service
printf "\033[37;1;41mss://$(echo -n $ENCRYPTION:$PASSWORD | base64 -w 0)@$IPADDR:9090/?plugin=xray%%3bmode=grpc\033[0m\n"
