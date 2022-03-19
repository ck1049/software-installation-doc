##### rabbitmq3.9.13安装配置（需要centOs8.x或者Alibaba cloud linux 3.x环境安装配置）

​	该文档基于csdn用户"[伊格尼斯](https://blog.csdn.net/gfk3009?type=blog)"发布教程编写

- ###### 1、安装wxWidgets环境

  ​		由于rabbitmq是erlang语言编写的，安装rabbitmq之前，需要先安装erlang，安装erlang之前需要安装wxWidgets，安装wxWidgets之前需要安装gtk2-devel与binutils-devel

  1. `yum update #先更新系统`

  2. `yum -y install gtk2-devel binutils-devel #安装wxWidgets相关依赖`

  3. 由于erlang依赖wxWidgets，我们下载下载wxWidgets安装包，wxWidgets版本为2.8.4或更高版本，这里我们下载最新版本3.1.5版本

     [官网下载地址](http://www.wxwidgets.org/downloads/)

     ```
     wget https://github.com/wxWidgets/wxWidgets/releases/download/v3.1.5/wxWidgets-3.1.5.tar.bz2
     ```

  4. `mkdir -p /usr/local/wxWidgets #创建wxWidgets的安装路径文件夹`

  5. `tar -xvf wxWidgets-3.1.5.tar.bz2 #解压wxWidgets`

  6. `cd wxWidgets-3.1.5/ #进入wxWidgets目录`

  7. ```
     ./configure --with-regex=builtin --with-gtk --enable-unicode --disable-shared --prefix=/usr/local/wxWidgets #编译wxWidgets`
     ```

  8. `make && make install #开始安装，过程非常久可能要几十分钟甚至更久，耐心等待`

  9. `cd /etc/ld.so.conf.d/ #准备设置其动态库`

  10. `touch wxWidgets.conf #创建文件`

  11. `vim wxWidgets.conf #编辑文件，增加 /usr/local/lib`

  12. `ldconfig #重新加载动态库配置信息`

  13. `vim /etc/profile #配置wxWidgets环境变量`

  14. 文件末尾添加以下内容，保存退出

      ```
      export WXPATH=/usr/local/wxWidgets/ 
      export PATH=$WXPATH/bin:$PATH
      ```

  15. `source /etc/profile #使环境变量刷新并生效`

  16. `wx-config --version #查看wx版本号`

- ###### 2、[安装java开发环境](https://www.cnblogs.com/shenyuanhaojie/p/15744357.html)

- ###### 3、开始安装 erlang

  1. 开始安装erlang相关依赖项

     ```
     1. `yum install -y epel-release`
     2. yum install -y make gcc gcc-c++ m4 openssl openssl-devel ncurses-devel unixODBC unixODBC-devel java java-devel
     ```

  2. [查看rabbitmq、erlang版本支持](https://www.rabbitmq.com/which-erlang.html)

  3. [官网下载erlang包](https://www.erlang.org/downloads)

  4. `tar -xvf otp_src_24.3.2.tar.gz -C /opt #对erlang压缩包进行解压至opt目录下`

  5. `mkdir -p /usr/local/erlang #创建erlang目录`

  6. `cd /opt/otp_src_24.3.2/ #进入解压目录`

  7. `/opt/otp_src_24.3.2/configure --prefix=/usr/local/erlang #开始编译`

  8. `make && make install #安装erlang，需要较长时间`

  9. `vim /etc/profile #修改环境变量`

  10. 文件末尾添加以下内容，保存退出

      ```
      export ERLPATH=/usr/local/erlang 
      export PATH=$ERLPATH/bin:$PATH
      ```

  11. `source /etc/profile #使环境变量刷新并生效`

  12. `erl #验证erlang是否安装成功`

      ![image-20220319194023036](C:\Users\18238\AppData\Roaming\Typora\typora-user-images\image-20220319194023036.png)

  13. 退出查看，先"halt()"回车后输入"."

      ```
      halt()
      .
      ```

      ![image-20220319194118937](C:\Users\18238\AppData\Roaming\Typora\typora-user-images\image-20220319194118937.png)

- ###### 4、[github获最新版本的RabbitMQ安装包](https://github.com/rabbitmq/rabbitmq-server/releases)

  ![image-20220319194332139](C:\Users\18238\AppData\Roaming\Typora\typora-user-images\image-20220319194332139.png)

- ###### 5、开始搭建RabbitMQ环境

  1. `rpm -ivh --nodeps rabbitmq-server-3.9.13-1.el8.noarch.rpm #直接安装`

     不加--nodeps可能会报erlang与rabbitmq版本不适配的错误，这里忽略依赖

  2. [配置文件地址](https://github.com/rabbitmq/rabbitmq-server/blob/v3.9.13/deps/rabbit/docs/rabbitmq.conf.example)根据需要下载，后续步骤并没有使用到该配置文件

  3. ```
     service rabbitmq-server start #启动
     service rabbitmq-server stop #停止
     service rabbitmq-server restart #重启
     ```

  4. 启动报以下错误需要修改rabbitmq-server文件

     ![image-20220319194644652](C:\Users\18238\AppData\Roaming\Typora\typora-user-images\image-20220319194644652.png)

     在line指定行数增加erlang环境变量

     ```
     ERLANGE_HOME=/usr/local/erlang
     export PATH=$PATH:$ERLANGE_HOME/bin
     ```

  5. 开启web界面管理工具，开启后能通过浏览器进行监控

     ```
     rabbitmq-plugins enable rabbitmq_management
     service rabbitmq-server restart
     ```

  6. `chkconfig rabbitmq-server on #开机启动`

  7. 防火墙开放端口

     ```
      firewall-cmd --zone=public --add-port=5672/tcp --permanent
      firewall-cmd --zone=public --add-port=15672/tcp --permanent
      firewall-cmd --zone=public --add-port=25672/tcp --permanent
      service firewalld reload
      firewall-cmd --list-ports
     ```

- ###### 6、建立新用户

  1. rabbitmq有一个默认的用户名和密码，guest和guest,但为了安全考虑，该用户名和密码		只允许本地访问，如果是远程操作的话，需要创建新的用户名和密码

     ```
     rabbitmqctl add_user username passwd  #添加用户，后面两个参数分别是用户名和密码
     rabbitmqctl set_permissions -p / username ".*" ".*" ".*"  #添加权限
     rabbitmqctl set_user_tags username administrator  #修改用户角色,将用户设为管理员
     ```

- ###### 7、浏览器地址栏访问rabbitmq主机ip地址:15672端口进入RibbitMQ管理界面

  ![image-20220319194919145](C:\Users\18238\AppData\Roaming\Typora\typora-user-images\image-20220319194919145.png)

  ![image-20220319195131456](C:\Users\18238\AppData\Roaming\Typora\typora-user-images\image-20220319195131456.png)