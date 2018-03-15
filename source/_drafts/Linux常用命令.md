dmesg -T | grep "(java)"  //查找进程挂掉的信息
dmesg | tail

$ dmesg | tail
[1880957.563150] perl invoked oom-killer: gfp_mask=0x280da, order=0, oom_score_adj=0
[...]
[1880957.563400] Out of memory: Kill process 18694 (perl) score 246 or sacrifice child
[1880957.563408] Killed process 18694 (perl) total-vm:1972392kB, anon-rss:1953348kB, file-rss:0kB
[2320864.954447] TCP: Possible SYN flooding on port 7001. Dropping request.  Check SNMP counters.
该命令会输出系统日志的最后10行。示例中的输出，可以看见一次内核的oom kill和一次TCP丢包。这些日志可以帮助排查性能问题。千万不要忘了这一步。
• 关闭防火墙
systemctl stop firewalld
systemctl stop  firewalld.service
防火墙    firewall
• 查看防火墙状态
firewall-cmd --state
uptime

$ uptime
23:51:26 up 21:31,  1 user,  load average: 30.02, 26.43, 19.02
这个命令可以快速查看机器的负载情况。在Linux系统中，这些数据表示等待CPU资源的进程和阻塞在不可中断IO进程（进程状态为D）的数量。这些数据可以让我们对系统资源使用有一个宏观的了解。

命令的输出分别表示1分钟、5分钟、15分钟的平均负载情况。通过这三个数据，可以了解服务器负载是在趋于紧张还是区域缓解。如果1分钟平均负载很高，而15分钟平均负载很低，说明服务器正在命令高负载情况，需要进一步排查CPU资源都消耗在了哪里。反之，如果15分钟平均负载很高，1分钟平均负载较低，则有可能是CPU资源紧张时刻已经过去。

上面例子中的输出，可以看见最近1分钟的平均负载非常高，且远高于最近15分钟负载，因此我们需要继续排查当前系统中有什么进程消耗了大量的资源。可以通过下文将会介绍的vmstat、mpstat等命令进一步排查。

vmstat 1

$ vmstat 1
procs ---------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
34  0    0 200889792  73708 591828    0    0     0     5    6   10 96  1  3  0  0
32  0    0 200889920  73708 591860    0    0     0   592 13284 4282 98  1  1  0  0
32  0    0 200890112  73708 591860    0    0     0     0 9501 2154 99  1  0  0  0
32  0    0 200889568  73712 591856    0    0     0    48 11900 2459 99  0  0  0  0
32  0    0 200890208  73712 591860    0    0     0     0 15898 4840 98  1  1  0  0
^C
vmstat(8) 命令，每行会输出一些系统核心指标，这些指标可以让我们更详细的了解系统状态。后面跟的参数1，表示每秒输出一次统计信息，表头提示了每一列的含义，这几介绍一些和性能调优相关的列：

r：等待在CPU资源的进程数。这个数据比平均负载更加能够体现CPU负载情况，数据中不包含等待IO的进程。如果这个数值大于机器CPU核数，那么机器的CPU资源已经饱和。
free：系统可用内存数（以千字节为单位），如果剩余内存不足，也会导致系统性能问题。下文介绍到的free命令，可以更详细的了解系统内存的使用情况。
si, so：交换区写入和读取的数量。如果这个数据不为0，说明系统已经在使用交换区（swap），机器物理内存已经不足。
us, sy, id, wa, st：这些都代表了CPU时间的消耗，它们分别表示用户时间（user）、系统（内核）时间（sys）、空闲时间（idle）、IO等待时间（wait）和被偷走的时间（stolen，一般被其他虚拟机消耗）。
上述这些CPU时间，可以让我们很快了解CPU是否出于繁忙状态。一般情况下，如果用户时间和系统时间相加非常大，CPU出于忙于执行指令。如果IO等待时间很长，那么系统的瓶颈可能在磁盘IO。

示例命令的输出可以看见，大量CPU时间消耗在用户态，也就是用户应用程序消耗了CPU时间。这不一定是性能问题，需要结合r队列，一起分析。

2.4、nohup

