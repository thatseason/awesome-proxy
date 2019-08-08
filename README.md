# awesome-proxy
一款基于ADSL拨号主机构建的高可用、高性能、高匿名代理池！



1. 修改配置(填写你购买的adsl主机ssh信息)
2. 构建Docker镜像、启动容器
3. 使用

**一次构建，长久使用，免维护**

* 已支持功能
1. API账户修改配置后1分钟左右会更新到数据库，动态配置账户。
2. 主机配置会定期检查，配置无误则会自动修正主机并管理。动态增减主机只需要修改配置文件即可。
3. 重启程序或重启容器会重新检测主机。
4. 根据配置文件设置定期更新整个代理池IP
5. JWT验权，Token 5小时有效。

---
* 存在问题
1. token并没有做过期校验的逻辑，如果有需要可以开发。
2. 因个服务商环境不同，自动配置ADSL的脚本并没有开发出来（如果出现问题，可能需要手动处理）。
3. 日志不是很详细。
4. 可能会出现Bug，保留了大量注释，交流学习，欢迎吐槽。

---
* 开发计划（可能开发的功能）
1. 客户端可以使用HTTPS连接代理服务器。
2. 可以控制一台代理主机的使用时间（防止高并发IP被封，充分利用IP）
3. 欢迎大家补充~

---
# OS支持
|操作系统|是否支持|
|---|---|
|Mac|是|
|Linux|是(Redhat系、Debian系)|
|Windows|不完全支持（只支持能执行sh脚本的环境cygwin、linux子系统等）|


---
# 架构图
![image](https://github.com/osof/awesome-proxy/blob/master/%E6%A1%86%E6%9E%B6%E5%9B%BEv1.png)


---
# 快速开始
## 部署
```
# 说明
可以在本地环境跑容器或者IDE中执行；也可以在云服务器上运行。
在IDE中运行需要执行api_server.py和tasks.py两个文件

# 拉取代码
git clone https://github.com/osof/awesome-proxy.git
cd awesome-proxy && chmod +x build_images.sh

# 修改配置
cd config # 并修改配置
# api_config.py : 接口、数据库等配置，只需修改代理账户、IP切换时间
# hosts.py : 配置您购买的adsl拨号主机
# supervisor.conf 该文件一般不用修改，根据api_config.py中的配置会自动覆盖相关配置。

# 检查本地环境是否已安装docker，如果没有安装则会自动安装。
./build_images.sh check


# 构建镜像
./build_images.sh make_images [you_images_tag_name]


# 启动容器
# 端口映射有三个：redis端口（可选）、API服务（必须）、Supervisor管理端口（可选）
docker run -idt --name [my_adsl] --restart=always -p xxx:xxx [you_images_tag_name]

```

---
## Demo



---
# API说明

**说明**
* 为方便用户使用，返回的代理是字典结构，用户只需loads一下即可直接使用，省去拼接字符串环节。
* 操作成功返回对应信息，操作失败返回错误信息（部分接口含状态码）
* 本程序只是为了方便用户，减少代码量！不是特别重要的部分会尽量简化状态信息。


## 验权
```
POST /api/v1/token
json: { "username": "admin", "password": "123456" }


验证成功返回token（5小时有效）
{"status": "200", "token": "token_str"}

验证失败返回错误信息
{"status": "403", 'error': 'Unauthorized access'}

```

---
## 查看API列表
```
POST /api/v1/index
提交json: { "token": "token_str"}

---------------------------------
操作成功返回： API列表

操作失败返回：
Token过期：{"status": "403", 'error': 'Unauthorized access'}
```

---
## 随机获取一个代理的值
```
POST /api/v1/random
提交json: { "token": "token_str"}

---------------------------------
操作成功返回：{ http : "http://123.456.678.789:3100" }

操作失败返回：
Token过期：{"status": "403", 'error': 'Unauthorized access'}
没有代理可用：{"status": "500", 'error': 'No proxy available'}
```

---
## 获取一个随机代理的详细信息
```
POST /api/v1/proxies
提交json: { "token": "token_str", "proxy_name": "myadsl1" }

---------------------------------
操作成功返回：{ "status": "200", "name": "myadsl1", "proxy": "{http : 'http://123.456.678.789:3100'}" }

操作失败返回：
Token过期：{"status": "403", 'error': 'Unauthorized access'}
没有代理可用：{"status": "500", 'error': '找不到代理！'}
```

---
## 获取所有代理
```
POST /api/v1/all
提交json: { "token": "token_str"}

---------------------------------
操作成功返回：{ "data": [{http : 'http://123.456.678.789:3100'},  {http : 'http://321.543.765.987:3100'}] }

操作失败返回：
Token过期：{"status": "403", 'error': 'Unauthorized access'}
没有代理可用：{"status": "500", 'error': 'No proxy available'}

```


---
## 代理数量统计
```
POST /api/v1/counts
提交json: { "token": "token_str"}

---------------------------------
操作成功返回：{ "status": "200", "counts": "代理数量（没有代理则为0）" }

操作失败返回：
Token过期：{"status": "403", 'error': 'Unauthorized access'}
```


---
## 获取机器名称
```
POST /api/v1/names
提交json: { "token": "token_str"}

---------------------------------
操作成功返回：{ "myadsl1", "myadsl2", "myadsl3"}

操作失败返回：
Token过期：{"status": "403", 'error': 'Unauthorized access'}
没有代理可用：{"status": "500", 'error': 'No proxy available'}
```


---
## 删除一个代理
```
POST /api/v1/delete
提交json: { "token": "token_str", "proxy_name": "myadsl1" }


---------------------------------
操作成功返回：{"status": "200", 'delete': 'Success'}

操作失败返回：
Token过期：{"status": "403", 'error': 'Unauthorized access'}
没有代理可用：{"status": "500", 'delete': 'Fail'}

```




---
# 其他
还没想好！