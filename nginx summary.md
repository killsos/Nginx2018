## nginx

## 1 nginx 安装

	- mac
	
		brew install nginx
		
	- linux
		
		yum install nginx
		
	- 安装编译
	
		1. 下载源码
			
			 cd /usr/local/src/
			
			 wget http://nginx.org/download/nginx-1.4.2.tar.gz
			
		2. 解压缩
		
			tar zxvf nginx-1.4.2.tar.gz 
			cd nginx-1.4.2
			
		3. 编译与安装
		
			./configure --prefix=/usr/local/nginx  
			
			make && make install
		
		4. 问题处理:
			在 ./config阶段 可能报错 缺少必备的库
			
			安装准备: nginx依赖于pcre库,要先安装pcre
			
			yum install pcre pcre-devel
			
			


#### 2 启动

	cd /ulsr/local/nginx, 看到如下4个目录
	
	 ....conf 配置文件  
	 
	 ... html 网页文件
	 
	 ...logs  日志文件 
	 
	 ...sbin  主要二进制程序
	 
	1. 启动nginx
	
		# ./sbin/nginx
	
	2. 问题
		
		有可能80端口被服务占用
		
		解决办法:
			
			netstat -antp
			
			把占用80端口的软件或服务关闭即可
		



#### 3 nginx

	
	nginx是有master主进程和至少一个worker子进程
	
	master进程 管理 worker子进程
	
	worker进程 响应 http请求
	



#### 4 nginx信号量

1. TERM, INT   quick shutdown

2. QUIT        Graceful shutdown  优雅的关闭进程  即等请求结束后再关闭

3. HUP         改变配置文件 平滑的重读配置文件
	
		官方解释:
		Configuration reload ,Start the new worker processes with
		
		a new configuration Gracefully shutdown the old worker processes
		
		更改配置后 新的worker进程以新配置进行工作
		
		同时优雅的结束旧的worker进程
		
		测试配置文件是否正确?
			
			./sbin/nginx -t
			
			如果返回值 successful 就是成功的
		

4. USER1       Reopen the log files 重读日志 在日志按月日分割时有用

		
		kill -USER1 nginx-PID
		
		kill -USER1 'cat logs/nginx.pid'
		
		备份日志步骤:
			
			1 将当前日志文件重命名
				
				mv access.log access-20180101.log
			
			2. 新建一个日志文件
				
				touch access.log
				
			3. 启用新的日志文件
			
				kill -USER1 'cat logs/nginx.pid'
			
			
		

5. USER2       Upgrade Executable on the fly 平滑的升级

6. WINCH       Gracefully shutdown the worker processes 优雅关闭的进程 配合USER2来升级

- 何为优雅的？
	
	就是当前worker进程执行任务 等待当前任务结束后 结束该worker进程

		
		具体语法:
			
			kill -信号选项 nginx的进程号
			
			ps aux | grep nginx  // 查看进程号  第二项就是PID
			
		在nginx的logs文件下有nginx.pid文件专门存放当前nginx的进程号
			
		所以可以 kill -信号选项 'cat logs/nginx.pid'
		
	
	
#### 5 nginx配置

	配置目录
		
		nginx/conf
		
	配置文件
		
		nginx.conf
	
	Nginx配置段
		
		1. 全局  worker_processes 1; 
		
		// 有1个工作的子进程,可以自行修改,但太大无益,因为要争夺CPU,一般设置为 CPU数*核数
		
		
		2. Event {
			// 一般是配置nginx连接的特性
			
			// 如1个worker能同时允许多少连接
		 	worker_connections  1024; 
			
			// 这是指 一个子进程最大允许连1024个连接
		}
		
		3. http {  //这是配置http服务器的主要段
			
     	   	Server1 { // 这是虚拟主机段
       
            	Location {  //定位,把特殊的路径或文件再次定位 ,如image目录单独处理
            	
				}             /// 如.php单独处理
     		}
			
     	   Server2 {
			   
     	   }
		}
		
		4. server {
		        listen 80;  #监听端口
				
		        server_name a.com; #监听域名
				
		        location / {
		                root /var/www/a.com;   #根目录定位
		                index index.html;
		        }
		}
		
		示例
			
			以域名
				
			    server {
			   		        listen 80;  #监听端口
				
			   		        server_name a.com; #监听域名
				
			   		        location / {
			   		                root /var/www/a.com;   #根目录定位
			   		                index index.html;
			   		        }
			   		}
			
			
			以端口
				
			    server {
			   		        listen 80;  #监听端口
				
			   		        server_name a.com; #监听域名
				
			   		        location / {
			   		                root a.com;   #相对目录定位 以是nginx安装目录以根目录
			   		                index index.html;
			   		        }
			   		}
			
			
			以ip
				
		    	server {
		   		        listen 80;  #监听端口
			
		   		        server_name 192.18.1.100; #监听ip   可以满足多网卡的需求
			
		   		        location / {
		   		                root a.com;   #相对目录定位 以是nginx安装目录以根目录
		   		                index index.html;
		   		        }
		   		}
				
			
			
	配置完成要验证 配置文件是否正确
		
		nginx -t
		
		如果验证成功 重启nginx service
		
			nginx -s reload
			
			systemctl restart nginx
	

