- nginx 调优的坑
> events {accept_mutex on;}  
#优化同一时刻只有一个请求而避免多个睡眠进程被唤醒的设置，on为防止被同时唤醒，默认为off，因此nginx刚安装完以后要进行适当的优化。  

>sendfile        on; #  
#是由后端程序负责把源文件打包加密生成目标文件，然后程序读取目标文件返回给浏览器；这种做法有个致命的缺陷就是占用大量后端程序资源，如果遇到一些访客下载速度巨慢，就会造成大量资源被长期占用得不到释放（如后端程序占用的CPU/内存/进程等），很快后端程序就会因为没有资源可用而无法正常提供服务。通常表现就是 nginx报502错误，而sendfile打开后配合location可以实现有nginx检测文件使用存在，如果存在就有nginx直接提供静态文件的浏览服务，因此可以提升服务器性能.

>keepalive_timeout  65 60;   
#后面的60为发送给客户端应答报文头部中显示的超时时间设置  
Keep-Alive:timeout=60   
#浏览器收到的服务器返回的报文  
Connection:close    
#浏览器收到的服务器返回的报文

- nginx 前置机 time_wait 过多  
>1. Linux内核参数调优  
>2.	netstat -anpt |grep -E '80|443' 过滤查看IP请求，排查  
>3.	根据日志排查

- 后端服务 time_wait 过多
> 开启内核参数调优：  
net.ipv4.tcp_syncookies = 1   
#表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；  
net.ipv4.tcp_tw_reuse = 1   
#表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；  
net.ipv4.tcp_tw_recycle = 1  
#表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。  
net.ipv4.tcp_fin_timeout   
修改系默认的 TIMEOUT 时间  

>#nginx配置如下：  
    proxy_http_version 1.1;  
    proxy_set_header Connection "";  
    upstream 配置 keepalive 1000  

- nginx upstram 常见问题 
>#上下游调用关机，看fail次数、timeout次数等，查看日志

- nginx localtion优先级
> =：对URI做精确匹配；  
	location = / {  
		...  
	}  
~：对URI做正则表达式模式匹配，区分字符大小写；  
~*：对URI做正则表达式模式匹配，不区分字符大小写；  
^~：对URI的左半部分做匹配检查，不区分字符大小写；  
不带符号：匹配起始于此uri的所有的url；  
location /lizi {  
		...  
	}  
	#例如www.example.com/lizi/xxx/xxx  
	#只要是路径以/lizi开头的都匹配，这种一般用于最后的通用匹配。  

>注意：匹配优先级：=, ^~, ～/～*，不带符号；
