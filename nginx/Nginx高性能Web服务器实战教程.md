# 第3章 Nginx安装

## 3.2 Linux环境下安装Nginx

### 3.2.1 获取Nginx

Nginx在官方网站提供了软件下载。

目前Nginx发布了3中类型的版本，分别为Mainline version(开发板)、Stable version(稳定版)和Legacy versions(早期版本)，每种类型的版本中又提供了Linux版本和Windows版本。

官方网站下载页面：http://nginx.org/en/download.html

本文章基于nginx-1.10.3讲解。

下载好后进行安装：

```shell
$ tar -zxvf nginx-1.10.3.tar.gz
```

执行上述命令后，文件就被解压到当前目录下的nginx-1.10.3目录中。

```shell
auto  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  Makefile  man  objs  README  src
```

关于上述目录结构的具体介绍：

（1）src目录：存放Nginx的源代码。

（2）man目录，存放Nginx的帮助文档。

（3）html目录：存放默认网站文件。

（4）contrib目录：存放其他机构或组织贡献的文档资料。

（5）auto目录：存放大量的脚本文件，和configure脚本程序相关。

（6）configure文件：Nginx自动安装脚本，用于检查环境，生成编译代码需要的makefile文件。

（7）CHANGES、CHANGES.ru、LICENSE和README都是Nginx服务器的相关文档资料。

### 3.2.2 编译安装Nginx

1、安装依赖包

由于Nginx中的而功能是模块化，而模块又依赖于一些软件包才能使用，因此在安装Nginx前，需要完成Nginx模块依赖的软件包安装。

| 软件包     | 说明                                                    |
| ---------- | ------------------------------------------------------- |
| pcre-devel | 为Nginx模块（如rewrite）提供正则表达式库                |
|            | 为Nginx模块（如gzip）提供数据压缩用的函数库             |
|            | 为Nginx模块（如ssl）提供密码算法、证书以及SSL协议等功能 |

通过yum方式安装Nginx相关依赖包：

```shell
$ yum -y install pcre-devel openssl-devel
```

由于openssl-devel依赖于zlib-devel，在通过yum进行安装时会自动解决依赖，因此上述命令中省略了zlib-devel。

2、Nginx的编译安装

（1）切换到Nginx解压目录nginx-1.10.3下执行命令：

```shell
$ ./configure --prefix=/usr/local/nginx --with-http_ssl_module
```

在上述命令中，源码安装的第一步./configure用于对即将安装的软件进行配置，检查当前的环境是否满足安装软件（Nginx）的依赖关系。其中，--prefix选项用于设置Nginx的安装目录，默认值是/usr/local/nginx，因此也可以省略此选项或指定到其他位置；--with-http_ssl_module选项用于设置在Nginx中允许使用http_ssl_module模块的相关功能。

在Nginx的安装包中还有许多其他模块，目前暂时用不到，当用到的时候重新编译Nginx并使用“--with-”选项添加需要的模块即可。

（2）通过make命令编译和安装Nginx

```shell
$ make && make install 
```

### 3.2.3 Nginx的启动与停止

1、启动Nginx

Nginx安装完成后，切换到Nginx安装目录中的sbin目录，通过执行该目录下Nginx编译后的二进制文件即可启动程序：

```shell
$ cd /usr/local/nginx/sbin
$ ./nginx
```

执行上述操作后，若成功启动Nginx，则程序没有任何提示。使用ps命令可以查看Nginx的运行状态：

```shell
$ ps aux | grep nginx
```

在输出结果中，前2行分别是Nginx主进程（master process）和工作进程（worker process）。当看到这两个Nginx进程时，说明Nginx已经启动。从第1列可以看出，Nginx主进程以root用户运行，而工作进程以nobody用户运行；第2列显示了两个进程的ID（即PID）分别为8170,和8171。

2、停止Nginx服务

当需要停止Nginx服务时，有多重停止方式，可以根据需求采取不同的方式。具体如下：

（1）立即停止服务

Nginx程序允许传递选项-s表示发送信号到主进程，如果后面跟上stop表示停止服务。

```shell
$ ./ginx -s stop
```

（2）从容停止服务

用stop的方式是立即停止Nginx服务，无论当前工作进程是否正在处理工作。而Nginx提供的从容停止方式quit，是在完成当前工作任务后再停止。

```shell
$ ./nginx -s quit
```

（3）通过kill或killall命令杀死进程

Linux中提供了kill和killall命令可以杀死进程，从而让指定进程停止运行。

```shell
$ kill Nginx主进程的PID
```

```shell
$ killall nginx
```

