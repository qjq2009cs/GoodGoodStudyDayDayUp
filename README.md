开启root登陆

sudo su

cd /root

修改authorized_keys文件（即ssh证书）

vi .ssh/authorized_keys

把ssh-rsa之前的文件都删除掉.

编辑ssh配置文件

nano /etc/ssh/sshd_config

找到PermitRootLogin, 把前面的#去掉 改成下面这样

PermitRootLogin yes

service sshd restart

ctrl+x 保存退出 选择y 然后回车

然后

reboot

重启服务器。就可以使用root用户名配合秘钥登入了

安装BBRplus

apt-get -y install ca-certificates

wget "https://github.com/chiakge/Linux-NetSpeed/raw/master/tcp.sh" && chmod +x tcp.sh && ./tcp.sh

centos专用BBRplus
wget "https://github.com/a8866051/bbrplus/raw/master/ok_bbrplus_centos.sh" && chmod +x ok_bbrplus_centos.sh && ./ok_bbrplus_centos.sh


开启防火墙

iptables -P INPUT ACCEPT

iptables -P FORWARD ACCEPT

iptables -P OUTPUT ACCEPT

iptables -F

apt-get purge netfilter-persistent

reboot 重启


准备工作

1、VPS 准备 Debian 9+

2、域名解析到VPS 的IP 上

申请SSL证书

apt update -y

apt install -y curl

apt install -y socat

curl https://get.acme.sh | sh

~/.acme.sh/acme.sh --register-account -m xxx@xxx.xxx

#更换真实邮箱成功率更高

~/.acme.sh/acme.sh --issue -d xxxx.xxxx.xxx --standalone

更换你的解析域名

~/.acme.sh/acme.sh --installcert -d xxxx.xxxx.xxx --key-file /root/private.key --fullchain-file /root/cert.crt

更换你的解析域名，此步完成后会在VPS root 目录下看到
证书公钥/root/cert.crt 及 密钥文件/root/private.key

安装BBR (视频中说此代码没用 但经网友测试貌似是有作用的！大家可以安装一下)
但别随便乱用网速搜的BBR脚本，很容易搞崩！！！

echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf

echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf

sysctl -p

lsmod | grep bbr

#打完最后一步，应看到 20480 或 16384 说明BBR 开启成功

安装X-ui面板

bash <(curl -Ls https://raw.githubusercontent.com/vaxilu/x-ui/master/install.sh)

私钥路径：/root/private.key

公钥路径：/root/cert.crt

x-ui模板：

{
  "api": {
    "services": [
      "HandlerService",
      "LoggerService",
      "StatsService"
    ],
    "tag": "api"
  },
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 62789,
      "protocol": "dokodemo-door",
      "settings": {
        "address": "127.0.0.1"
      },
      "tag": "api"
    }
  ],
  "outbounds": [
      {
      "protocol": "freedom",
      "settings": {
         "domainStrategy": "UseIPv6"         
       },
      "tag": "IP-V6"
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "policy": {
    "system": {
      "statsInboundDownlink": true,
      "statsInboundUplink": true
    }
  },
  "routing": {
    "rules": [
      {
        "type":"field",
        "domain": [
          "geosite:netflix"
        ],
        "inboundTag":  [
          "all-in"
         ],
        "outboundTag": "IP-V6"
      },
      {
        "inboundTag": [
          "api"
        ],
        "outboundTag": "api",
        "type": "field"
      },
      {
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "blocked",
        "type": "field"
      },
      {
        "outboundTag": "blocked",
        "protocol": [
          "bittorrent"
        ],
        "type": "field"
      }
    ]
  },
  "stats": {}
}


debain系统镜像源

deb http://deb.debian.org/debian/ buster main
deb-src http://deb.debian.org/debian/ buster main
deb http://security.debian.org/debian-security buster/updates main
deb-src http://security.debian.org/debian-security buster/updates main
deb http://deb.debian.org/debian/ buster-updates main
deb-src http://deb.debian.org/debian/ buster-updates main

解锁视频限制：wget -N https://raw.githubusercontent.com/fscarmen/warp/main/menu.sh && bash menu.sh d

