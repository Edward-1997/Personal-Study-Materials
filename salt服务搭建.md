一、安装saltstack

1、准备工作

准备两台机器：

master: 192.168.3.120

slaver: 192.168.3.121



设置两台主机hostname

```shell
hostnamectl set-hostname Master
hostnamectl set-hostname slaver
```

2、安装yum,

```shell
如果找不到先执行 yum install https://repo.saltstack.com/yum/redhat/salt-repo-latest-2.el7.noarch.rpm
Master主机安装
[root@Master ~]# yum install -y epel-release salt-master salt-minion
slaver主机安装
[root@slaver ~]# yum install -y epel-release salt-minion
```

Master主机配置：

```shell
[root@Master ~]# vim /etc/salt/minion
master: 192.168.3.121
[root@Master ~]# vim /etc/salt/master
master: 192.168.3.121
[root@Master ~]# systemctl start salt-master
[root@Master ~]# systemctl start salt-minion
```

slaver主机配置：

```shell
[root@slaver ~]# vim /etc/salt/minion
master: 192.168.3.121
[root@slaver ~]# systemctl start salt-minion
```

3、配置认证，只在Master服务器上配置

```shell
[root@Master ~]# salt-key -a Master
[root@Master ~]# salt-key -a Master
```

> -a ：accept ，-A：accept-all，-d：delete，-D：delete-all，可以使用salt-key -L 查看所有key

4、Linux开放4505，4506端口

5、验证测试：

```shell
[root@Master ~]# salt '*' test.ping
slaver:
    True
Master:
    True
```

二、安装saltstack-api 

(官方salt-api文档https://docs.saltproject.io/en/latest/ref/netapi/all/salt.netapi.rest_cherrypy.html)

(java客户端：https://github.com/SUSE/salt-netapi-client)

1、安装

```
yum -y install salt-api pyOpenSSL
```

2、配置自签名证书

```shell
cd /etc/pki/tls/certs/
make testcert


Enter pass phrase:    ===>  输入加密短语，这里我使用salt2017
Verifying - Enter pass phrase:    ===>  确认加密短语
```

生成无密码key文件：

```shell
cd /etc/pki/tls/private/
openssl rsa -in localhost.key -out localhost_nopass.key
修改文件权限：
chmod 755 /etc/pki/tls/certs/localhost.crt 
chmod 755 /etc/pki/tls/private/localhost.key 
chmod 755 /etc/pki/tls/private/localhost_nopass.key
```

添加用户：

```undefined
useradd -M -s /sbin/nologin saltapi
passwd saltapi   
```

配置salt-api

修改/etc/salt/master文件

```swift
sed -i '/#default_include/s/#default/default/g' /etc/salt/master
```

创建配置文件

```bash
mkdir -p /etc/salt/master.d/
cd /etc/salt/master.d/
touch eauth.conf
touch api.conf
```

编辑eauth.conf

```bash
external_auth:
  pam:
    saltapi:   # 用户
      - .*     # 该配置文件给予saltapi用户所有模块使用权限，出于安全考虑一般只给予特定模块使用权限
```

编辑api.conf:

```tsx
rest_cherrypy:
  port: 8001
  ssl_crt: /etc/pki/tls/certs/localhost.crt
  ssl_key: /etc/pki/tls/private/localhost_nopass.key
```

启动

```ruby
systemctl restart salt-master
systemctl start salt-api
```

3、验证服务

获取token

POST：https://192.168.3.120:8001/login

body里配置：

```json
{
    "username":"saltapi",
    "password":"root",
    "eauth":"pam"
}
```



```json
{
    "return": [
        {
            "perms": [
                ".*"
            ],
            "start": 1616210117.094833,
            "token": "4ea559cd74079e0bae1d93003ede0d060165519d",
            "expire": 1616253317.094835,
            "user": "saltapi",
            "eauth": "pam"
        }
    ]
}
```

测试服务：

POST：https://192.168.3.120:8001

header里配置：

X-Auth-Token：4ea559cd74079e0bae1d93003ede0d060165519d

body里配置：

```json
{
    "client":"local", 
    "tgt":"*",
    "fun":"cmd.run",
    "arg":"free -m"
}
```

client：这里是查询Minions状态，因此使用local参数，详情请见：https://docs.saltproject.io/en/latest/ref/clients/index.html#client-apis

* "local" uses [`LocalClient`](https://docs.saltproject.io/en/latest/ref/clients/index.html#salt.client.LocalClient) which sends commands to Minions. Equivalent to the `salt` CLI command.

* * tgt：下发服务的目标主机

    fun：执行的函数命令

    arg：参数

    对应的控制台命令是：

    salt tgt fun arg

* "runner" uses [`RunnerClient`](https://docs.saltproject.io/en/latest/ref/clients/index.html#salt.runner.RunnerClient) which invokes runner modules on the Master. Equivalent to the `salt-run` CLI command.
* "wheel" uses [`WheelClient`](https://docs.saltproject.io/en/latest/ref/clients/index.html#salt.wheel.WheelClient) which invokes wheel modules on the Master. Wheel modules do not have a direct CLI equivalent but they typically manage Master-side resources such as state files, pillar files, the Salt config files, and the [`key wheel module`](https://docs.saltproject.io/en/latest/ref/wheel/all/salt.wheel.key.html#module-salt.wheel.key) exposes similar functionality as the `salt-key` CLI command.



返回体：

```json
{
    "return": [
        {
            "Master": "              total        used        free      shared  buff/cache   available\nMem:            972         830          69           1          71          35\nSwap:          2047        1871         176",
            "slaver": "              total        used        free      shared  buff/cache   available\nMem:            972         258         169          19         544         542\nSwap:          2047           0        2047"
        }
    ]
}
```

