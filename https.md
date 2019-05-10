二，域名解析到服务器
在阿里云控制台-产品与服务-云解析DNS-找到需要解析的域名点“解析”，进入解析页面后选择【添加解析】按钮会弹出如下页面：

主机记录这里选择@，记录值就是服务器ip地址，确认。



三，申请ca证书
在阿里云控制台-产品与服务-安全(云盾)-CA证书服务(数据安全)，点击购买证书，



选择“免费版DV SSL”，点击立即购买:



然后点去支付:



最后确认支付:



就会回到管理界面：



点击“补全”，输入要解析的域名，点下一步：

说明：因为我们这里申请的是开发版免费证书，所以一个证书仅支持一个域名认证，不支持通配符。



等待几分钟，证书状态变为“已签发”后，证书就申请成功了。

四，下载证书
列表中找到已签发的证书，下载：



进入下载页面，找到ngin页签中nginx配置信息，并“下载证书 for Nginx”：



记录以下内容，为了一会儿配置nginx用：



下载的文件有两个：

1，214292799730473.pem

2，214292799730473.key

五，服务器安装，配置nginx
登录到服务器：

$ apt-get update // 更新软件
$ apt-get install nginx // 安装nginx
六，配置ca证书
1，nginx的安装目录为：/etc/nginx/。进入目录，增加cert/文件夹，把刚刚下载的两个文件上传到cert/文件夹中。

2，在/etc/nginx/sites-enabled/下，增加bjubi.com文件。内容如下：

说明：下面的配置是对443端口和80端口进行监听，443端口要启用ssl。监听443端口的server配置可以仿照上面ca认证页面的nginx配置示例进行配置。

root节点笔者创建了一个bjubi.com/的文件夹，专门存放来自这个域名的请求以示区分。

bjubi.com/文件夹下增加一个index.html文件，里面仅仅写了一行<h1>welcome。

复制代码
server {
    listen 443 ssl;
    server_name bjubi.com; // 你的域名
    ssl on;
    root /var/www/bjubi.com; // 前台文件存放文件夹，可改成别的
    index index.html index.htm;// 上面配置的文件夹里面的index.html
    ssl_certificate  cert/214292799730473.pem;// 改成你的证书的名字
    ssl_certificate_key cert/214292799730473.key;// 你的证书的名字
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    location / {
        index index.html index.htm;
    }
}

server {
    listen 80;
    server_name bjubi.com;// 你的域名
    rewrite ^(.*)$ https://$host$1 permanent;// 把http的域名请求转成https
}

https 访问php文件需要增加配置：

location ~ \.php$ {
    root /data/wwwroot/main;
    fastcgi_pass unix:/dev/shm/php-cgi.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}

配置完成后，检查一下nginx配置文件是否可用，有successful表示可用。

$ nginx -t // 检查nginx配置文件
配置正确后，重新加载配置文件使配置生效：

$ nginx -s reload // 使配置生效
至此，nginx的https访问就完成了，并且通过rewrite方式把所有http请求也转成了https请求，更加安全。

如需重启nginx，用以下命令：

$ service nginx stop // 停止
$ service nginx start // 启动
$ service nginx restart // 重启
