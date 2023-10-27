---

tags: [let's encrypt,https,certbot]
categories: [运维]

---

## let's encrypt YYDS
[Let's Encrypt](https://letsencrypt.org/) 是一个免费、自动化和开放的证书颁发机构 (CA)。 Let's Encrypt 颁发的证书如今受到大多数浏览器的信任，包括旧版浏览器，例如 Windows XP SP3 上的 Internet Explorer。 此外，Let's Encrypt 可以完全自动化证书的颁发和更新。

## 它如何工作
在颁发证书之前，Let's Encrypt 会验证您域的所有权。 在您的主机上运行的 Let’s Encrypt 客户端会创建一个包含所需信息的临时文件（令牌）。 Let’s Encrypt 验证服务器然后发出 HTTP 请求来检索文件并验证令牌，这会验证您的域的 DNS 记录是否解析为运行 Let’s Encrypt 客户端的服务器。


## 如何使用
它提供一些主流web服务器的插件，比如`apache`、`nginx`等,本文以最流行的`nginx`为例 
### 前置条件  

1. 拥有一台服务器
2. 安装完毕nginx，并配置好相应的config
3. 拥有或控制证书的注册域名。 没有注册域名，可以使用域名注册商，例如 GoDaddy 或 dnsexit。
4. 已完成域名的DNS解析配置

### 下载客户端
以Ubuntu 16.04为例
```bash
$ apt-get update
$ sudo apt-get install certbot
$ apt-get install python-certbot-nginx
```
nginx简单配置
```javascript
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html;
    server_name blog.justwannarun.com;
}
```
保存文件并重启
```bash
nginx -t && nginx -s reload
```
使用客户端进行证书生成
```bash
sudo certbot --nginx -d justwannarun.com -d blog.justwannarun.com
```

在生成过程中遇到了一个小插曲 **DNS-01 challenge**
因为它要通过DNS验证你是否控制了该域名，所以需要在DNS中配置它给出的token
![DNS.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1636641637338-1efd221b-65f2-4fa4-beb1-b622d90c7eaa.png#clientId=u1d2030fa-8470-4&from=ui&id=u5a6f6bf1&originHeight=268&originWidth=1986&originalType=binary&ratio=1&rotation=0&showTitle=false&size=45503&status=done&style=none&taskId=ub4d2068e-220e-4ee3-8737-6b904b46c12&title=)
配置完之后，再进行确认操作，等待提示成功，就说明成功了（此处废话学问）

此时nginx配置会被修改
![certbot-nginx.png](https://cdn.nlark.com/yuque/0/2021/png/1512483/1636641776856-8e83e924-8894-46df-a545-bf7419acdb60.png#clientId=u1d2030fa-8470-4&from=ui&id=u6fb67512&originHeight=1140&originWidth=1526&originalType=binary&ratio=1&rotation=0&showTitle=false&size=191589&status=done&style=none&taskId=uc81f611e-c779-4ad9-b82b-577a5e9100e&title=)
## 自动续期
证书有效期一般两个月，所以我们还要配置下自动续期
```bash
$ crontab -e
```
打开这个定时任务文件，添加一行
```bash
0 12 * * * /usr/bin/certbot renew --quiet
```
这样每12小时就会执行一次续期，自动化起来了，完美

## 结语
感谢let's encrypt，有机会一定会捐赠哒~（手动狗头）
