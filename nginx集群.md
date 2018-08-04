## nginx aggregate


## nginx集群思路

	
	高性能的服务器的架设
	
	对于高性能网站 ,请求量大,如何支撑?
	
	1: 要减少请求 对于开发人员----合并css, 背景图片, 减少mysql查询等.
	
	2: 对于运维 nginx的expires ,利用浏览器缓存等,减少查询
	
	3: 利用cdn来响应请求
	
	4: 最终剩下的,不可避免的请求----服务器集群+负载均衡来支撑
	
	所以,来到第4步后,就不要再考虑减少请求这个方向了
	
	而是思考如何更好的响应高并发请求
	
	大的认识-------既然响应是不可避免的,我们要做的是把工作内容”平均”分给每台服务器
	
	最理想的状态 每台服务器的性能都被充分利用
	
	
	



#### 压力测试

	
	Apache安装包中自带的压力测试工具 Apache Benchmark(简称ab)
	
	安装: yum install httpd-tools
	
	.bin/ab -c 200 -n 8000 http://url
	
	-c 并发数   concurrent connections
	
	-n 测试次数
	
	
	在nginx编译指定 --with-http_stub_status_module 选项
	
	


#### 优化思路:

	
	nginx响应请求
	
		1: 建立socket连接
		
		2: 打开文件,并沿socket返回
		
		操作系统对于一个进程默认是只允许打开1024个文件
			
			ulimit -n // 查看当前打开文件个数
			
			ulimit  -n 20000 // 设置一个进程最大打开20000个文件
		
	排查问题,也要注意观察这两点,
	
	主要从系统的dmesg ,和nginx的error.log来观察
	
	
	socket分为两个层面
	
		系统层面:
			
			1. 最大连接数  somaxconn
				
				more /proc/sys/net/core/somaxconn
				
				echo 50000 > /proc/sys/net/core/somaxconn
			
			2. 洪水攻击   不能洪水抵御
			
				cat  /proc/sys/net/ipv4/tcp_synccookies
				
				echo 0 > /proc/sys/net/ipv4/tcp_synccookies
			
			3. 加快tcp连接回收 recycle
				
				cat  /proc/sys/net/ipv4/tcp_tw_recycle
				
				echo 1 > /proc/sys/net/ipv4/tcp_tw_recycle
			
			4. 空的tcp是否允许回收利用 reuse
				
				cat  /proc/sys/net/ipv4/tcp_tw_reuse
				
				echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse
		
		nginx层面:
		
			1. 子进程允许打开的连接数  也就是 worker_connections
				
				在nginx.conf中events选中 worker_connections 设置
				
			2. 加快http连接快速关闭 keep-alive
			
				在nginx.conf中events选中 keepalive——timeout 设置为 2秒以内
	
	文件分为两个层面
		
	  系统层面:
	  
		1. 设置打开文件数最够大       
		
			ulimit  -n 20000 // 设置最大打开20000个文件
			
			ulimit -a // 查看结果
		
	 nginx层面:
	 
	 	1. 子进程允许打开的文件数  worker_rlimit_nofile
		
			在nginx.conf中worker_rlimit_nofile设置
				
				worker_rlimit_nofile 10000;
		
	
	
	
	connnection: keep-alive分析
		
		1 优点 节省Tcp连接过程 复用已有连接
		
		2. 对于高并发的网站 Tcp连接很珍贵 所以要尽快释放掉
		
			在nginx.conf中events选中 keepalive——timeout 设置为 2秒以内
			


#### 高并发架构

	
	1. php高并发
	
		fpm进程中有master进程和child进程两种进程
		
		master进程管理child进程
		
		child进程如果没有使用会被回收
		
		高并发时时刻刻使用 所以不希望chid进程被回收
			
			设置 pm = static; // 始终存在 ondemand选项是根据需要产生和回收进程
			
			pm.max_children = 16 // 设置进程最大个数 
	
	   
	   根据listen选项 设置不同端口 产生多个不同配置文件 进而启动多个不同进程
	   
	   upstream phpServer {
	   
	   		./sbin/php-fpm -y etc/php-fpm-9000.conf    //  listen 192.168.1.1:9000
	
	  	  	./sbin/php-fpm -y etc/php-fpm-9001.conf    //  listen 192.168.1.1:9001
	   
	   	 	./sbin/php-fpm -y etc/php-fpm-9002.conf    //  listen 192.168.1.1:9002
	   
	   	 	./sbin/php-fpm -y etc/php-fpm-9003.conf    //  listen 192.168.1.1:9003
	   
	   	 	./sbin/php-fpm -y etc/php-fpm-9006.conf    //  listen 192.168.1.1:9004
	  
	   }
	   
	   这样就启动5个主进程  每个主进程有16个子进程
	   
	   所以总共有 80个子进程 处理php请求
	   
	   
    2. memcache 高并发
		
		upstream memcacheServer {
			consistent_hash $request_uri;
			
			server ....
		
			
		}
		
	3. mysql 高并发
		
		
	   
	   
