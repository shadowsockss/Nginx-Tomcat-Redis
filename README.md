# Nginx-Tomcat-Redis
Nginx+Tomcat+Redis实现负载均衡、资源分离、session共享


CentOS安装Nginx 
参考 http://centoscn.com/CentosServer/www/2013/0910/1593.html
CentOS安装Tomcat
参考 http://blog.csdn.net/zhuying_linux/article/details/6583096
CentOS安装Redis
参考 http://www.cnblogs.com/zhuhongbao/archive/2013/06/04/3117997.html
多个Tomcat负载均衡实例：可在服务器上复制出多个Tomcat分别修改Tomcat的
http访问端口（默认为8080端口）
 
Shutdown端口（默认为8005端口）
 
JVM启动端口（默认为8009端口）
 
1、Nginx实现多Tomcat负载均衡
Tomcat服务
192.168.1.177:8001
192.168.1.177:8002
192.168.1.177:8003
Nginx配置
upstream mytomcats {  
server 192.168.1.177:8001;  
server 192.168.1.177:8002;  
server 192.168.1.177:8003;  
}
server {  
listen 80;  
server_name www.iu14.com; 
location ~* \.(jpg|gif|png|swf|flv|wma|wmv|asf|mp3|mmf|zip|rar)$ {  
        root /web/www/html/;  
}  
location / {  
        proxy_pass http://mytomcats;  
        proxy_redirect off;  
        proxy_set_header Host $host;  
        proxy_set_header X-Real-IP $remote_addr;  
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
        client_max_body_size 10m;  
        client_body_buffer_size 128k;  
        proxy_connect_timeout 90;  
        proxy_send_timeout 90;  
        proxy_read_timeout 90;  
        proxy_buffer_size 4k;  
        proxy_buffers 4 32k;  
        proxy_busy_buffers_size 64k;  
        proxy_temp_file_write_size 64k; 
}
}
upstream指定负载均衡组，指定其Tomcat成员
location ~* \.(jpg|gif|……实现了静态资源分离。ps：在location指令使用正则表达式后再用alias指令，Nginx是不支持的。
2、Nginx实现静态资源分离
Tomcat服务
192.168.1.177:8000
Nginx配置
server {  
listen 80;  
server_name www.iu14.com;  
root /web/www/html; 
location /img/ {  
alias /web/www/html/img/;  
}
location ~ (\.jsp)|(\.do)$ {  
proxy_pass http://192.168.1.177:8000;  
proxy_redirect off;  
        proxy_set_header Host $host;  
        proxy_set_header X-Real-IP $remote_addr;  
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
        client_max_body_size 10m;  
        client_body_buffer_size 128k;  
        proxy_connect_timeout 90;  
proxy_send_timeout 90;  
proxy_read_timeout 90;  
        proxy_buffer_size 4k;  
        proxy_buffers 4 32k;  
        proxy_busy_buffers_size 64k;  
        proxy_temp_file_write_size 64k;  
}   
}
第一个location指令将/web/www/html/img/目录下的静态文件交给Nginx来完成。最后一个location指令将所有以.jsp、.do结尾的文件都交给Tomcat服务器的8080端口来处理。
3、Nginx+Tomcat+Redis实现session共享
Redis服务
192.168.1.178:6379
Tomcat服务
192.168.1.177:8001
192.168.1.177:8002
192.168.1.177:8003
Nginx服务
192.168.1.179
配置Tomcat让其session保存到redis上，在context.xml配置(Value标签一定要在Manager标签前面)：
 
配置Nginx
upstream mytomcats {  
server 192.168.1.177:8001;  
server 192.168.1.177:8002;  
server 192.168.1.177:8003;  
}
log_format www_iu14_com '$remote_addr - $remote_user [$time_local] $request ' '"$status" $body_bytes_sent "$http_referer"'  '"$http_user_agent" "$http_x_forwarded_for"';  
server {
listen  80;  
server_name www.iu14.com;   
    location / {  
        proxy_pass http:// mytomcats;  
        proxy_set_header Host $host;  
        proxy_set_header X-Real-IP $remote_addr;  
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
}  
access_log /usr/tmp/logs/redis.iu14.log www_iu14_com;  
}  
依次启动Redis、Tomcat、Nginx，访问Nginx

