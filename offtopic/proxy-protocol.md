# 常见应用配置 Proxy Protocol

配置 Proxy Protocol 可以让您穿透的本地服务获取到客户端真实 IP。

在进行下列配置前，请先阅读 [获取真实 IP](/bestpractice/realip#proxy-protocol) 并修改隧道配置，否则可能造成 **隧道完全不可用**。

## Web 服务器

### Nginx

在需要启用 Proxy Protocol 的 `server` 块找到 `listen` 字段，并在尾部（分号前面）添加用空格分开的 `proxy_protocol` 即可。举个例子：

<div style="display: flex;overflow: auto">
<div style="flex:1;padding-right: 8px">
修改前：

```nginx
http {
    server {
        listen 80;
        listen 443 ssl;
        # ...
    }
}
```
</div>
<div style="flex:1;padding-left: 8px">
修改后：

```nginx
http {
    server {
        listen 80 proxy_protocol;
        listen 443 ssl proxy_protocol;
        # ...
    }
}
```
</div>
</div>

配置完成后，您可以通过 `$proxy_protocol_addr` 变量获取到真实 IP。一个常见的做法是将该变量设置为一个 HTTP 头以便后续应用使用：

```nginx
server {
    listen 80 proxy_protocol;
    listen 443 ssl proxy_protocol;

    # 反向代理
    location ... {
        proxy_set_header X-Real-IP $proxy_protocol_addr;
        proxy_set_header X-Forwarded-For $proxy_protocol_addr;

        # 原有内容
        proxy_pass ...;
    }

    # FastCGI
    location ... {
        fastcgi_param HTTP_X_REAL_IP $proxy_protocol_addr;
        fastcgi_param HTTP_X_FORWARDED_FOR $proxy_protocol_addr;

        # 原有内容
        fastcgi_pass ...;
    }
}
```

现在，您可以通过 `X-Forwarded-For` 和 `X-Real-IP` 两个请求头获取真实 IP 了。

### Apache

如果您的 Apache 版本 **>= 2.4.30**，启用 [mod_remoteip](https://httpd.apache.org/docs/current/mod/mod_remoteip.html#remoteipproxyprotocol) 模块后只需在 Apache 配置文件的对应 `VirtualHost` 中添加 `RemoteIPProxyProtocol On` 即可。

```apache
<VirtualHost *:80>
    ServerName ...

    # 新增此行
    RemoteIPProxyProtocol On
</VirtualHost>
```

顺便一提，您可以使用 `a2enmod remoteip` 命令启用 **mod_remoteip** 模块。

如果您使用的 **Ubuntu 18.04** 和 **CentOS 7** 正大步迈向 EOL 以至于源中没有 Apache 2.4.30 以上的版本，您可以考虑升级系统、自己编译或是使用 [mod-proxy-protocol](https://github.com/roadrunner2/mod-proxy-protocol/) 模块。具体配置方式请参考 README。

## Minecraft 服务器

### BungeeCord / Waterfall

请参考 [官方文档](https://www.spigotmc.org/wiki/bungeecord-configuration-guide/) 进行配置，您可以搜索 `proxy_protocol` 找到对应内容（在页面底部）。

```yaml
listeners:
- query_port: 25577
  motd: '&1Another Bungee server'

  # 新增此行
  proxy_protocol: true
```

如果您想同时允许 frp 和直接连接，请使用 [这个插件](https://github.com/andylizi/bc-haproxy-detector)，配置指南见 [MCBBS 帖子](https://www.mcbbs.net/thread-1111852-1-1.html)。

### 其他服务端

STFW。