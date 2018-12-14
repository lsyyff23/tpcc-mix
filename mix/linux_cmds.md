
#### ----[mount]----

    https://blog.csdn.net/tojohnonly/article/details/71374984
    # mount -t cifs -o username=Bob,password=123456 //192.168.0.102/Share /usr/local/bin/code
    参数说明 : username , Window 系统登录用户名 ; password : Window 系统登录密码 ; //192.168.0.102/Share : 设置Window共享目录的路径 ; /usr/local/bin/code : 挂载到 Linux 下的那个目录

    --
    http://blog.51cto.com/lancatsky/1738520
    centos7挂载windows共享时，提示被共享的位置写保护，只能以只读方式挂载，紧接着就是以只读方式挂载失败
    初时以为是另一台centos6.4已挂载此共享，所以同时挂载会引起读写保护，取消6.4的mount后依然是这样，
    后来各种搜索，发现可能是组件少装了，于是

    yum install cifs-utils -y

    安装完后，正常挂载使用。

    --

    mount error(13): Permission denied
    https://blog.csdn.net/jingxia2008/article/details/50218933/
    mount -v -t cifs -o username=yl1660,password=abc_123,iocharset=utf8,sec=ntlm //10.86.4.12/YBASE/ ~/windows/YBASE

    mount -v -t cifs -o username=yl1660,password=abc_123,iocharset=utf8,sec=ntlm //10.86.4.12/YDBC_test/ ~/windows/YDBC_test

    mount -v -t cifs -o username=yl1660,password=abc_123,iocharset=utf8,sec=ntlm //10.86.4.12/YDBC_SDK/ ~/windows/ydbc_sdk

    mount -v -t cifs -o username=yl1660,password=abc_123,iocharset=utf8,sec=ntlm //10.86.4.12/pgxa/ ~/windows/pgxa

-------------

