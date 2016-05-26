
#lnmp + xdebug 配置


此笔记记录 CentOS 7 下安装 lnmp 环境，并配置xdebug。

## 步骤

1. 安装 nginx
	- 更新源
    	```bash
    	sudo wget http://repo.webtatic.com/yum/el7/x86_64/RPMS/epel-release-7-5.noarch.rpm && rpm -ivh epel-release-7-5.noarch.rpm
    	```

	- 安装 nginxs
    	```bash
    	sudo yum install nginx -y
    
    	sudo systemctl enable nginx.service
    	```

	- 配置 nginx

		编辑 `/etc/nginx/nginx.conf` 文件，修改 `server` 代码段如下：
		```bash
		server{
			listen       80;
			server_name  yourdomain.com;
			root         /var/www/html;

			access_log /var/log/nginx/html-access.log;
			error_log  /var/log/nginx/html-error.log error;

			location = /favicon.ico { access_log off; log_not_found off; }
			location = /robots.txt  { access_log off; log_not_found off; }

			charset utf-8;
		    index index.html index.htm index.php;

			location / {
		       try_files $uri $uri/ /index.php$is_args$args;
		    }

			location ~ \.php$ {
				fastcgi_split_path_info ^(.+\.php)(/.+)$;
				fastcgi_pass unix:/var/run/php-fpm/php5-fpm.sock;
				fastcgi_index index.php;
				include fastcgi.conf;
			}

		} 
		```
	- 启动 nginx
		
		```bash
		sudo systemctl start nginx	
		```
  
2. 安装php
	- 更新源
        ```bash
        sudo rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
        ```
    
	- 安装php5.5
    	```bash
    	sudo yum install php55w php55w-fpm php55w-mysqlnd php55w-gd php55w-eaccelerator php55w-pdo php55w-mbstring php55w-mhash php55w-cli php55w-mcrypt php55w-imap php55w-ldap php55w-odbc php55w-pear php55w-xml php55w-xmlrpc php55w-snmp php55w-pear php55w-common php55w-devel php55w-pecl-xdebug curl openssl openssl-devel -y
    
    	sudo systemctl enable php-fpm
    	```

	- 配置 php
	
		编辑 `/etc/php.ini` 文件，修改 `;cgi.fix_pathinfo=1` 为 `cgi.fix_pathinfo=0`
		
		编辑 `/etc/php-fpm.d/www.conf` 文件
		- 找到 `;listen = 127.0.0.1:9000` 修改为 `listen = /var/run/php-fpm/php5-fpm.sock`
		- 找到 `;listen.mode = 0660` 修改为 `listen.mode = 0666`

	- 启动 php-fpm 
        ```bsah
        sudo systemctl start php-fpm
        ```

			
3. 安装MariaDB

	```bash
	sudo yum install mariadb-server
	sudo systemctl enable mariadb
	sudo systemctl start mariadb
	sudo mysql_secure_installation
	```


4. 配置xdebug
	
	- 在php.ini中加入如下配置：
    	```bash
    	;xdebug
    	[xdebug]
    	xdebug.remote_enable=1
    	xdebug.remote_connect_back=1
    	xdebug.remote_port=9000
    	```
	
	- 重启 nginx 和 php 进程即可
    	```bash
    	sudo systemctl restart php-fpm
    	sudo systemctl restart nginx
    	```
