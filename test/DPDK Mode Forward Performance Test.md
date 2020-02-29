
# DPDK Mode Forward Performance Test Report
## Overview                                 
	OpenContrail support DPDK mode since version 2.2. This report is base on DPDK mode, and the tunnel type is MPLS Over UDP.





2.2硬件

服务器	设备类型	数量	CPU	内存	网卡	用途
Dell R730	物理机	1	12核 E5-2620v3	32GB	4*1G+2*10G(光口)	控制节点
浪潮	物理机	2	20核 E5-2690_v2	128GB	2*1G+2*10G(光口)	计算节点

3测试版本信息

测试的主版本信息：
版本号	发布日期	版本状态
3.0.2.1-16081855.el7.centos	2016-08-18	■正式发布版本

 

4测试工具
测试工具名称	介绍
Pketgen	内核发包工具，使用pktgen 持续发包，查看contrail vrouter的转发性能


5测试方法
5.1BIOS设置
测试DPDK模式的转发需要设置计算节点BIOS，关闭以下几项。
Hyperthreading							disable
Hardware Prefetcher					disable
Adjacent Cable Line Prefetch			disable
NUMA Optimized						disable
SATA Mode							Compatablity

5.2Hugepage 设置
使用hugepage size: 2M, 另外计算节点需要设置hugepage的个数，计算方法：系统总的hugepage个数=vrouter-dpdk固定个数(39)x2(该计算节点有两个node)+DPDK socket mem(1024)+虚拟机所需的个数。 Nova调用libvirt创建虚拟机，需要用到hugepage, 默认mount的路径为/dev/hugepages。
设置如下：
mount -t hugetlbfs nodev /dev/hugepages
添加“nodev /dev/hugepages hugetlbfs defaults 0 0”到 /etc/fstab file
编辑/etc/sysctl.conf 设置vm.nr_hugepages 
5.3Coremask设置
经过多次测试，使用0xfc3f0 和 0xfff0 丢包最少，设置方法：
修改/etc/contrail/supervisord_vrouter_files/contrail-vrouter-dpdk.ini设置taskset 为0xfc3f0
Htop 查看taskset 结果如下

5.4Contrail-Vrouter 设置
修改/etc/contrail/contrail-vrouter-agent.conf 设置
[FLOWS]
thread_count=16
VM之间可以有16个thread建立flow。

5.5虚拟机信息
建立虚拟机之前需要创建支持dpdk 的flavor：
nova flavor-create m2.large.hpgs auto 2000 20 2
nova flavor-key m2.large.hpgs set hw:mem_page_size=large
nova flavor-key m2.large.hpgs set aggregate_instance_extra_specs:hpgs=true

镜像：ubuntu-12.04.2_qcow2
VCPU数量:4 VCPU
磁盘:80GB
云主机类型名称:m1.large.hpgs
内存:7.9GB

5.6Pktgen脚本设置

#! /bin/sh

#modprobe pktgen


function pgset() {
    local result

    echo $1 > $PGDEV

    result=`cat $PGDEV | fgrep "Result: OK:"`
    if [ "$result" = "" ]; then
         cat $PGDEV | fgrep Result:
    fi
}

function pg() {
    echo inject > $PGDEV
    cat $PGDEV
}

# Config Start Here -----------------------------------------------------------


# 总共4个CPU，每一个都设置到eth1上,eth1为转发网卡

PGDEV=/proc/net/pktgen/kpktgend_0
  echo "Removing all devices"
 pgset "rem_device_all" 
  echo "Adding eth1"
 pgset "add_device eth1" 
#  echo "Setting max_before_softirq 10000"
# pgset "max_before_softirq 10000"


PGDEV=/proc/net/pktgen/kpktgend_1
  echo "Removing all devices"
 pgset "rem_device_all" 
  echo "Adding eth1"
 pgset "add_device eth1" 
#  echo "Setting max_before_softirq 10000"
# pgset "max_before_softirq 10000"

PGDEV=/proc/net/pktgen/kpktgend_2
  echo "Removing all devices"
 pgset "rem_device_all" 
  echo "Adding eth1"
 pgset "add_device eth1" 
#  echo "Setting max_before_softirq 10000"
# pgset "max_before_softirq 10000"

PGDEV=/proc/net/pktgen/kpktgend_3
  echo "Removing all devices"
 pgset "rem_device_all" 
  echo "Adding eth1"
 pgset "add_device eth1" 
#  echo "Setting max_before_softirq 10000"
# pgset "max_before_softirq 10000"

# device config
# ipg is inter packet gap. 0 means maximum speed.

CLONE_SKB="clone_skb 0"
# NIC adds 4 bytes CRC
PKT_SIZE="pkt_size 64" #包大小

# COUNT 0 means forever
#COUNT="count 0"
COUNT="count 1000000" #包个数

PGDEV=/proc/net/pktgen/eth1
  echo "Configuring $PGDEV"
 pgset "$COUNT"
 pgset "$CLONE_SKB"
 pgset "$PKT_SIZE"
 pgset "dst 192.168.20.3" 
 pgset "dst_mac 02:73:00:d9:6e:be" #目标MAC
 pgset "udp_dst_min 60000"
 pgset "udp_dst_max 60016" #设置16个UDP口，创建16个flow
 pgset "udp_src_min 50100"
 pgset "udp_src_max 50100"


# Time to run
PGDEV=/proc/net/pktgen/pgctrl

 echo "Running... ctrl^C to stop"
 pgset "start" 
 echo "Done"

# Result can be vieved in /proc/net/pktgen/eth1


6测试结果


6.1 单向打流VM1-->VM2
pkt-size	vm1发包	Vrouter发包	Vrouter收包	vm2收包	Vrouter丢包	物理及系统丢包	发送速率	pps	丢包率
64	400000000	400000000	399997163	373549400	26447764	2837	399Mb/s	780160	0.0661265
64	400000000	400000000	399996924	328599530	71397394	3076	464Mb/s	907600	0.178501175
64	400000000	400000000	399959653	394274257	5685396	40347	457Mb/s	893336	0.014314358
									
1468	400000000	400000000	399999418	399885057	114361	582	8564Mb/s	729295	0.000287358
1468	400000000	400000000	399999502	399937439	62063	498	7649Mb/s	651385	0.000156403
1468	400000000	400000000	399999482	399972074	27409	518	8772Mb/s	747013	0.000069815
1468	400000000	400000000	399999305	399968190	31115	695	8147Mb/s	693716	0.000079525
1468	400000000	400000000	399999405	399924583	74822	595	8530Mb/s	726352	0.000188543


6.2 双向打流VM1<-->VM2

6.3 两对双向打流VM1<-->VM2，VM3<-->VM4

 			
