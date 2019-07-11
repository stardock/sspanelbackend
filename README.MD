

## 准备阶段  

* 已经完成前端配置  

## 安装基本组件和SSR后端  

* 安装基本组件  
```  
yum -y install python-setuptools && easy_install pip && pip install cymysql speedtest-cli && yum install git  
```  
* 添加CHACHA20加密方式支持  
```
yum -y groupinstall "Development Tools" && yum -y install libsodium  
```  

* 下载程序源代码  
```  
git clone -b manyuser https://github.com/NimaQu/shadowsocks
yum -y install python-devel  
yum -y install libffi-devel  
yum -y install openssl-devel  
cd shadowsocks  
pip install -r requirements.txt  
cp apiconfig.py userapiconfig.py  
cp config.json user-config.json  
```  
该版本现已支持chain-f  

## 后端配置  

* 配置userapiconfig.py  
    * 公用数据库
    ```    
    #节点ID-和你网站添加的节点分配的ID一致!  
    NODE_ID = 3  
    #这里必须填，要么选择数据库要么选择httpapi（大多数使用的是数据库，httpapi不知怎么弄，2333）  
    API_INTERFACE = 'glzjinmod' #glzjinmod (数据库方式连接)，modwebapi (http api)  
    # Mysql 数据库连接信息  
    MYSQL_HOST = '127.0.0.1'  
    MYSQL_PORT = 3306  
    MYSQL_USER = 'ss'  
    MYSQL_PASS = 'ss'  
    MYSQL_DB = 'shadowsocks'  
    MYSQL_UPDATE_TIME = 60       
    ```   
    glzjinmod即为公用数据库模式  
    

   * API接口方式  
   ```  
    #节点ID-和你网站添加的节点分配的ID一致!  
    NODE_ID = 3  
    #这里必须填，要么选择数据库要么选择httpapi（大多数使用的是数据库，httpapi不知怎么弄，2333）  
    API_INTERFACE = 'modwebapi' #glzjinmod (数据库方式连接)，modwebapi (http api)  
    WEBAPI_URL = 'http://for.mss.me'  
    WEBAPI_TOKEN = 'mupass'  
    ```  
    这里也就是一键脚本为什么需要输入NODE_ID、WEBAPI_URL、WEBAPI_TOKEN的原因，脚本默认使用api进行对接，这样数据库可以不必暴露在公网。  


## 测试运行
  `python server.py`  
  测试一下，应该就可以用看到相应用户数据了。  
  ctrl + c退出即可  

## 添加守护进程自动运行  

* 安装必备软件  
```  
pip install --upgrade pip
yum install epel-release -y
yum install supervisor -y
```  

* 修改 `/etc/supervisord.conf` 添加如下内容  
```  
[inet_http_server]  
port = 127.0.0.1:9001  

[rpcinterface:supervisor]  
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface  

[program:mu]  
command=python /root/shadowsocks/server.py  
directory=/root/shadowsocks  
autorestart=true  
startsecs=10  
startretries=36  
redirect_stderr=true  
user=root ; setuid to this UNIX account to run the program  
log_stdout=true ; if true, log program stdout (default true)  
log_stderr=true ; if true, log program stderr (def false)  
logfile=/var/log/mu.log ; child log path, use NONE for none; default AUTO  
;logfile_maxbytes=1MB ; max # logfile bytes b4 rotation (default 50MB)  
;logfile_backups=10 ; # of logfile backups (default 10)  
```  
* 若 `supervisord.conf` 文件为空，请补上如下前置内容  

```
; Sample supervisor config file.  
  
[unix_http_server]  
file=/var/run/supervisor/supervisor.sock   ; (the path to the socket file)  
;chmod=0700                 ; sockef file mode (default 0700)  
;chown=nobody:nogroup       ; socket file uid:gid owner  
;username=user              ; (default is no username (open server))  
;password=123               ; (default is no password (open server))  
  
;[inet_http_server]         ; inet (TCP) server disabled by default  
;port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)  
;username=user              ; (default is no username (open server))  
;password=123               ; (default is no password (open server))  
  
[supervisord]  
logfile=/var/log/supervisor/supervisord.log  ; (main log file;default $CWD/supervisord.log)  
logfile_maxbytes=50MB       ; (max main logfile bytes b4 rotation;default 50MB)  
logfile_backups=10          ; (num of main logfile rotation backups;default 10)  
loglevel=info               ; (log level;default info; others: debug,warn,trace)  
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)  
nodaemon=false              ; (start in foreground if true;default false)  
minfds=1024                 ; (min. avail startup file descriptors;default 1024)  
minprocs=200                ; (min. avail process descriptors;default 200)  
;umask=022                  ; (process file creation umask;default 022)  
;user=chrism                 ; (default is current user, required if root)  
;identifier=supervisor       ; (supervisord identifier, default is 'supervisor')  
;directory=/tmp              ; (default is not to cd during start)  
;nocleanup=true              ; (don't clean up tempfiles at start;default false)  
;childlogdir=/tmp            ; ('AUTO' child log dir, default $TEMP)  
;environment=KEY=value       ; (key value pairs to add to environment)  
;strip_ansi=false            ; (strip ansi escape codes in logs; def. false)  
  
[supervisorctl]  
serverurl=unix:///var/run/supervisor/supervisor.sock ; use a unix:// URL  for a unix socket  
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket  
;username=chris              ; should be same as http_username if set  
;password=123                ; should be same as http_password if set  
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")  
;history_file=~/.sc_history  ; use readline history if available  
  
[include]  
files = supervisord.d/*.ini  
```


* 执行如下代码启动守护进程  
```  
systemctl start supervisord  
systemctl enable supervisord  
```  

* 查询是否成功  
`supervisorctl status mu`  
  * 如下显示说明成功  
  `mu                               RUNNING   pid 3254, uptime 4 days, 0:42:33`  
  

