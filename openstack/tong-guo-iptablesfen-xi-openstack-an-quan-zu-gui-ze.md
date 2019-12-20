#  

![](https://mmbiz.qpic.cn/mmbiz_png/3zoYydJjr10Ag5WmTicPwRPjaPdfvFKNDcLyOfbomJT1KM7JRsMHqfu0xcy9yWbpfjJHeHooh75TuwUlgZRWPEw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**在OpenStack中创建一个实例，同时会生成如下的bridge和port :**

·tap-xxx vm用的端口，配置在libvirt配置文件中的

·vnet-x图中有vnet，实际上是没有的，直接用tap插到了qbr上

·qbr-xxx虚拟网桥，桥接tap和qvb

·qvb-xxx连接br-int的veth端口

·qvo-xxx qvb-xxx的另一端

**例如：**

21:qbr14c032e9-bc: &lt;BROADCAST,MULTICAST,UP,LOWER\_UP&gt; mtu 1450 qdisc noqueuestate UP  
22: qvo14c032e9-bc@qvb14c032e9-bc:&lt;BROADCAST,MULTICAST,PROMISC,UP,LOWER\_UP&gt; mtu 1450 qdisc pfifo\_fastmaster ovs-system state UP qlen 1000

23:qvb14c032e9-bc@qvo14c032e9-bc: &lt;BROADCAST,MULTICAST,PROMISC,UP,LOWER\_UP&gt;mtu 1450 qdisc pfifo\_fast master qbr14c032e9-bc state UP qlen 1000

25:tap14c032e9-bc: &lt;BROADCAST,MULTICAST,UP,LOWER\_UP&gt; mtu 1450 qdiscpfifo\_fast master qbr14c032e9-bc state UNKNOWN qlen 500

**查看bridge:**

$ brctl show

**bridgename         bridgeid                STPenabled        interfaces**  
qbr14c032e9-bc8000.b21e7be143d6        no                qvb14c032e9-bc tap14c032e9-bc

分析Security Group规则

**分析FORWARD链**

由于规则是配置在host的，所以进出以上端口的规则都只走forward链

删除security group所有规则后，查看iptables

**\# iptables--list -v**

**Chain FORWARD**\(policyACCEPT0packets,0bytes\)  
** pkts bytes target     protopt in     out     source              destination**  
348K  19Mneutron-filter-top  all --  anyany   anywhere             anywhere             
348K  19Mneutron-openvswi-FORWARD  all --  anyany   anywhere             anywhere           

**Chainneutron-openvswi-FORWARD**\(1references\)  
** pkts bytes target     protopt in     out     source              destination**  
214 25538neutron-openvswi-sg-chain  all --  anyany   anywhere             anywhere            PHYSDEV match**--physdev-outtapc0a350e0-43**--physdev-is-bridged_/\* Direct trafficfrom the VM interface to the security group chain. \*/_  
248 23854neutron-openvswi-sg-chain  all --  anyany   anywhere             anywhere            PHYSDEV match --physdev-**in**tapc0a350e0-43--physdev-is-bridged_/\* Direct traffic from the VM interface tothe security group chain. \*/_

**Chainneutron-openvswi-sg-chain**\(4references\)  
** pkts bytes target     protopt in     out     source              destination**  
214 25538neutron-openvswi-ic0a350e0-4all -- anyany    anywhere            anywhere            PHYSDEV match**--physdev-out tapc0a350e0-43**--physdev-is-bridged_/\* Jump to the VM specific chain. \*/_  
248 23854neutron-openvswi-oc0a350e0-4all -- anyany    anywhere            anywhere             PHYSDEVmatch--physdev-**in**tapc0a350e0-43--physdev-is-bridged_/\* Jump to the VM specific chain. \*/_



**--physdev-out tapc0a350e0-43**是指tapc0a350e0-43即vm发送到bridge的包

匹配到的包跳转到了neutron-openvswi-ic0a350e0-4

**Chainneutron-openvswi-ic0a350e0-4**\(1references\)  
**num   pkts bytes target     prot opt in     out    source              destination**  
1169 20508RETURN     all  --  any    any    anywhere             anywhere           **state**RELATED,ESTABLISHED /\* Directpackets associated with a known session**to**the RETURN chain. \*/

22731RETURN    udp  --  any    any    192.168.1.2        anywhere             udpspt:bootps dpt:bootpc

300DROP       all  -- any    any     anywhere            anywhere            **state**INVALID /\* Drop packets that appear related**to**an existing connection\(e.g. TCP ACK/FIN\) but do not have an entry**in**conntrack. \*/

461944neutron-openvswi-sg-fallback  all  --  any    any    anywhere             anywhere            /\* Send unmatched traffic**to**the fallback chain. \*/

**Chainneutron-openvswi-sg-fallback**\(4references\)  
**num   pkts bytes target     prot opt in     out    source              destination**  
1947 71484DROP       all  --  any    any    anywhere             anywhere            /\* Default drop**rule for**unmatchedtraffic. \*/

·num:1放行所有已建立连接的包

·num:2放行192.168.1.2（dhcp服务器）发过来的udp包

·num:3丢弃状态异常的tcp包

·num:4丢弃不匹配以上三条的所有包



 --physdev-in tapc0a350e0-43是指从tapc0a350e0-43即vm发出来的包

匹配到的包跳转到了neutron-openvswi-oc0a350e0-4

**Chainneutron-openvswi-oc0a350e0-4**\(2references\)  
**num   pkts bytes target     prot opt in     out    source              destination**  
12648RETURN     udp  --  any    any    default              255.255.255.255    udp spt:bootpc dpt:bootps_/\* Allow DHCP client traffic. \*/_

2246 23206neutron-openvswi-sc0a350e0-4 all  --  any   any     anywhere            anywhere              
341272RETURN     udp  --  any    any    anywhere             anywhere            udp spt:bootpc dpt:bootps_/\* Allow DHCPclient traffic. \*/_

400DROP       udp  -- any    any     anywhere            anywhere             udp spt:bootpsdpt:bootpc_/\* Prevent DHCP Spoofing by VM. \*/_

5210 19802RETURN     all  --  any    any    anywhere             anywhere            state RELATED,ESTABLISHED_/\* Directpackets associated with a known session to the RETURN chain. \*/_

600DROP       all  -- any    any     anywhere            anywhere             state INVALID_/\* Droppackets that appear related to an existing connection \(e.g. TCP ACK/FIN\) but donot have an entry in conntrack. \*/_

700neutron-openvswi-sg-fallback  all --  any    any     anywhere            anywhere            _/\* Sendunmatched traffic to the fallback chain. \*/_Chainneutron-openvswi-sc0a350e0-4\(1references\)  
**num   pkts bytes target     prot opt in     out    source              destination**  
1246 23206RETURN     all  --  any    any   192.168.1.12        anywhere            MAC FA:16:3E:C3:EA:D5_/\* Allowtraffic from defined IP/MAC pairs. \*/_

200DROP       all  -- any    any     anywhere            anywhere            _/\* Drop trafficwithout an IP/MAC allow rule. \*/_Chain neutron-openvswi-sg-fallback \(4references\)  
**num   pkts bytes target     prot opt in     out    source              destination**  
1947 71484DROP       all  --  any    any    anywhere             anywhere           _/\* Default drop rule for unmatchedtraffic. \*/_

·num1允许vm发出来的dhcp udp广播包允许源端口是67，目标端口是68端口的数据包通过

·num2只允许ip地址为192.168.1.12（vm的分配的ip）通过

·num3允许vm（dhcp客户端）发出来的UDP单播报文

·num4禁止vm做dhcp嗅探

·num5允许通过所有已建立连接的包通过

·num6丢弃所以异常连接的包

·num7丢弃不匹配以上任何规则包

说明：

obootpc服务器向67端口\(bootpc\)广播dhcp回应请求

obootps客户端向68端口\(bootps\)广播dhcp请求配置

可以看出，在不匹配security规则的情况下，除了dhcp包可以通过之外，其他数据包全部丢弃



**配置securitygroup，新增规则后再查看iptables**

增加规则1：允许vm发出的所有数据包

查看neutron-openvswi-oc0a350e0-4链

**Chainneutron-openvswi-oc0a350e0-4**\(2references\)  
**num   pkts bytes target     prot opt in     out    source              destination**  
12648RETURN     udp  --  any    any    default              255.255.255.255    udp spt:bootpc dpt:bootps_/\* Allow DHCP client traffic. \*/_

2246 23206neutron-openvswi-sc0a350e0-4 all  --  any   any     anywhere            anywhere              
341272RETURN     udp  --  any    any    anywhere             anywhere            udp spt:bootpc dpt:bootps_/\* Allow DHCPclient traffic. \*/_

400DROP       udp  -- any    any     anywhere            anywhere             udp spt:bootpsdpt:bootpc_/\* Prevent DHCP Spoofing by VM. \*/_

5210 19802RETURN     all  --  any    any    anywhere             anywhere            state RELATED,ESTABLISHED_/\* Directpackets associated with a known session to the RETURN chain. \*/_

600RETURN     all  -- any    any     anywhere            anywhere              
700DROP       all  -- any    any     anywhere            anywhere             state INVALID_/\* Droppackets that appear related to an existing connection \(e.g. TCP ACK/FIN\) but donot have an entry in conntrack. \*/_

800neutron-openvswi-sg-fallback  all --  any    any     anywhere            anywhere            _/\* Sendunmatched traffic to the fallback chain. \*/_

·num6为新增的规则，为放行所有包



增加规则2：允许vm发出的icmp协议包通过

查看neutron-openvswi-ic0a350e0-4链

Chainneutron-openvswi-ic0a350e0-4\(1references\)  
num   pkts bytes target     prot opt in     out    source               destination         
1169 20508RETURN     all  --  any    any    anywhere             anywhere            state RELATED,ESTABLISHED_/\* Directpackets associated with a known session to the RETURN chain. \*/_

22731RETURN     udp  --  any    any   192.168.1.2         anywhere            udp spt:bootps dpt:bootpc

300RETURN     icmp --  any   any     anywhere            anywhere              
400DROP       all  -- any    any     anywhere            anywhere             state INVALID_/\* Droppackets that appear related to an existing connection \(e.g. TCP ACK/FIN\) but donot have an entry in conntrack. \*/_

561944neutron-openvswi-sg-fallback  all  --  any   any     anywhere            anywhere            _/\* Send unmatchedtraffic to the fallback chain. \*/_

·num3为新增的规则，放行所有icmp协议包



可以看到优化前在实时性方面原始的KVM还是会出现毛刺。在优化后基本上能达到很好的结果。

