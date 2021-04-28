# MPI
服务器系统：Ubbantu 16

## Installation 
### Before install
Dependency:
```
$ sudo apt-get update -y
$ sudo apt-get install gcc -y
$ sudo apt-get install g++ -y
$ sudo apt-get install gfortran -y
$ apt-get install make -y
$ apt-get install python -y
```
### Download  & install 
官网：www.mpich.org
install path: /usr/local/mpich2
```
$ cd ~ && wget https://www.mpich.org/static/downloads/1.0.8/mpich2-1.0.8.tar.gz
$ tar -zvxf mpich2-1.0.8.tar.gz -C /usr/local/
$ cd /usr/local/mpich2-1.0.8
$ mkdir /usr/local/mpich2 && ./configure --prefix=/usr/local/mpich2
$ make && make install
```

### env setting 
配置环境变量的方法有很多，把安装路径下的`bin` 加进去环境变量就行，然后 `source` 一下
you can change the `/etc/environment` or `.bashrc` or `etc/profile` as you like 
we will modify the `/etc/environemt`
```
$ vim /etc/environemt 
```
Path to add the mpich path: `/usr/local/mpich2/bin`
Here is my `/etc/environemt`
```
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/mpich2/bin"
MANPATH="/usr/local/mpich2/share/man"
```
Source the env file: `$ source /etc/environemt`

## Network configurate
配置多机运行的关键：
- 相同的 secretword
- `/etc/hosts` 内有各个 node 的 ip 和 hostname
- 能相互通过 ssh 访问
- `mpd.host` 内的 hostname 代表了想使用的设备名（至少要填入自己的设备名）
- 允许流量入站出站；防火墙打开（不安全的设置，但实用）
- 
### Change the hostname 
`$vim /etc/hostname` then enter you hostname 

### set up visit via ssh-key
after create the rsa key
快捷设置 ssh `$ ssh-copy-id username@hostname`
then enter the `root passowrd` of the hostname 
you can use `$ ssh hostname` to visit other hosts

#### configurate /etc/mpd.conf
create: `$ vim /etc/mpd.conf`
all the nodes should have a **same** secretword
```
secretword=123456
```

### configurate /etc/hosts
`$ vim /etc/hosts`
add all the nodes you have, including the main node 

format:
```
ipaddr1  hostname1
ipaddr2  hostname2
...
```

#### configurate ~/mpd.host 
`$ vim ~/mpd.host`
add all the nodes you want to use
format:
```
hostname1
hostname2
...
```

#### iptable and firewall
iptable 设置全部流量自由出入
```
$ iptables -A INPUT -j ACCEPT
$ iptables -A FORWARD -j ACCEPT
```
关闭防火墙
```
$ ufw disable
```

## Check mpd & exec
处理思路：单机运行 -> 两台机器之间运行 -> 多台机器之间运行
启动：`mpd &`
查看mpd情况：`mpdtrace`
关闭全部mpd：`mpdallexit`
检查和其他机器的连接：`mpdcheck -f  ~/mpd.hosts -ssh`
启动mpd（数字3要改成host的数量） `mpdboot -n 3 -f ~/mpd.hosts -v `
编译程序：`make clean && make cpi`
运行程序（10代表线程数 ./cpi是编译后的程序名） `mpirun -np 10 ./cpi`