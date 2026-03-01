# 静态IP配置
## 两种方法：
### **进入命令界面后：**
`vi /etc/sysconfig/network-scripts/ifcfg-ens33`	  					

	（ens33，看自己的网卡是多少）

改为以下：

`BOOTPROTO=static`

`ONBOOT=yes`

`IPADDR=192.168.30.100`	//以下看自己需求

`NETMASK=255.255.255.0`

`GATEWAY=192.168.30.2`

`DNS1=114.114.114.114`

`DNS2=8.8.8.8`

`DNS3=223.5.5.5`



单独配置DNS则：

`vi /etc/resolv.conf`

修改或增加：

`nameserver x:x:x:x`



配完记得重启网络服务：

`service network restart`



### **在部署CentOS时，对网络参数修改：（推荐）**
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/64871437/1771906227088-876de33d-6b9d-402c-aa22-24c09263a440.png)

如图进入配置界面后，先打开网卡，然后直接可视化配置:

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/64871437/1771906342168-a42f3108-483d-4f2e-a6a4-094997418775.png)

> 修改虚拟网络编辑器
>
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/64871437/1772349366123-fa021d31-1db7-46ae-8000-c0d413b14dbb.png)
>
>     1. 子网IP改为与虚拟机IP同网段：192.168.13.0
> <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/64871437/1772349618475-4c1f90af-d354-4ae5-b222-62346eaa5c15.png)
>     2. NAT设置中的网关IP地址为：192.168.13.2	（同网段下的2）
>

# yum环境部署
使用yum出现以下报错时:

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/64871437/1771915317029-7157c768-f150-4a34-aece-3eea061e1159.png)

**<font style="color:rgb(77, 77, 77);">分析：**<font style="color:rgb(77, 77, 77);">由第二行报错信息得在尝试从 <font style="color:rgb(78, 161, 219) !important;">CentOS<font style="color:rgb(77, 77, 77);"> 镜像列表获取信息时遇到了问题， `mirrorlist.centos.org`<font style="color:rgb(77, 77, 77);"> 这个域名在CentOS 7中已经不被维护。同时也可能是由于<font style="color:rgb(78, 161, 219) !important;">网络连接<font style="color:rgb(77, 77, 77);">问题导致的。

解决方法：

### 先确保网络通讯正常
`ping www.baidu.com`

> Ctrl + C 停止访问
>

### 网络正常，修改原配置文件的镜像地址
通过 vi 修改 `/etc/yum.repos.d/CentOS-Base.repo` 文件，修改停止维护的 `mirror.centos.org` 可用的镜像源。

`vi /etc/yum.repos.d/CentOS-Base.repo `

进入修改界面，将红下划线行 # 注释，将蓝下划线取消注释：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/64871437/1771916918496-807fdf22-5616-47b7-b6c5-cefd9982e021.png)

改成如下：

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/64871437/1771916382598-8c47a1fb-2f9d-499b-bc25-99d0e71f2283.png)



在将四个vaseurl地址中的 `mirror.centos.org` 改成 `**mirrors.aliyun.com**` :

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/64871437/1771917271869-c0b856d6-4ca3-42c3-a522-d5278db37113.png)



然后清除yum缓存并重新加载yum:

> sudo yum clean all
>
> sudo yum makecache
>



# 配置阿里云镜像源
### 下载依赖包utils
`sudo yum install -y yum-utils device-mapper-persistent-data lvm2`

### 挂载阿里云的Docker的镜像源(配置yum)
`sudo yum-config-manager --add-repo http://mirrors.ailyun.com/docker-ce/linux/centos/docker-ce.repo`

更新yum

`sudo yum makecache fast`



# 部署Docker
### 安装docker
`# 安装最新版本` 

`sudo yum install -y docker-ce docker-ce-cli containerd.io`



`# 安装20.10.*版本`

`sudo yum install -y docker-ce-20.10.* docker-ce-cli-20.10.* containerd.io`