| 命令                             | 说明                                                |
| -------------------------------- | --------------------------------------------------- |
| nginx -s reload                  | 在Nginx已经启动的情况下重新加载配置文件（平滑重启） |
| nginx -s reopen                  | 重新打开日志文件                                    |
| nginx -c /特定目录/nginx.conf    | 以特定目录下的配置文件启动Nginx                     |
| nginx -t                         | 检测当前配置文件是否正确                            |
| nginx -t -c /特定目录/nginx.conf | 检测特定目录下的Nginx配置文件是否正确               |
| nginx -v                         | 显示版本信息                                        |
| nginx -V                         | 显示版本信息和编译选项                              |

### 

# 第4章 Nginx基本配置

## 4.1 认识配置文件

Nginx服务器安装完成后，默认安装时自带配置文件全部存储在conf目录下，并且为了备份还原，每个配置嗯嗯建都提供了一个以.default结尾的备份文件。其中，nginx.conf是Nginx默认的主配置文件，所有功能的实现都与此文件的配置相关。

### 4.1.1 配置文件结构

打开nginx.conf配置文件，从整体结构可以看出，该配置文件主要由以下几部分组成：

```tex
main
events {...}
http {
	server {
		location {...}
	}
}
```

从上面的结构可以看出，Nginx的默认主配置文件主要由main、events、http、server和location5个块组成。并且对于嵌套块（如http、server、location）中的指令，执行的顺序为从外到内依次执行，内层块中的大部分指令会自动获取外层块指令的值作为默认值，只有某些特殊指令除外。

| 块     | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| main   | 主要控制Nginx子进程所属的用户和用户组、派生子进程数、错误日志位置与级别、pid位置、子进程优先级、进程对应CPU、进程能够打开的文件描述符数目等 |
|        | 控制Nginx处理连接的方式                                      |
| http   | Nginx处理http请求的主要配置块，大多数配置都在这里进行        |
| server | Nginx中主机的配置块，可用于配置多个虚拟主机                  |
|        | server中对应目录级别的控制块，可以有多个                     |

Nginx的指令由指令名称和参数组成。当一个指令含有多个子指令作为参数时，需要使用大括号{}进行包裹，且每条指令都以分号“;”结尾。

默认配置指令：

| 指令               | 说明                                                     |
| ------------------ | -------------------------------------------------------- |
| worker_processes   | 配置Nginx的工作进程数，一般设为CPU总核数或者总核数的两倍 |
| worker_connections | 配置Nginx允许单个进程并发连接的最大请求数                |
|                    | 用于引入配置文件                                         |
| default_type       | 设置默认文件类型                                         |
|                    | 默认值为on，表示开启高效文件传输模式                     |
| keepalive_timeout  | 设置长连接超时时间（单位：秒）                           |
|                    | 监听端口，默认监听80端口                                 |
| server_name        | 设置主机域名                                             |
|                    | 设置主机站点根目录地址                                   |
|                    | 指定默认索引文件                                         |
| error_page         | 自定义错误页面                                           |

### 4.1.2 设置用户和组

Nginx服务是由一个主进程（master process）和多个工作进程（worker process）组成的。其中主进程以root权限运行，而工作进程在默认情况下以nobody用户运行。原因在于nobody用户是一个不能登录的账号，有一个专用的ID，可将每个运行的工作进程隔离出来，这样即使黑客破坏了服务器程序，因其不是root用户，也不会影响到其他数据。

为工作进程设置的执行用户权限越低，则服务器安全系数越高。

Nginx提供两种设置用户和组的方式，一种是在安装时通过编译选项进行设置，另一种是修改配置文件。不论哪种方式在配置之前，都需要提前创建好用户和组。

1、编译安装配置文件方式

在.configure编译安装Nginx时的选项中，添加如下两个选项：

```shell
--user=<user>
--group=<group>
```

在上述选项中，user用于指定用户名称，group用于指定用户所在组的名称。

2、修改配置文件方式

打开Nginx的配置文件，找到配置用户和组的指令user：

```shell
# user nobody;
```

接下来以用户nuser和ngroup为例，修改后的配置如下：

```shell
user nuser ngroup
```

上述配置中，nuser用于指定执行工作进程的用户，ngroup用于指定nuser用户所属的组。按照上述命令修改完成后，保存nginx.conf配置文件，平滑重启。再次查询Nginx进程相关信息时，可以看到工作进程用户已成功修改为nuser。

### 4.1.3 自定义错误页

在网站访问过程中，经常会遇到各种各样的错误，如果找不到访问的页面则会提示404 Not Found错误，没有访问权限会提示403 Forbidden等，对于普通人而言，这样的提示界面并不友好。在Nginx的主配置文件中，给出了以下处理方式：

```tex
error_page 500 502 503 504 /50x.html;
```

上述配置中，error_page指令用于自定义错误页面，500、502/503和504指的就是HTTP错误代码，/50x.html用于表示当发生上述指定的任意一个错误时，都使用网站根目录下的50x.html文件处理。

除此之外，error_page指令还可以指定单个错误的处理页面、利用在线资源处理指定的错误，更改网站响应的状态码等多种设置。

1、为每种类型的错误设置单独的处理方式















































