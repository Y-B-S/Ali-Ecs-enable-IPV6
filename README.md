# Ali-Ecs-enable-IPV6
Ali ecs服务器启用ipv6通过苹果上架检测
	阿里云ECS开启IPV6 
	系统环境
 
	ECS“经典网络”类型，如果是“专有网络”，需要将HE配置隧道地址命令中的IPv4地址修改为ECS实例的内网地址。
一 启用ipv6的配置
1. 编辑 /etc/sysctl.conf 文件，将其中三条禁用IPv6的设置更改为： 
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
2.让配置生效
运行 sysctl -p 的命令，启用IPv6 
二 申请隧道
 (1). 在浏览器中打开 https://www.tunnelbroker.net 
注册账号,并登陆,然后点击页面上Create Regular Tunnel,进入下一页.
 
(2). IPv4 Endpoint (Your side):填入服务器的公网地址, 如下图
 
(3). Available Tunnel Server:可用的通道服务器选择其中一项,如下
 
然后点击最下方按钮 Create Tunnel 进入下一页
(4). 选择Example Configurations ,并在下拉框中选择Linux-route2选项,如下图
 
选择Linux-route2之后下面文本区域会出现命令,将命令复制
(此处注意ECS环境为经典网络还是专有网络,专有网络下面LOCAL 之后需要改为ECS实例的内网地址)
 
(5).将上一步复制的命令,粘贴到linux服务上执行,命令如下
//挂载ipv6模块
modprobe ipv6   
//添加点对点通道
ip tunnel add he-ipv6 mode sit remote 216.218.221.6 local 103.244.67.114 ttl 255
//启用he-ipv6  
ip link set he-ipv6 up
//添加新的协议地址
ip addr add 2001:470:18:e50::2/64 dev he-ipv6   
//添加路由使本地所有网络经过he-ipv6
ip route add ::/0 dev he-ipv6     
//强制使用IPV6协议
ip -f inet6 addr
三 服务监听IPV6地址
如nginx服务同时监听ipv4 和ipv6配置如下
server {
            listen 80;
# 监听IPv6的80端口
            listen [::]:80;
}
四 用HE提供的免费DNS解析服务通过IPv6 DNS检测 
问题:在 http://ipv6-test.com 提交网址检测IPv6访问时，会问到为什么“IPv6 DNS server”检测不能通过。 
可能因为国内大环境的条件限制，包括阿里云提供的免费“云解析DNS”服务，虽然能提供AAAA解析，但DNS服务器本身可能是没有IPV6地址的，所以使用阿里云DNS的域名（如ns9.hichina.com）在ipv6-test.com测试时，“IPv6DNS server”检测不能通过。 
解决办法:
1. 在 https://dns.he.net 登录(和面申请隧道使用同一账号)或注册一个新用户，添加新域名前，提示需要将域名的DNS服务地址修改为：ns2.he.net, ns3.he.net, ns4.he.net 和 ns5.he.net 
 

2.点击Add a new domain,填入要解析的域名如下图
 
     3. 在 Zone Management 列表中，点击 Edit 编辑 如图
 
4. 点击 New AAAA ，添加一条IPv6解析记录 
 
5. 在Name里填写主机名，如www，IPv6 Address 里添加 IPv6地址，之后点击 Submit 提交 
注意:TTL 时间代表dns解析多久生效,这里选择最小值
 
6.添加IPV6解析记录成功
 
7.IPV6测试地址:
http://ipv6-test.com/validate.php 
IPV6解析成功效果如下图:
 
参考阿里云开发者论坛:
https://bbs.aliyun.com/read/285557.html?spm=5176.100241.0.0.7cPWcv
https://bbs.aliyun.com/read/313524.html?spm=5176.bbsl239.0.0.ChziMf