#### 日志管理

	
	access_log   logs/host.access.log main;
	
	说明 该server 它的访问日志的文件是在logs目录下的host.access.log文件 
	
	使用的格式main格式  还可以自定义格式
	
	main格式:
		
		是定义好日志格式 并起个名字main 便于引用
		
	log_format main '$remote_addr-$remote_user[$time_local]"$request"'
					'$status $body_bytes_sent "$http_referer"'
					'"$http_user_agent" "$http_x_forwarded_for"';
					
					
	
	
	


### cpu知识补充

	
	总核数 = 物理CPU个数 X 每颗物理CPU的核数 
	 
	总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数
	
	超线程:
		
		所谓“超线程（Hyper-Threading，简称“HT”）”技术
		
		超线程技术就是利用特殊的硬件指令，把一个物理内核模拟成两个逻辑内核
		
		让单个处理器都能使用线程级并行计算，进而兼容多线程操作系统和软件
		
		减少了CPU的闲置时间，提高了CPU的运行速度
		
		采用超线程即是可在同一时间里，应用程序可以使用芯片的不同部分
		
		虽然单线程芯片每秒钟能够处理成千上万条指令，但是在任一时刻只能够对一条线程进行操作
		
		而超线程技术可以使芯片同时进行多线程处理，使芯片性能得到提升
		
		虽然采用超线程技术能同时执行两个线程，但它并不像两个真正的CPU那样
		
		每个CPU都具有独立的资源。当两个线程都同时需要某一个资源时，其中一个要暂时停止
		
		并让出资源，直到这些资源闲置后才能继续。因此超线程的性能并不等于两颗CPU的性能。
	
	
	查看物理CPU个数
	
	cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
	
	查看每个物理CPU中core的个数(即核数)
	
	cat /proc/cpuinfo| grep "cpu cores"| uniq
	
	查看逻辑CPU的个数
	
	cat /proc/cpuinfo| grep "processor"| wc -l
	
	查看CPU信息（型号）
	
	cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
	
	


#### NSCD

		
		(Name Service Cache Daemon)是域名服务缓存守护进程
		
		当在/etc/hosts修改后 一定要重启nscd
		
		安装  yum install -y nscd
		
		启动  systemctl start nscd
		
		重启  systemctl restart nscd
		
		添加自自动服务  systemctl enable nscd
		
	



#### 定时任务 日志分割

	
	#! /bin/bash
	
	LOGPATH=/usr/local/nginx/logs.z.com.access.log
	
	BASEPATH=/data/$(date -d yesterday + %Y%m)
	
	mkdir -p $BASEPATH/$(date -d yesterday +%Y%m)
	
	bak=$BASEPATH/$(date -d yesterday +%Y%m%d%H%m).zcom.access.log
	
	echo $bak
	
	
	
	mv $LOGPATH  $bak
	
	touch $LOGPATH
	
	kill -USER1 'cat /usr/local/nginx/logs/nginx.pid'
	
	


#### location

	
	location 语法
	
	location 有”定位”的意思, 根据Uri来进行不同的定位.
	
	在虚拟主机的配置中,是必不可少的,location可以把网站的不同部分,定位到不同的处理方式上.
	
	比如, 碰到.php, 如何调用PHP解释器?  --这时就需要location
	
	location 的语法
	
	location [=|~|~*|^~] patt {
	}
	
	中括号可以不写任何参数,此时称为一般匹配
	也可以写参数
	因此,大类型可以分为3种
	
	location = patt {} [精准匹配]
	
	location patt {}  [一般匹配]
	
	location ~ patt {} [正则匹配]
	
	如何发挥作用?
	
	首先看有没有精准匹配,如果有,则停止匹配过程
	
	
	location = patt {
	    config A
	}
	如果 $uri == patt,匹配成功，使用configA
	
	location = / {
		root   /var/www/html/;
		index  index.htm index.html;
	}
         
	location / {
		root   /usr/local/nginx/html;
		index  index.html index.htm;
	}
	  
	如果访问　　http://xxx.com/  定位流程是
	　
	1: 精准匹配中　”/”   ,得到index页为　　index.htm
	
	2: 再次访问 /index.htm , 此次内部转跳uri已经是”/index.htm” , 根目录为/usr/local/nginx/html
	
	3: 最终结果,访问了 /usr/local/nginx/html/index.htm
	
	
	总结:
		
		精准  优于  一般
		
		正则  优于  一般
		
		精准  优于  正则
		
		一般匹配如果有多个命中 返回匹配长度最长那个
		
		对于多个正则匹配任一个命中 就会返回结果停止匹配
	