用途：不挂断地运行命令，说的见简单点就是让程序能够在后台运行。 
语法：nohup Command [ Arg … ] [　& ]

描述：nohup 命令运行由 Command 参数和任何相关的 Arg 参数指定的命令，忽略所有挂断（SIGHUP）信号。在注销后使用 nohup 命令运行后台中的程序。要运行后台中的 nohup 命令，添加 & （ 表示”and”的符号）到命令的尾部。

无论是否将 nohup 命令的输出重定向到终端，输出都将附加到当前目录的 nohup.out 文件中。如果当前目录的 nohup.out 文件不可写，输出重定向到 $HOME/nohup.out 文件中。如果没有文件能创建或打开以用于追加，那么 Command 参数指定的命令不可调用。如果标准错误是一个终端，那么把指定的命令写给标准错误的所有输出作为标准输出重定向到相同的文件描述符

意思就是让我们的Java程序在后台运行，并且将程序产生的日志记录到$LOG_FILE中，>表示写入（先清除内容再写入内容），>>表示追加

2>&1的含义

对于& 1 更准确的说应该是文件描述符 1,而1 一般代表的就是STDOUT_FILENO,实际上这个操作就是一个dup2(2)调用.他标准输出到all_result ,然后复制标准输出到文件描述符2(STDERR_FILENO),其后果就是文件描述符1和2指向同一个文件表项,也可以说错误的输出被合并了.其中0 表示键盘输入 1表示屏幕输出 2表示错误输出.把标准出错重定向到标准输出,然后扔到/DEV/NULL下面去。通俗的说，就是把所有标准输出和标准出错都扔到垃圾桶里面。

2>&1 是将标准出错重定向到标准输出，这里的标准输出已经重定向到了$LOG_FILE文件，即将标准出错也输出到$LOG_FILE文件中

有关详细介绍可以查看这篇博文：http://blog.csdn.net/annicybc/article/details/4814872
网络配置文件地址  /etc/sysconfig/network-scripts

查看进程启动时间
ps -eo pid,lstart,etime | grep 5176

| wc -l

例如： 查看22端口现在运行的情况 lsof -i :22

2.按文件修改时间查看文件
a.按降序，即最近的修改 ls -lt
moudaen@morton:~$ ls -lt

pmap

　　可以根据进程查看进程相关信息占用的内存情况，(进程号可以通过ps查看)如下所示：
　　$ pmap -d 14596

ifstat 查看网络
RX Pkts/Rate  数据包接收流量
RX Errs/Drop  丢包
TX Pkts/Rate  数据包发送流量
RX Data/Rate 数据接收流量
TX Data/Rate 数据发送流量
lo与p2p1表示两个网卡

nethogs: 按进程查看流量占用
iptraf: 按连接/端口查看流量
ifstat: 按设备查看流量
ethtool: 诊断工具
tcpdump: 抓包工具
ss: 连接查看工具
其他: dstat, slurm, nload, bmon

sar -n DEV 1

rxpck/s    每秒接收的数据包
txpck/s    每秒发送的数据包
rxbyt/s     每秒种接收的字节数
txbyt/s      每秒种发送的字节数
rxcmp/s   每秒接收的压缩包数
txcmp/s    每秒发送的压缩包数
rxmcst/s   每秒接收的多播数据包

查看对内存占用
jmap -histo:live 30443|sort -n -r -k 2 |head -20

cat stack | grep 'com.aliyun' | sort | uniq -c | sort -k 1 -nr

image

这个例子中，第四行就是我们应用出现问题的地方，有250个线程都block在这个方法上

逐步分解一下这个命令组合

cat stack | grep 'com.aliyun'： 在控制台输出stack文件的内容，并通过grep过滤只有自己业务代码的堆栈信息

sort :排序，把相同的字符串放在一起

uniq -c:相同出现的行，合并成一行，并计算重复的次数，输出的结果第一列是出现次数、第二列是内容

sort -k 1 -nr: 按第一列数字从大到小进行排列

##带宽限制
添加带宽限制
tc qdisc add dev eth0 root tbf rate 200kbit latency 50ms burst 1540

删除带宽限制
/sbin/tc qdisc del dev eth0 root tbf

