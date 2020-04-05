---
title: pacificrack 搭建 v2ray
date: 2020-03-21 21:30:37
tags: [linux]
description: <center>搭建科学上网的记录</center>
---

## 使用感受

太卡了，别买。先用tcping测下速度。但是服务商提供的IP基本速度都是很快的，没有什么参考意义，这时候要去找其他用户自己购买的vps来测速，这样比较贴近实际效果。

## 登录

付费之后vps不能直接使用，查看订单状态还在等待中，成功之后会发送标题为**New Virtual Server Is Ready**邮件，使用**Server Details**中的IP地址和密码进行登录，这里要使用root账户，hostname访问ssh失败

```
ssh root@IP地址
```

## 时间校准

V2Ray的验证方式包含时间，就算配置没有任何问题，如果时间不正确，也无法连接 V2Ray 服务器的，服务器会认为你这是不合法的请求

```
# date -R
Sun, 15 Mar 2020 22:13:34 -0400
# cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# date -R
Mon, 16 Mar 2020 10:14:23 +0800
```

## 脚本安装

使用脚本安装，该脚本由 V2Ray 官方提供。该脚本仅可以在 Debian 系列或者支持 Systemd 的 Linux 操作系统使用

```
bash <(curl -L -s https://install.direct/go.sh)
```

生成的PORT,UUID是后面要用到的，可以在`/etc/v2ray/config.json`修改

## 启动命令

```
## 启动
systemctl start v2ray

## 停止
systemctl stop v2ray

## 重启
systemctl restart v2ray

## 开机自启
systemctl enable v2ray
```

## 添加用户

```json
{
  "inbounds": [{
    "port": 19645,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "8481050a-ad35-4e2e-90b4-bcd1440fa71e",
          "level": 1,
          "alterId": 64
        }
      ],"detour": {
          "to": "dynamicPort"
        }
      }
    },
    { //新增加的用户
    "port": 19646,//修改
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "cd3eb57a-8395-43c1-8f55-a50b6eb03179",//修改
          "level": 1,
          "alterId": 65//修改
        }
      ],"detour": {
          "to": "dynamicPort"
        }
      }
    },//新增结束
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": ["geoip:private"],
        "outboundTag": "blocked"
      }
    ]
  }
}
```



## WebSocket+TLS+Web

参考https://toutyrater.github.io/advanced/wss_and_web.html

其中nginx的配置在/etc/nginx/conf.d/v2ray.conf

## 参考

https://www.v2ray.com/awesome/tools.html