#### rewrite

	
		重写中用到的指令:\
		
		if  (条件) {}  设定条件,再进行重写
	
		set #设置变量
	
		return #返回状态码 
	
		break #跳出rewrite
	
		rewrite #重写
	
	If  语法格式:
	
		If 空格 (条件) {
	    	重写模式
		}
	
	
	条件又怎么写?
	
		答:3种写法
	
		1: “=”来判断相等, 用于字符串比较
		
		2: “~” 用正则来匹配(此处的正则区分大小写)
		
	   	   “~*“ 不区分大小写的正则
			
		3: -f -d -e来判断是否为文件,为目录,是否存在.
	
	
	例子:
	
		if  ($remote_addr = 192.168.1.100) {
	    	return 403;
		}
		
		if ($http_user_agent ~ MSIE) {
			
	    	rewrite ^.*$ /ie.htm;
	        
			break; #(不break会循环重定向)
	     }
		 
		 
	    if (!-e $document_root$fastcgi_script_name) {
	    	rewrite ^.*$ /404.html 
			break;
	    } 
		
		以 xx.com/dsafsd.html这个不存在页面为例,
		
		我们观察访问日志, 日志中显示的访问路径,依然是GET /dsafsd.html HTTP/1.1
		
		提示: 服务器内部的rewrite和302跳转不一样. 
		
		跳转的话URL都变了,变成重新http请求404.html, 而内部rewrite, 上下文没变,
		
		就是说 fastcgi_script_name 仍然是 dsafsd.html,因此 会循环重定向.
		
	
		set 是设置变量用的, 可以用来达到多条件判断时作标志用
		
		达到apache下的 rewrite_condition的效果
		
		如下: 判断IE并重写,且不用break; 我们用set变量来达到目的
		
		if ($http_user_agent ~* msie) {
			set $isie 1;
		}
					
		if ($fastcgi_script_name = ie.html) {
			set $isie 0;
		}
		
		if ($isie 1) {
			rewrite ^.*$ ie.html;
		}
	
	
	Rewrite语法
	
	Rewrite 正则表达式  定向后的位置 模式
	
	Goods-3.html ---->Goods.php?goods_id=3
	
	goods-([\d]+)\.html ---> goods.php?goods_id =$1  
	
	location /ecshop {
		index index.php;
		
		rewrite goods-([\d]+)\.html$ /ecshop/goods.php?id=$1;
		
		rewrite article-([\d]+)\.html$ /ecshop/article.php?id=$1;
		
		rewrite category-(\d+)-b(\d+)\.html /ecshop/category.php?id=$1&brand=$2;
	
		rewrite category-(\d+)-b(\d+)-min(\d+)-max(\d+)-attr([\d\.]+)\.html 
	
		/ecshop/category.php?id=$1&brand=$2&price_min=$3&price_max=$4&filter_attr=$5;
	
		rewrite category-(\d+)-b(\d+)-min(\d+)-max(\d+)-attr([\d+\.])-(\d+)-([^-]+)-([^-]+)\.html 
	
		/ecshop/category.php?id=$1&brand=$2&price_min=$3&price_max=$4&filter_attr=$5&page=$6&sort=$7&order=$8;
	}
	
	


