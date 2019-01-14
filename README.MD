https://www.hengmt.com/378.html

相关
•	魔改一键脚本
•	魔改前端教程
•	魔改支付宝面对面付款
•	系统要求：建议centos7x64，centos6 X64也可以。

此为一键脚本后端的手动搭建版本

第一步、安装基本组件和SSR后端
yum -y install python-setuptools && easy_install pip && pip install cymysql speedtest-cli && yum install git
第二步、CHACHA20加密方式支持：
yum -y groupinstall "Development Tools" && 
wget https://raw.githubusercontent.com/mmmwhy/ss-panel-and-ss-py-mu/master/libsodium-1.0.13.tar.gz&&tar xf libsodium-1.0.13.tar.gz && cd libsodium-1.0.13&&./configure && make -j2 && make install&&echo /usr/local/lib > /etc/ld.so.conf.d/usr_local_lib.conf&&ldconfig&& rm -rf /root/libsodium-1.0.13.tar.gz && cd /root
第三步、下载程序源代码
git clone -b manyuser https://github.com/glzjin/shadowsocks.git
yum -y install python-devel
yum -y install libffi-devel
yum -y install openssl-devel
cd shadowsocks
pip install -r requirements.txt
cp apiconfig.py userapiconfig.py
cp config.json user-config.json
这里为止，之前的操作通过一键脚本即可完成

配置userapiconfig.py
•	公用数据库
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
glzjinmod即为公用数据库模式
•	api接口对接
#节点ID-和你网站添加的节点分配的ID一致!
NODE_ID = 3
#这里必须填，要么选择数据库要么选择httpapi（大多数使用的是数据库，httpapi不知怎么弄，2333）
API_INTERFACE = 'modwebapi' #glzjinmod (数据库方式连接)，modwebapi (http api)
WEBAPI_URL = 'http://for.mss.me'
WEBAPI_TOKEN = 'mupass'
这里也就是一键脚本为什么需要输入NODE_ID、WEBAPI_URL、WEBAPI_TOKEN的原因，脚本默认使用api进行对接，这样数据库可以不必暴露在公网。

测试运行
python server.py
测试一下，应该就可以用看到相应用户数据了。
ctrl + c退出，然后./logrun.sh运行即可