> 该版本的docker compose并不存在
>
> `# 安装Compose`
>
> `<font style="background-color:#FBDE28;">sudo yum install -y docker-compose-plugin`
>

### 验证
<u># 启动服务</u>

sudo systemctl start docker

<u># 设置开机自启</u>

sudo systemctl enable docker

<u>#查看版本</u>

docker version

<u># 验证服务状态</u>

sudo systemctl status docker

### 配置镜像加速器
<u>#创建配置文件目录</u>

sudo mkdir -p /etc/docker 

<u>#通过命令行生成配置   （注意缩进）</u>

sudo tee /etc/docker/daemon.json <<-'EOF'

{

    "registry-mirrors": [

        "https://docker.m.daocloud.io",

        "https://docker.imgdb.de",

        "https://docker-0.unsee.tech",

        "https://docker.hlmirror.com"

    ]

}

EOF

<u>#加载配置</u>

sudo systemctl daemon-reload 

sudo systemctl restart docker 

### 运行hello-world镜像
<u>#拉取hello-world镜像 </u>

sudo docker pull hello-world 

<u>#运行hello-world</u>

sudo docker run hello-world 

### 下载nano
sudo yum install nano

### 配置docker-compose.yml
```yaml
nano docker-compose.yml
```

```javascript
version: '3.0'
services:
  mytb:
    restart: always
    image: "thingsboard/tb-postgres"
    ports:
      - "8080:9090"
      - "1883:1883"
      - "7070:7070"
      - "5683-5688:5683-5688/udp"
    environment:
      TB_QUEUE_TYPE: in-memory
    volumes:
      - ~/.mytb-data:/data
      - ~/.mytb-logs:/var/log/thingsboard
```

然后Ctrl + O 然后回车确定名称保存

Ctrl + X 退出

> 其中的 image 行可以根据自己需求确定：
>
> <font style="color:rgb(33, 37, 41);">根据所使用的数据库有三种类型的ThingsBoard单实例docker映像：
>
> + [<font style="color:rgb(42, 125, 236);">thingsboard/tb-postgres](https://hub.docker.com/r/thingsboard/tb-postgres/)<font style="color:rgb(33, 37, 41);"> <font style="color:rgb(33, 37, 41);">- ThingsBoard与PostgreSQL数据库的单实例
>
> <font style="color:rgb(33, 37, 41);">对于具有至少1GB内存的小型服务器的推荐选项。建议使用2-4GB。
>
> + [<font style="color:rgb(42, 125, 236);">thingsboard/tb-cassandra](https://hub.docker.com/r/thingsboard/tb-cassandra/)<font style="color:rgb(33, 37, 41);"> <font style="color:rgb(33, 37, 41);">- 具有Cassandra数据库的ThingsBoard的单个实例。
>
> <font style="color:rgb(33, 37, 41);">最高性能和推荐的选项但至少需要4GB的RAM。建议使用8GB。
>
> + [<font style="color:rgb(42, 125, 236);">thingsboard/tb](https://hub.docker.com/r/thingsboard/tb/)<font style="color:rgb(33, 37, 41);"> <font style="color:rgb(33, 37, 41);">- 具有嵌入式HSQLDB数据库的ThingsBoard的单个实例。
>
> **<font style="color:rgb(33, 37, 41);">注意：**<font style="color:rgb(33, 37, 41);"> 不建议用于任何评估或生产用途仅用于开发目的和自动测试。
>

```plain
mkdir -p ~/.mytb-data && sudo chown -R 799:799 ~/.mytb-data
mkdir -p ~/.mytb-logs && sudo chown -R 799:799 ~/.mytb-logs
```

### 启动ThingsBoard平台
```plain
#拉取 thingsboard 镜像
docker compose -p thingsboard pull		
#启动并后台运行
docker compose -p thingsboard up -d
#停止 thingsboard 镜像
docker compose -p thingsboard stop
#停止并删除 thingsboard 镜像 
docker compose -p thingsboard down
```