#### php + nginx

	
	nginx+php的编译
	
	apache一般是把php当做自己的一个模块来启动的
	
	而nginx则是把http请求变量(如get,user_agent等)转发给 php进程
	
	即php独立进程,与nginx进行通信. 称为 fastcgi运行方式
	
	因此,为apache所编译的php,是不能用于nginx的
	
	注意: 我们编译的PHP 要有如下功能:
	
	连接mysql, gd, ttf, 以fpm(fascgi)方式运行
	
	在源码目录执行
	
		./configure -help | grep mysql  # 显示php编译过程与mysql相关的配置
		
		
	./configure  --prefix=/usr/local/fastphp \
	--with-mysql=mysqlnd \
	--enable-mysqlnd \
	--with-gd \
	--enable-gd-native-ttf \
	--enable-gd-jis-conv
	--enable-fpm
		
	
	// prefix 指定安装目录
	
	// mysqlnd mysql native driver 是php自带mysql驱动程序
	
	// with-mysql=mysqlnd 指定mysql的驱动mysqlnd
	
	// enbale-mysqlnd 打开mysqlnd功能
	
	// gd 相关都是图库功能
	
	// fpm  FPM（FastCGI 进程管理器）用于替换 PHP FastCGI 的大部分附加功能，对于高负载网站是非常有用的 
	
	
	php7中需要拷贝两个配置文件
		
		1. php7源文件中 php.ini-development   ------>        php7安装目录的lib下php.ini
		
		2. 将php7安装目录的etc/php-fpm.d/www.conf.default   ------>  生成一个www.conf
		
	nginx+php的配置比较简单,核心就一句话----
	
	把请求的信息转发给9000端口的PHP进程, 
	
	让PHP进程处理 指定目录下的PHP文件.
	
	location ~ \.php$ {
		root html; // 存放php文件的目录 这里是相对目录 相对于nginx的安装目录 也可以是绝对目录
	    fastcgi_pass   127.0.0.1:9000;
	    fastcgi_index  index.php;
	    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
	    include        fastcgi_params;
	 }
	 
	 // $document_root参数 就是指的这个location的root
	 
	 // fastcgi_script_name参数  就是指所请求的php文件名 
	
	
	1:碰到php文件,
	
	2: 把根目录定位到 html,
	
	3: 把请求上下文转交给9000端口PHP进程,
	
	4: 并告诉PHP进程,当前的脚本是 $document_root$fastcgi_scriptname
	
	(注:PHP会去找这个脚本并处理,所以脚本的位置要指对)
	
	
	
	在linux下 用localhost连接的时候 不是通过 TCP/IP Socket协议来连接 而是用UNIX Domain Socket来连接的
	
	这是 Unix Domain Socket 是用本机进程间通信IPC
	
	而TCP/IP socket 是主机之间远程通信的协议
	
	所以当使用localhost会认为是本机进程间通信 使用 Unix Domain Socket 
	
	对于127.0.0.1就是使用TCP/IP Socket
	
	



#### Gzip

	
	请求:
	Accept-Encoding:gzip,deflate,sdch
	
	响应:
	Content-Encoding:gzip
	Content-Length:36093
	
	
	再把页面另存下来,观察,约10W字节,实际传输的36093字节
	原因-------就在于gzip压缩上.
	原理: 
	浏览器---请求----> 声明可以接受 gzip压缩 或 deflate压缩 或compress 或 sdch压缩
	从http协议的角度看--请求头 声明 acceopt-encoding: gzip deflate sdch  
	
	(是指压缩算法,其中sdch是google倡导的一种压缩方式,目前支持的服务器尚不多)
	
	
	服务器-->回应---把内容用gzip方式压缩---->发给浏览器
	浏览<-----解码gzip-----接收gzip压缩内容----
	
	推算一下节省的带宽:
	假设 news.163.com  PV  2亿
	2*10^8  *  9*10^4 字节 == 
	2*10^8 * 9 * 10^4  * 10^-9 = 12*K*G = 18T
	节省的带宽是非常惊人的
	
	
	gzip配置的常用参数:
	
	gzip on|off;  #是否开启gzip
	
	gzip_buffers 32 4K| 16 8K #缓冲(压缩在内存中缓冲几块? 每块多大?)
	
	gzip_comp_level [1-9] #推荐6 压缩级别(级别越高,压的越小,越浪费CPU计算资源)
	
	gzip_disable #正则匹配UA 什么样的Uri不进行gzip
	
	gzip_min_length 200 # 开始压缩的最小长度(再小就不要压缩了,意义不在)
	
	gzip_http_version 1.0|1.1 # 开始压缩的http协议版本(可以不设置,目前几乎全是1.1协议)
	
	gzip_proxied          # 设置请求者代理服务器,该如何缓存内容
	
	gzip_types text/plain  application/xml # 对哪些类型的文件用压缩 如txt,xml,html ,css
	
	gzip_vary on|off  # 是否传输gzip压缩标志
	
	注意: 
	图片/mp3这样的二进制文件,不必压缩
	因为压缩率比较小, 比如100->80字节,而且压缩也是耗费CPU资源的.
	比较小的文件不必压缩
	
	


