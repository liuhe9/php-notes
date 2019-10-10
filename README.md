
# 问题集锦
## mac下ab测试出现 
```
apr_socket_recv: Connection reset by peer (54)
```

解决办法

```
# 查看当前内核和进程能打开的文件描述符限制
$ sysctl -A | grep kern.maxfiles
kern.maxfiles: 12288             # 系统级的限制
kern.maxfilesperproc: 10240    # 内核级的限制

# 通过sysctl命令热更改 系统重启后失效
$ sysctl -w kern.maxfiles=204800
$ sysctl -w kern.maxfilesperproc=204800

# 通过配置文件永久更改 重启生效
# 在/etc/sysctl.conf中写入
kern.maxfiles=204800 kern.maxfilesperproc=204800
$ ulimit -S -n
10240
$ ulimit -S -n 204800
$ ulimit -S -n
204800
```

+ 查看端口命令
```
netstat -ntpl | grep 6379
```
+ 内存不够，交换空间
sudo fallocate -l 3G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

+ mysql 8
```
sudo wget -c https://dev.mysql.com/get/mysql-apt-config_0.8.13-1_all.deb 
sudo dpkg -i mysql-apt-config_0.8.13-1_all.deb
sudo apt update
sudo apt install mysql-server
```
客户端ssh远程连接数据库出现Client does not support authentication protocol requested by server; consider upgrading MySQL client
登录到服务器mysql
```
mysql -u root -p
use mysql;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
FLUSH PRIVILEGES;
```
+ php7.3
```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
sudo apt-get install -y php7.3
sudo apt-get install -y php7.3-mysql php7.3-curl php7.3-mbstring php7.3-bcmath php7.3-xml
sudo apt-get install -y php7.3-fpm
sudo apt-get install -y php7.3-redis
```

+ composer
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'a5c698ffe4b8e849a443b120cd5ba38043260d5c4023dbf93e1558871f1f07f58274fc6f4c93bcfd858c6bd0775cd8d1') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
sudo php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/bin/composer
composer -v
sudo composer config -g repo.packagist composer https://packagist.phpcomposer.com
```
+ Content-Length mismatch, received 348704 bytes out of the expected 1017196
```
composer clear-cache
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```


+ redis服务端
```
sudo apt-get install -y redis-server
//修改redis持久化存储
sudo vim /etc/redis/redis.conf
```

+ nginx服务器
```
sudo apt-get install -y nginx
sudo apt-get autoremove -y apache2
```

+发布脚本
```
<?php
namespace Deployer;

require 'recipe/laravel.php';

// Project name
set('application', 'app');

// Project repository
set('repository', 'git@github.com:xxx.git');

// 保存最近5个版本
set('keep_releases', 5);

// [Optional] Allocate tty for git clone. Default value is false.
set('git_tty', true);

// Shared files/dirs between deploys
add('shared_files', []);
add('shared_dirs', []);

// Writable dirs by web server
add('writable_dirs', []);

// Hosts
host('liuhetx')
    ->stage('prod')
    ->user('ubuntu')
    ->port(22)
    ->set('deploy_path', '/data/deploy/wuliu-api')
    ->forwardAgent(true)
    ->multiplexing(true)
    ->addSshOption('UserKnownHostsFile', '/dev/null')
    ->addSshOption('StrictHostKeyChecking', 'no');

// 自定义任务：重置 opcache 缓存
task('opcache_reset', function () {
    run('{{bin/php}} -r \'opcache_reset();\'');
});


// nginx文件
desc('Upload nginx file');
task('nginx_conf', function () {
    upload('nginx/api.conf', '/etc/nginx/sites-enabled/');
    upload('nginx/cert/', '/etc/nginx/cert/');
    run('sudo service nginx reload');
});

// 将本地的 .env 文件上传到代码目录的 .env
desc('Upload .env file');
task('env:upload', function () {
    upload('./.env.prod', '{{release_path}}/.env');
});

after('deploy:shared', 'env:upload');

after('deploy:failed', 'deploy:unlock');
before('deploy:symlink', 'artisan:migrate');
//只
// before('deploy:symlink', 'artisan:db:seed');
after('deploy:symlink', 'opcache_reset');
//after('deploy:symlink', 'nginx_conf');

```
### redis
+ redis报 WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
```
sudo vim /etc/sysctl.conf
net.core.somaxconn= 1024
```
+ Could not connect to Redis at 127.0.0.1:6379: Connection refused
```
sudo redis-cli
sudo redis-cli /etc/redis/redis.conf
bind 127.0.0.1
```

### nginx指定环境变量,larvavel判断 .env  .env.local
fastcgi_param  APP_ENV local;


### mac 更新到catalina  根目录只读，read-only filesystem
+ 关闭sip https://www.jianshu.com/p/fe78d2036192 按步骤来
+ 执行自启动脚本mount https://jingyan.baidu.com/article/9c69d48fe7a2c913c9024eb6.html
start.sh 脚本代码
```
#!/bin/bash 
echo "sudo密码" | sudo -S mount -uw /
```