#### -----[rsync]----

    rsync  src dest
    rsync -ravz  ~/windows/YBASE/include/* ./include
    rsync -ravz  ~/windows/YDBC_test/code/cpp/YBASE/src/Main.cpp /root/YDBC/2ydbc/code/cpp/YBASE/src/Main.cpp

    rsync -ravz  ~/windows/YDBC_test/code/cpp/YBASE/* /root/YDBC/2ydbc/code/cpp/YBASE
    rsync -ravz  ~/windows/YDBC_test/code/cpp/YDBC/* /root/YDBC/2ydbc/code/cpp/YDBC
    rsync -ravz  ~/windows/YDBC_test/code/cpp/YSYNC/* /root/YDBC/2ydbc/code/cpp/YSYNC
    rsync -ravz  ~/windows/YDBC_test/code/cpp/YMSG/* /root/YDBC/2ydbc/code/cpp/YMSG

    g++ --std=c++11 -fPIC $DAR -shared -o libYBASE.so $IINCLUDEPATH $LLIBPATH $CPPS $PGLIBPATH $LLIBS -Wl,-rpath,/usr/local/lib -g

####  ------[sync]------

    密码登录：
    server A: 10.200.110.164 (current)
    server B: 10.200.110.167

    从A登录B
    A:
    #生成key
    $ ssh-keygen -t rsa
    #复制pub-key 到远端要登录的服务器
    $ ssh-copy-id -i ~/.ssh/id_rsa.pub root@10.200.110.167

    10.200.110.164 dbcb
    10.200.110.165 operloga
    10.200.110.166 operlogb
    10.200.110.162 synca
    10.200.110.179 syncb
    ssh-copy-id -i ~/.ssh/id_rsa.pub  root@10.200.110.164
    ssh-copy-id -i ~/.ssh/id_rsa.pub  root@10.200.110.165
    ssh-copy-id -i ~/.ssh/id_rsa.pub  root@10.200.110.166
    ssh-copy-id -i ~/.ssh/id_rsa.pub  root@10.200.110.162;
    ssh-copy-id -i ~/.ssh/id_rsa.pub  root@10.200.110.179


####  -------[find]--------
    查询大文件
    find / -type f -size +100M  -print0 | xargs -0 ls -lh

#### -------[gstack]--------

抓取进程运行栈信息

    gstack <pid>
#### -------[watch]--------

    watch -n 1 "ps -aux | grep pgdata"

#### -------[ssh]--------
    ssh-copy-id -i ~/.ssh/id_rsa.pub root@ip

    ssh -T root@ip  "shell cmds"
    -T 禁用终端
    ssh -tt root@ip "shell cmds"
    -tt 强制启用终端

    ssh root@ip < cmd.sh
    发送远程脚本命令

    ssh root@ip 'bash -s' < cmd.sh params
    发送带参远程脚本命令

    https://www.cnblogs.com/wanghetao/p/3872919.html
    -
    方法一、
    修改/etc/ssh/sshd_config配置文件，找到ClientAliveCountMax（单位为分钟）修改你想要的值，
    执行service sshd reload 
    --
    方法二、
    找到所在用户的.ssh目录,如root用户该目录在：
    /root/.ssh/
    在该目录创建config文件
    vi /root/.ssh/config
    加入下面一句：
    ServerAliveInterval 60
    保存退出，重新开启root用户的shell，则再ssh远程服务器的时候，
    不会因为长时间操作断开。应该是加入这句之后，ssh客户端会每隔一
    段时间自动与ssh服务器通信一次，所以长时间操作不会断开。
    ---
    方法三、
    修改/etc/profile配置文件
    # vi /etc/profile
    增加：TMOUT=1800
    这样30分钟没操作就自动LOGOUT
    方法四、
    利用expect 模拟键盘动作，在闲置时间之内模拟地给个键盘响应,将下列代码保存为xxx，然后用expect执行
    #!/usr/bin/expect  
    set timeout 60  
    spawn ssh user@host   
          interact {          
                timeout 300 {send "\x20"}  
          } 
    expect xxx
    接着按提示输入密码就可以了，这样每隔300秒就会自动打一个空格(\x20)，具体的时间间隔可以根据具体情况设置。
    方法五、
    如果你在windows下通过工具连接，可以设置为
    secureCRT：选项---终端---反空闲 中设置每隔多少秒发送一个字符串，或者是NO-OP协议包
    putty：putty -> Connection -> Seconds between keepalives ( 0 to turn off ), 默认为0, 改为300.

#### -------[主机名修改]--------
    依次修改每台主机的network文件：

    vim /etc/sysconfig/network

    根据服务器信息设定主机名：

    HOSTNAME=broker-b
####  -------[算术运算]--------
    在Bash的算术运算中有以下几种方法：
    名称                语法                    范例
    算术扩展            ((算术式))r=((1+2*3))
    使用外部程序expr    expr 算术式              r=`expr 1+2*3`
    使用[][算术式]                r=$[1+2]
    使用内置命令        declare -i 变量=算术式    declare -i r=1+2*3
    使用内置命令let     let 算术式                let r=1+2

#### -------[awk]--------

1. 查看不同状态的连接数数量

        [root@cp-nginx ~]# netstat -an | awk '/^tcp/ {++y[$NF]} END {for(w in y) print w, y[w]}'
        LISTEN 8
        ESTABLISHED 2400
        FIN_WAIT1 2
        TIME_WAIT 6000

2. 查看每个ip跟服务器建立的连接数

        [root@cp-nginx ~]# netstat -nat|awk '{print$5}'|awk -F : '{print$1}'|sort|uniq -c|sort -rn
         31 45.116.147.178
         20 45.116.147.186
         12 23.234.45.34
         11 103.56.195.17

        （PS：正则解析：显示第5列，-F : 以：分割，显示列，sort 排序，uniq -c统计排序过程中的重复行，sort -rn 按纯数字进行逆序排序）

3. 查看每个ip建立的ESTABLISHED/TIME_OUT状态的连接数

        [root@cp-nginx ~]# netstat -nat|grep ESTABLISHED|awk '{print$5}'|awk -F : '{print$1}'|sort|uniq -c|sort -rn
         24 103.56.195.17
         19 45.116.147.186
         18 103.56.195.18
         17 45.116.147.178

4. 获得普通外部变量

        $ test='awk code'
        $ echo | awk  '{print test}' test="$test"
        awk code
        $ echo | awk  test="$test" '{print test}'

#### ------[sysctl.conf]--------
    https://blog.csdn.net/bluetjs/article/details/80965967
    vim /etc/sysctl.conf
    添加以下配置文件：

    net.ipv4.tcp_syncookies = 1
    net.ipv4.tcp_tw_reuse = 1
    net.ipv4.tcp_tw_recycle = 1
    net.ipv4.tcp_fin_timeout = 300

    /sbin/sysctl -p 让参数生效，调优完成

    参数详解：

    1.net.ipv4.tcp_syncookies = 1 表示开启 syn cookies 。当出现 syn 等待队列溢出时，启用 cookies 来处理，可防范少量 syn ***，默认为 0 ，表示关闭； 
    2.net.ipv4.tcp_tw_reuse = 1 表示开启重用。允许将 time-wait sockets 重新用于新的 tcp 连接，默认为 0 ，表示关闭；
    3.net.ipv4.tcp_tw_recycle = 1 表示开启 tcp 连接中 time-wait sockets 的快速回收，默认为 0 ，表示关闭。
    4.net.ipv4.tcp_fin_timeout 修改系靳默认的 timeout 时间

#### reverse-i-search

    keyboard: ctrl+r
        (reverse-i-search)`':

#### sed
    from '/tmp/csv/warehouse.csv' WITH CSV;
    sed -i "s/'\(.*warehouse.csv\)'/'\/a\/b'/g" txtsqlTableCopies
    替换'/tmp/csv/warehouse.csv' 为 '/a/b'
    用\转义(), '为匹配， .* 为匹配任意字符

[如何用Sed和正则表达式提取子字符串](http://blog.sina.com.cn/s/blog_5656bf3e0102w9xb.html)

    Since sed can take any char as delimiter, you can try using another one that doesn't appear in your replacement string:
[转义字符问题](https://stackoverflow.com/questions/9366816/sed-fails-with-unknown-option-to-s-error)
    
    替换from '/tmp/csv/order-line.csv' WITH CSV; 里''的内容
    文件名sqlTableCopies

    echo "---warehouse num $gen---"
    cur_path=`pwd`
    # $1 = 1 , generate data, and load to db
    if [ $gen -gt 0 ]; then
        mkdir -p $cur_path/dt/$gen
        rm $cur_path/dt/$gen/* -rf;
        #./runLoader.sh props.pgxl numWarehouses $gen fileLocation $cur_path/dt/$gen/

        for j in warehouse.csv item.csv stock.csv district.csv customer.csv  cust-hist.csv order.csv order-line.csv new-order.csv
        do
            tpath=$cur_path/dt/$gen/$j
            echo $tpath
            sed -i "s@'\(.*$j\)'@'$tpath'@g" sqlTableCopies 
        done
    else
        if [ ! -f $cur_path/dt/default/customer.csv ]; then
            echo "csv files no exists, exists 0"
            exit 0
        fi
    fi

### su
    su - postgres -c "initdb -D /usr/local/pgdata"


### strace  pstack pstree

    pstack: 与gdb 一起安装
    strace: yum install -y strace
    pstree: yum install -y psmisc

    然后通过strace工具跟踪Coordinator进程，采集系统调用栈样本做分析。

    # 将11437进程的系统调用栈输出到11437.strace文件，Ctrl+C停止
    strace -o 11437.strace -Tttyi -s 400 -p 11437

    最后通过pstack工具多次跟踪Coordinator进程，采集进程栈样本结合源码做分析。

    # 将11437进程的当前时刻的进程栈输出到11437.pstack文件
    pstack 11437
    # 反复多次
    for ((i=0;i<=1000;i++)); do pstack 11437 >> 11437.pstack; sleep 0.01; done
    # 根据方法名搜索，找到源码文件
    grep -r PostmasterMain postgres-xl-9.5r1.1/src/
    grep -r PortalRun postgres-xl-9.5r1.1/src/



### ntpdate
    [r1](https://blog.csdn.net/qq_27754983/article/details/69386408)
    ntpdate time.nist.gov
    ntpdate -u asia.pool.ntp.org
    1、使用ntpdate时候需要关闭ntpd服务；
    2、虽然你的Linux防火墙允许123端口的udp协议，但是可能你的上层网络服务ISP是禁用特殊端口来传输ntp协议的。这时候使用-u 参数即可

### timezone
    [r1](https://blog.csdn.net/zsg88/article/details/75212835/)
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    hwclock -w


### 查看硬盘型号
    [rf](https://blog.csdn.net/harbor1981/article/details/42772377/)
    yum install smartmontools -y

    1、fdisk -l 查看你的硬盘编号，如sda,sdb 等

    2、smartctl --all /dev/sda