#### expires

	
	nginx的缓存设置  提高网站性能
	
	对于网站的图片,尤其是新闻站, 图片一旦发布, 改动的可能是非常小的.我们希望 能否在用户访问一次后, 图片缓存在用户的浏览器端,且时间比较长的缓存.
	可以, 用到 nginx的expires设置 
	
	nginx中设置过期时间,非常简单
	
	在location或if段里,来写
	
	格式 
		
	location ~* \.(jppg|jpeg|gif|png)
		expires 30s;
	    expires 30m;
	    expires 2h;
	    expires 30d;
		  
	(注意:服务器的日期要准确,如果服务器的日期落后于实际日期,可能导致缓存失效)
	
	另: 304 也是一种很好的缓存手段
	
	原理是: 服务器响应文件内容是,同时响应etag标签(内容的签名,内容一变,他也变), 和 last_modified_since 2个标签值
	
	浏览器下次去请求时,头信息发送这两个标签, 服务器检测文件有没有发生变化,如无,直接头信息返回 etag,last_modified_since
	
	浏览器知道内容无改变,于是直接调用本地缓存.
	
	这个过程,也请求了服务器,但是传着的内容极少.
	
	对于变化周期较短的,如静态html,js,css,比较适于用这个方式
	
	
	


#### nginx反向代理服务器+负载均衡

	
	用nginx做反向代理和负载均衡非常简单
	
	支持两个用法 1个proxy_pass, 1个upstream,分别用来做:  反向代理 和  负载均衡
	
		proxy_pass http://192.168.1.200:80   # 指到处理php的服务器
		
		upstream ServerGroupName {
			
			server address1 weight=1 fail_timeout=3,  // address1=ip1:listen
			
			server address2 weight=2 fail_timeout=3,   // address1=ip2:listen
		
		}
	
	server {
		listen 81,
		
		server_name ip1,
		
		root: /hhtm
	}
	
	server {
		listen 82,
		
		server_name address2,
		
		root: /hhtm
	}
	
	proxy_pass http://ServerGroupName
	
	反向代理后端如果有多台服务器,自然可形成负载均衡,
	
	但proxy_pass如何指向多台服务器?
	
		把多台服务器用 upstream 指定绑定在一起并起个组名,然后  proxy_pass  指向该组
	
		默认的均衡的算法很简单,就是针对后端服务器的顺序,逐个请求.
	
		也有其他负载均衡算法,如一致性哈希,需要安装第3方模块 例如: ngx_http_upstream_consistent_hash
		
	
	proxy_set_header X-Forwarded-For $remote_addr
	
	X-Forwarded-For:
	
		简称XFF头，它代表客户端，也就是HTTP的请求端真实的IP，
		
		只有在通过了HTTP 代理或者负载均衡服务器时才会添加该项。它不是RFC中定义的标准请求头信息，
		
		标准格式如下：X-Forwarded-For: client1, proxy1, proxy2
		
		鉴于伪造这一字段非常容易，应该谨慎使用X-Forwarded-For字段
		
		正常情况下XFF中最后一个IP地址是最后一个代理服务器的IP地址, 这通常是一个比较可靠的信息来源
			


#### nginx---memcached

	
	设置 memcached
	
	set  $memcached_key   "$uri",
	
	memcached_pass  ip:port,
	
	error_page 404 /callback.php    # memcached没有找到结果 去PHP查找结果
	
	
	多台memcached时候 nginx与php 如何保持集群上的算法同步？
	
	
	nginx 附带外部模块的编译  ./configure --prefix=/usr/local/nginx/ --add-module=modulePath
	
	􏰖􏰗配置memcache集群
		 
		 upstream memserver { // 把用到的memcached节点,声明在一个组里
			 
		 	hash_key $request_uri;  // hash计算时的依据,以uri做依据来hash
			
			server localhost:11211; 
			
			server localhost:11212;
		}
		
	Location里
	
		location / {
	    	root   html;
			
	    	set $memcached_key $uri;
			
	    	memcached_pass memserver;  // memserver为上面的memcache节点的名称
			
	    	error_page 404 /writemem.php;
		}
	
	安装第三方模块后的配置
	
	Nginx.conf中
	
	upstream memserver {
		
		consistent_hash $request_uri;
		
		server ip:11211;  // server只能ip地址 千万不能用localhost
		
		server ip:11212;
	}
		
	在PHP.ini中,如下配置
	
	memcache.hash_strategy = consistent
	
	这样: nginx与PHP即可完成对memcached的集群与负载均衡算法
		
		
		
