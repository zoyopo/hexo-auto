tags: [微前端,iframe,nginx]
categories: [前端]

---

## 背景
因为产品的合规要求，需要在用户购买前将条款等内容弹出来告知。
但是条款内容是已开发过的页面，所以可以使用iframe来复用，提高开发效率。


## 问题

- 但是开发过程中，产品设计对于复用的理解和开发不太一样。他们会要求和原页面不一样的东西。
- 而且iframe会加载一些此页面不需要的资源，拖慢页面速度。

所以，笔者认为，在时间充裕的条件下，可以考虑页面组件化复用，将会更合理。

## 状况
因为不光是本网站调用自己的页面，还涉及到公司的另一个网站要调用此网站页面，所以最大的一个问题应该是不同域之间的兼容问题。

### 跨域问题
浏览器为了安全，禁止了不同域之间的访问。
hack的办法应该很多，我们这里使用了 `nginx` 进行域名无刷新转发

```nginx
 upstream p_resin_server{
        server 127.0.0.1:9090;
 }

server {
    listen 80;
    server_name www.b.cn;
    fastcgi_buffer_size 128k;
    fastcgi_buffers 8 128k; 
    fastcgi_busy_buffers_size 256k;
    fastcgi_temp_file_write_size 256k;
    client_body_buffer_size 128k;
    client_max_body_size 10m;	

    location /w/yyy.do {
      proxy_set_header Host "www.a.cn"; 
      proxy_set_header X-Real-IP $remote_addr; 
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_pass http://p_resin_server;  
   }
   location /w/xxx.do {
      proxy_set_header Host "www.a.cn"; 
      proxy_set_header X-Real-IP $remote_addr; 
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_pass http://p_resin_server;           
   }
  
}
```
### 资源加载问题

iframe里的相对路径的资源，会相对于主域名，显然主域里是不存在这些资源的。
这里我们可以在iframe里得页面 `head` 里加上 `<base href="www.a.cn">`
来对路径的指向进行纠正。
