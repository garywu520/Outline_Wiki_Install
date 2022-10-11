#### Clone项目

```Bash
cd /opt/
git clone https://github.com/vicalloy/outline-docker-compose.git
cd outline-docker-compose
cp scripts/config.sh.sample scripts/config.sh
```

#### 修改配置

cat scripts/config.sh

```python
#这里修改为最终访问的域名地址
#部署到公网-示例：URL=https://xxx.xxx.com
URL=https://xxx.xxx.com
TIME_ZONE=Asia/Hong_Kong

#ALLOW DOMAIN
ALLOWED_DOMAINS=xxx.xxx.com

# Docker-Nginx
HTTP_IP=127.0.0.1
HTTP_PORT_IP=8888

#查看这2个组件的最新版本号填入即可使用最新版本
OUTLINE_VERSION=0.66.1
POSTGRES_VERSION=14.5
```

#### Make并启动

```bash
cd /opt/outline-docker-compose/
make install

#此过程将创建超级管理帐号和密码
```

#### Nginx反向代理本机Outline配置

```
server {
    listen 80;
    listen 443 ssl http2;
    server_name xxx.xxx.com;

    ssl_certificate /etc/nginx/ssl/xxx.xxx.com-full.crt;
    ssl_certificate_key /etc/nginx/ssl/xxx.xxx.com.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers TLS13-AES-256-GCM-SHA384:TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-128-CCM-8-SHA256:TLS13-AES-128-CCM-SHA256:EECDH+CHACHA20:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_timeout 10m;
    ssl_session_cache builtin:1000 shared:SSL:10m;
    ssl_buffer_size 1400;
  
    add_header Strict-Transport-Security "max-age=31536000; preload";
    add_header HTTPS "on";
    #修改上传大小
    client_max_body_size 1024m;

    # HTTPS 301跳转
    if ($ssl_protocol = "") {
        return 301 https://$server_name$request_uri;
    }

     location / {
        ssi on;
        proxy_pass http://localhost:8888/;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

     location /realtime {
        proxy_pass http://localhost:8888/realtime;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $http_connection;
        proxy_read_timeout 86400;
    }
}
```



#### Web访问并登陆：

- Outline WIKI: https://xxx.xxx.com

- OIDC帐号后台：https://xxx.xxx.com/uc/admin/


<br>
<br>

#### 日常管理

```Bash
$ cd /opt/outline-docker-compose
$ docker-compose restart     #重启相关实例
$ docker ps -a   #查看所有容器(包括已停止或退出的容器)
$ docker-compose down #停服
$ docker-compose up -d #启动服务
```

<br>
<br>


#### 关于Outline版本升级

- 备份数据导出到本地

- 搜索Outline Realease查看官方最新版本，如0.66.1，将其替换到文件 scripts/config.sh的OUTLINE_VERSION字段

- 执行重构命令

  ```
  make stop
  make clean 
  make clean-data
  ```

- 导入备份的数据

<br>
<br>

###### 基于项目更新: https://github.com/vicalloy/outline-docker-compose