Common traffic control solutions
  1. Limit total bandwidth to a known rate; TBF, HTB with child class(es).
  2. Limit the bandwidth of a particular user, service or client; HTB classes and classifying with a filter. traffic.
  3. Maximize TCP throughput on an asymmetric link; prioritize transmission of ACK packets, wondershaper.
  4. Reserve bandwidth for a particular application or user; HTB with children classes and classifying.
  5. Prefer latency sensitive traffic; PRIO inside an HTB class.
  6. Managed oversubscribed bandwidth; HTB with borrowing.
  7. Allow equitable distribution of unreserved bandwidth; HTB with borrowing.
  8. Ensure that a particular type of traffic is dropped; policer attached to a filter with a drop action.
实践案例

pfifo_fast

pfifo_fast是系统默认的队列类型，只起到调度的作用，不对数据流量进行控制。
其中pfifo中的p是packet的缩写，表示queue的大小计量单位为packet。
可以通过ip命令查看当前网络队列设置

# ip link list
TBF

TBF队列通过设置令牌的产生速度来限制数据包的发送。

// 在eth0上设置一个tbf队列，网络带宽为200kbit，延迟50ms，缓冲区为1540个字节
// rate表示令牌的产生速率
// latency表示数据包在队列中的最长等待时间
// 对burst参数解释一下：
//   burst means the maximum amount of bytes that tokens can be available for instantaneously.
//   如果数据包的到达速率与令牌的产生速率一致，即200kbit，则数据不会排队，令牌也不会剩余
//   如果数据包的到达速率小于令牌的产生速率，则令牌会有一定的剩余。
//   如果后续某一会数据包的到达速率超过了令牌的产生速率，则可以一次性的消耗一定量的令牌。
//   burst就是用于限制这“一次性”消耗的令牌的数量的，以字节数为单位。
# tc qdisc add dev eth0 root tbf rate 200kbit latency 50ms burst 1540

# tc qdisc ls dev eth0 // 查看eth0上的队列规则
SFQ

SFQ队列通过一个hash函数将不同会话(如TCP流)分到不同的FIFO队列中，从而保证
数据流的公平性。

// perturb表示每10秒更新一次hash函数
# tc qdisc add dev eth0 root sfq perturb 10
HTB

// handle是一组用户指定的标示符，格式为major:minor。
// 如果是一条queueing discipline，minor需要一直为0。
# tc qdisc add dev eth0 root handle 1:0 htb

// parent指明该新增的class添加到那一个父handle上去
// classid指明该class handle的唯一ID，minor需要是非零值
// ceil定义rate的上界
# tc class add dev eth0 parent 1:1 classid 1:6 htb rate 256kbit ceil 512kbit

// 新建一个带宽为100kbps的root class, 其classid为1:1
# tc class add dev eth0 parent 1: classid 1:1 htb rate 100kbps ceil 100kbps
// 接着建立两个子class，指定其parent为1:1，ceil用来限制子类最大的带宽
# tc class add dev eth0 parent 1:1 classid 1:10 htb rate 40kbps ceil 100kbps
# tc class add dev eth0 parent 1:1 classid 1:11 htb rate 60kbps ceil 100kbps
// 随后建立filter指定哪些类型的packet进入那个class
# tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32 match ip src 1.2.3.4 match ip dport 80 0xffff flowid 1:10
# tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32 match ip src 1.2.3.4 flow 1:11
// 最后为这些class添加queuing disciplines
# tc qdisc add dev eth0 parent 1:10 handle 20: pfifo limit 5
# tc qdisc add dev eth0 parent 1:11 handle 30: sfq perturb 10
其他

// 同时模拟20Mbps带宽，50msRTT和0.1%丢包率
# tc qdisc add dev eth5 root handle 1:0 tbf rate 20mbit burst 10kb limit 300000
# tc qdisc add dev eth5 parent 1:0 handle 10:0 netem delay 50ms loss 0.1 limit 300000


top -H 显示线程信息
10进制转16进制：
　　$ echo "obase=16;ibase=10;100" | bc

netcat(nc)

telnet 

watch

ss

pidstat