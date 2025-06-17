

## 安装gams
1,/opt/gams
2,wget https://d37drm4t2jghv5.cloudfront.net/distributions/45.4.0/linux/linux_x64_64_sfx.exe
3,vi ~/.bashrc 添加path，添加alias
4，source ～/.bashrc

gams --version      45.4.0


## 安装 anaconda3
1，/opt/anaconda
2，wget https://repo.anaconda.com/archive/Anaconda3-2023.09-0-Linux-x86_64.sh
3，安装 sh Anaconda3-2023.09-0-Linux-x86_64.sh -- python3


jupyter notebook --allow-root


#找到配置文件的位置，若没有配置文件可使用以下命令创建
jupyter notebook --generate-config
#编辑配置文件
vi /your_path/.jupyter/jupyter_notebook_config.py
#找到以下配置，去除代码前面的 # ，并将值设置为 True
c.NotebookApp.allow_remote_access=True
#设置允许所有IP可访问，localhost仅支持本地访问
c.NotebookApp.ip='*'
#指定访问端口
c.NotebookApp.port = 8888
#重新启动
jupyter notebook


##

查看防火墙开启状态:
systemctl status firewalld
开启防火墙
systemctl start firewalld.service
关闭防火墙
systemctl stop firewalld.service
查看防火墙允许开放的端口
firewall-cmd --list-ports

防火墙对8091端口进行开放
firewall-cmd --add-port=8091/tcp --permanent
开启之后需要重新加载使配置生效
firewall-cmd --reload


python3 -m pip install ixmp
python3 -m pip install message-ix


conda install -c conda-forge ixmp



conda activate message_env
jupyter notebook --allow-root


