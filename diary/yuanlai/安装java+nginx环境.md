

>安装依赖：yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel

>mkdir nginx
>wget http://nginx.org/download/nginx-1.26.2.tar.gz

>查看http://nginx.org/download/ 找到相应版本

//进入nginx目录
cd /usr/local/nginx
//进入目录
cd nginx-1.13.7
//执行命令 考虑到后续安装ssl证书 添加两个模块
./configure --with-http_stub_status_module --with-http_ssl_module
//执行make命令
make
//执行make install命令
make install

启动nginx服务
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

# 打开配置文件
vi /usr/local/nginx/conf/nginx.conf