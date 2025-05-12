[toc]

## 🧊的 domjudge 与 ICPC tools 的搭建笔记

### 安装 DOMjudge 和相关依赖

```bash
sudo apt-get update
sudo apt-get upgrade
```

```bash
sudo apt install acl zip unzip mariadb-server nginx php-fpm php-gd php-cli php-intl php-mbstring php-mysql php-curl php-json php-xml php-zip composer ntp
```

```bash
sudo apt install make gcc g++ debootstrap libcgroup-dev lsof procps libcurl4-gnutls-dev libjsoncpp-dev libmagic-dev
```

```bash
sudo apt install pkg-config
```

```bash
pip install requests
pip install jinja2 jsonpatch jsonschema
```



进入到根目录中，将文件下载

```bash
sudo wget https://www.domjudge.org/releases/domjudge-8.3.1.tar.gz
sudo tar -zxvf domjudge-8.3.1.tar.gz
```

解压后文件会存入当前目录下的 domjudge-8.3.1 ，进入目录下的 domjudge-8.3.1 目录

```bash
cd ./domjudge-8.3.1
```



### 编译 Domjudge 并且配置数据库和 web 服务器

```BASH
sudo ./configure --prefix=/domjudge --with-domjudge-user=root --with-baseurl=127.0.0.1
```

> 如果这一步警告缺少 `pkg-config` 则 `sudo apt install pkg-config`

```bash
sudo make domserver
sudo make install-domserver
```

进入到刚才 --prefix 指定的路径下的 domserver

```bash
cd /domjudge/domserver
sudo bin/dj_setup_database -s install
```

> 此处的 php 版本需要根据自己电脑里的进行修改，如要查看，具体操作为
>
> ```bash
> cd /etc/php
> ls
> ```
>
> 此处以 php8.1 为例子
>
> 如果提示 No module named 'requests' 则执行 `pip install requests`
>
> 如果安装 requests 的时候提示
>
> ```bash
> cloud-init 19.1 requires jinja2, which is not installed.
> cloud-init 19.1 requires jsonpatch, which is not installed.
> cloud-init 19.1 requires jsonschema, which is not installed.
> ```
>
> 则执行`pip install jinja2 jsonpatch jsonschema`

```bash
sudo ln -s /domjudge/domserver/etc/nginx-conf /etc/nginx/sites-enabled/domjudge
sudo ln -s /domjudge/domserver/etc/domjudge-fpm.conf /etc/php/8.1/fpm/pool.d/domjudge.conf
sudo service php8.1-fpm reload
```

```bash
cd /etc/nginx/sites-enabled
sudo rm default
sudo service nginx reload
cd /domjudge/domserver
sudo chown www-data:www-data -R webapp/public/*
```

> 若出现 404 则是因为没有执行 `sudo rm default`

接着在网页里面 login，账号为 admin，密码通过以下命令获取：

```bash
sudo cat /domjudge/domserver/etc/initial_admin_password.secret
```



### 配置 php 和 mysql

```bash
cd /etc/php/8.1/fpm/pool.d
sudo nano domjudge.conf
```

打开配置文件后找到 `php_admin_value[memory_limit]` 改成

```bash
php_admin_value[memory_limit] = 1024M
```

再找到 `php_admin_value[date.timezone]` 一栏，改成

```bash
php_admin_value[date.timezone] = Asia/Shanghai
```

保存退出后，执行

```bash
sudo service php8.1-fpm reload
sudo nano /etc/mysql/conf.d/mysql.cnf
```

将文件中的 `[mysql]` 删掉，接着粘贴以下内容：

```bash
[mysqld]
max_connections = 1000
max_allowed_packet = 512MB
innodb_log_file_size = 2560MB
```

> 在实际运行中，`max_allowed_packet` 要改成两倍于题目测试数据文件的大小；`innodb_log_file_size` 要改成十倍于题目测试数据文件的大小

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

找到 `max_allowed_packet = 1G` 取消掉这一行的注释

```bash
sudo systemctl restart mysql
```



若上述配置成功，则下述界面 (DOMjudge --> Config checker) 里面会显示

![](./img/domjudge-1.png)

![](./img/domjudge-2.png)



### 配置 judgehost (docker)

按照以下命令下载 docker

```bash
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
```

> 若提示 `密钥存储在过时的 trusted.gpg 密钥环中` 则执行下列命令
>
> ```bash
> cd /etc/apt
> sudo cp trusted.gpg trusted.gpg.d
> sudo apt-get update
> ```

```
sudo apt-get install docker-ce
cd /etc/docker
sudo touch daemon.json
sudo nano daemon.json
```

然后在 json 文件中粘贴以下内容

```json
{
    "registry-mirrors": ["https://docker.1ms.run"]
}
```

然后执行

```bash
sudo service docker restart
```

接着配置 cgroups

```bash
sudo nano /etc/default/grub
```

找到 `GRUB_CMDLINE_LINUX_DEFAULT=""` 这一行，改成

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet cgroup_enable=memory swapaccount=1 systemd.unified_cgroup_hierarchy=0"
```

保存退出后

```bash
sudo update-grub
sudo reboot
```

使用下列命令获得密码

```bash
sudo cat /domjudge/domserver/etc/restapi.secret
```

将下列的 `<domserver password>` 改成上面获得的

```bash
sudo docker run -d -it --privileged -v /sys/fs/cgroup:/sys/fs/cgroup --name judgehost-new0 --hostname localhost --network="host" -e DAEMON_ID=0 -e CONTAINER_TIMEZONE=Asia/Shanghai -e JUDGEDAEMON_PASSWORD=<domserver password> -e DOMSERVER_BASEURL=http://localhost/domjudge/ domjudge/judgehost:8.3.1
```

> 如果需要起多个 judgehost，则需要修改 `--name` 和 `DAEMON_ID` 参数，后者是核编号，不能超过物理机的核心数。



起成功后，输入 `sudo docker ps -a` 会查看到以下状态

![](./img/domjudge-3.png)

![](./img/domjudge-4.png)



### pol2dom 的安装以及使用

#### 安装环境及其依赖

```bash
sudo apt install texlive-latex-base texlive-latex-extra
sudo apt install texlive-lang-chinese
sudo apt install texlive-fonts-recommended texlive-fonts-extra
```

```bash
pip install webcolors
pip3 install tqdm
```



#### 书写 config.yaml

```bash
contest_name: test

front_page_statements: absolute_path/statements_frontpage.pdf # A single-page pdf
front_page_solutions: absolute_path/solutions_frontpage.pdf # A single-page pdf
header_image: absolute_path/header.pdf # A rectangular-shaped image (it can be in any format supported by \includegraphics in latex).

polygon:
  key: xxxx
  secret: xxxx
domjudge:
  server: http://xxx/domjudge
  username: admin
  password: xxx
  contest_id: xxx
problems:
- name: hello-minecraft
  label: A
  color: CornflowerBlue
  polygon_id: 375741
  author: Galaxy
  preparation: Galaxy
```

> 这里的 polygon 的 key 和 secret 可以在 polygon 中的 `settings` 下的 `New API key` 中获得
>
> problems 中的 polygon_id 为 problem 中右边的 id 
>
> domjudge 中的 server 为你的 server
>
> username 需要为管理员
>
> password 需要为上述 username 的密码
>
> contest_id 为打开比赛网页时，上面的链接的数字
>
> 例如 `http://xxx/domjudge/jury/contests/2` 那么即为 2
