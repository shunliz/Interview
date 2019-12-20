场景介绍：在我们的IDC中，存在着运行了3-6年的Ceph集群的服务器，这些服务器性能和容量等都已经无法满足当前业务的需求，在购入一批高性能机器后，希望将旧机器上的集群整体迁移到新机器上，当然，是保证**业务不中断**的前提下，再将旧机器下架回收。本文就介绍了一种实现业务不中断的数据迁移方案，并已经在多个生产环境执行。

> 本文的环境均为：Openstack+Ceph 运行虚拟机的场景，即主要使用RBD，不包含RGW，MDS。虚机的系统盘\(Nova\)，云硬盘\(Cinder\)，镜像盘\(Glance\)的块均保存在共享存储Ceph中。

## **环境准备**

> 本文环境为 Openstack \(Kilo\) + Ceph\(Jewel\)

本文所用的环境包含一套完整的 Openstack 环境，一套 Ceph 环境，其中 Nova/Cinder/Glance 均已经对接到了 Ceph 集群上，具体节点配置如下：

| 主机名 | IP地址 | Openstack 组件 | Ceph 组件 |
| :--- | :--- | :--- | :--- |
| con | 192.168.100.110 | nova,cinder,glance,neutron | mon，osd\*1 |
| com | 192.168.100.111 | nova,neutron | mon，osd\*1 |
| ceph | 192.168.100.112 |   | mon，osd\*1 |

在集群整体迁移完后，各个组件分布如下，也就是说，将运行于 con,com,ceph三个节点的 Ceph 集群迁移到 new\_mon\_1,new\_mon\_2,new\_mon\_3 这三台新机器上。

| 主机名 | IP地址 | Openstack 组件 | Ceph 组件 |
| :--- | :--- | :--- | :--- |
| con | 192.168.100.110 | nova,cinder,glance,neutron |   |
| com | 192.168.100.111 | nova,neutron |   |
| ceph | 192.168.100.112 |   |   |
| new\_mon\_1 | 192.168.100.113 |   | mon，osd\*1 |
| new\_mon\_2 | 192.168.100.114 |   | mon，osd\*1 |
| new\_mon\_3 | 192.168.100.115 |   | mon，osd\*1 |

在迁移之前，我们创建一个虚机，一个云盘，上传一个镜像，虚机此时正常运行，并将这这块云盘挂载到虚机上：

    [root@con ~(keystone_admin)]# nova list 

    +--------------------------------------+---------+--------+------------+-------------+-------------------------+

    | ID                                   | Name    | Status | Task State | Power State | Networks                |

    +--------------------------------------+---------+--------+------------+-------------+-------------------------+

    | 4f52191f-9645-448f-977b-80ca515387f7 | vm-test | ACTIVE | -          | Running     | provider=192.168.88.111 |

    +--------------------------------------+---------+--------+------------+-------------+-------------------------+

    [root@con ~(keystone_admin)]# cinder list 

    +--------------------------------------+--------+--------------+------+-------------+----------+--------------------------------------+

    |                  ID                  | Status | Display Name | Size | Volume Type | Bootable |             Attached to              |

    +--------------------------------------+--------+--------------+------+-------------+----------+--------------------------------------+

    | 39c76d96-0f95-490c-b7db-b3da6d17331b | in-use |  cinder-rbd  |  1   |     None    |  false   | 4f52191f-9645-448f-977b-80ca515387f7 |

    +--------------------------------------+--------+--------------+------+-------------+----------+--------------------------------------+

    [root@con ~(keystone_admin)]# ip netns exec `ip netns` ssh cirros@192.168.88.111

    cirros@192.168.88.111's password: 

    $ lsblk

    NAME   MAJ:MIN RM    SIZE RO TYPE MOUNTPOINT

    vda    253:0    0      1G  0 disk 

    `-vda1 253:1    0 1011.9M  0 part /

    vdb    253:16   0      1G  0 disk 

    $ 

Ceph 集群状态：

```
[root@ceph cluster]# ceph -s


    cluster 166889ab-fa7b-4a07-83da-6dfc92913a3d


     health HEALTH_OK


     monmap e47: 3 mons at {ceph=192.168.100.112:6789/0,com=192.168.100.111:6789/0,con=192.168.100.110:6789/0}


            election epoch 174, quorum 0,1,2 con,com,ceph


     osdmap e57: 3 osds: 3 up, 3 in


            flags sortbitwise,require_jewel_osds


      pgmap v6577: 768 pgs, 3 pools, 45659 kB data, 23 objects


            178 MB used, 766 GB / 766 GB avail


                 768 active+clean




[root@ceph cluster]# ceph osd tree


ID WEIGHT  TYPE NAME     UP/DOWN REWEIGHT PRIMARY-AFFINITY 


-1 0.74876 root default                                    


-2 0.24959     host ceph                                   


 0 0.24959         osd.0      up  1.00000          1.00000 


-3 0.24959     host con                                    


 1 0.24959         osd.1      up  1.00000          1.00000 


-4 0.24959     host com                                    


 2 0.24959         osd.2      up  1.00000          1.00000 




[root@ceph cluster]# ceph osd pool ls detail 


pool 1 'volumes' replicated size 2 min_size 1 crush_ruleset 0 object_hash rjenkins pg_num 256 pgp_num 256 last_change 58 flags hashpspool stripe_width 0


    removed_snaps [1~3]


pool 2 'vms' replicated size 2 min_size 1 crush_ruleset 0 object_hash rjenkins pg_num 256 pgp_num 256 last_change 59 flags hashpspool stripe_width 0


    removed_snaps [1~3]


pool 3 'images' replicated size 2 min_size 1 crush_ruleset 0 object_hash rjenkins pg_num 256 pgp_num 256 last_change 60 flags hashpspool stripe_width 0


    removed_snaps [1~9]
```

## **OSD 的数据迁移**

本次迁移主要分为两个组件的迁移，即 MON 和 OSD，这里我们先介绍 OSD 的数据迁移。相比迁移MON来说，OSD的数据迁移步骤更为单纯一些，因为所有操作均在 Ceph 侧执行，对 Openstack 来说是透明的。

### 原理简介

由于CRUSH算法的**伪随机性**，对于一个PG来说，如果 OSD tree 结构不变的话，它所分布在的 OSD 集合总是固定的（同一棵tree下的OSD结构不变/不增减），即对于两副本来说： PG 1.0 =&gt; \[osd.66, osd.33\]

* 当副本数减少时，PG 1.0 =&gt; \[osd.66\] ,也就是说会删除在osd.33上的第二副本，而在 osd.66上的主副本是维持不变的，所以在降低副本数时，底层OSD实际上只删除了一份副本，而并没有发生数据的迁移。

* 当副本数增加时，PG 1.0 =&gt; \[osd.66, osd.33, osd.188, osd.111\]，也就是说四副本的前两个副本依旧是之前的两个副本，而后面增加的两副本会从主副本osd.66将数据backfill到各自的OSD上。

### 迁移思路

1. 我们首先会将新的节点的所有OSD初始化完毕，然后将这些OSD **加入到另一棵osd tree下** \(这里简称原先的集群的osd tree叫做old\_tree，新建的包含新OSD的 osd tree 叫做 new\_tree\)，这样部署完毕后，不会对原有集群有任何影响，也不会涉及到数据迁移的问题，此时新的OSD下还没有保存数据。

2. 导出 CRUSHMAP，编辑，添加三条 CRUSH rule：

3. •  crush rule 0 \(原先默认生成的\): 从 old\_tree 下选出size副本（这里size=2\)。

4. •  crush rule 1 \(新生成的第一条\): 从 old\_tree 下选出两副本。对于副本数为2的集群来说，crush\_rule\_0 和 crush\_rule\_1 选出的两副本是一样的。

5. •  crush rule 2 \(新生成的第二条\): 从 old\_tree 下选出两副本，再从 new\_tree 下选出两副本。由\*\*原理简介第二段\*\*可知， 由于 old\_tree 下面的 OSD结构不变也没有增加，所以 crush\_rule\_0 和 crush\_rule\_1 选出的前两副本是\*\*一样的\*\*。

6. •  crush rule 3 \(新生成的第三条）: 从 new\_tree 下选出两副本。 由于crush\_rule\_1 的第二次选择为选出 new\_tree下的前两副本，这和 crush\_rule\_2 选出两副本（也是前两副本）其实是\*\*一样的\*\*。

7. 注入新的CRUSHMAP后，我们做如下操作：

   • 将所有pool\(当然建议一个pool一个pool来，后面类似\) 的 CRUSH RULE 从  crush\_rule\_0 设置为 crush\_rule\_1 ，此时所有PG均保持active+clean，状态没有任何变化。

     •  将所有pool的副本数设置为4， 由于此时各个 pool 的CRUSH RULE 均为 crush\_rule\_1 ，而这个 RULE 只能选出两副本，剩下两副本不会被选出，所以此时所有PG状态在重新 peer 之后，变为 active + undersized + degraded，由于不会生成新的三四副本，所以集群没有任何数据迁移\(backfill\) 动作，此步骤耗时短暂。

    •  将所有pool的 CRUSH RULE 从 crush\_rule\_1 设置为 crush\_rule\_2，此时上一条动作中没有选出的三四副本会从 new\_tree 下选出，并且原先的两副本不会发生任何迁移，整个过程宏观来看就是在 old\_tree -&gt; new\_tree 单向数据复制克隆生成了新的两副本，而旧的两副本没有移动。此时生成新的两副本耗时较长，取决于磁盘性能带宽数据量等，可能需要几天到一周的时间，所有数据恢复完毕后，集群所有PG变为 active+clean 状态。

     •  将所有pool的 CRUSH RULE 从 crush\_rule\_2 设置为 crush\_rule\_3，此时所有PG状态会变为 active+remapped，发生的另一个动作是，原先四副本的PG的主副本是在 old\_tree 上的某一个OSD上的，现在这个PG的主副本变为 new\_tree下的选出的第一个副本，也就是发生了主副本的切换。比如原先 PG 1.0 =&gt; \[osd.a, osd.b, osd.c, osd.d\] 在这步骤之后会变成 PG 1.0 =&gt; \[osd.c, osd.d\]。原先的第三副本也就是new\_tree下的第一副本升级为主副本。

    •  将所有pool的副本数设置为2，此时PG状态会很快变为 active+clean，然后在OSD层开始删除 old\_tree 下的所有数据。此时数据已经全部迁移到新的OSD上。

### 迁移指令

#### 1.初始化新的OSD

一定要记住：**在部署目录下的ceph.conf内添加 osd\_crush\_update\_on\_start =false**，再开始部署新的OSD，并且一定要检查新节点配置，包括但不限于： yum 源，免秘钥配置，ceph的版本，主机名，防火墙，selinux，**ntp**，**ntp**，**ntp**，重要的时间对齐说三遍！

部署新的OSD：

```
### 前往部署目录


cd /root/cluster


echo "osd_crush_update_on_start = false " 
>
>
 ceph.conf 


ceph-deploy --overwrite-conf osd prepare new_mon_1:sdb  new_mon_2:sdb  new_mon_3:sdb --zap-disk


ceph-deploy --overwrite-conf osd activate  new_mon_1:sdb1  new_mon_2:sdb1  new_mon_3:sdb1
```

添加完这三个OSD后，集群总共有六个OSD，此时不会有数据迁移。结构如下：

```
[root@ceph ~]# ceph osd tree


ID WEIGHT  TYPE NAME     UP/DOWN REWEIGHT PRIMARY-AFFINITY 


-1 0.75000 root default                                    


-2 0.25000     host ceph                                   


 0 0.25000         osd.0      up  1.00000          1.00000 


-3 0.25000     host con                                    


 1 0.25000         osd.1      up  1.00000          1.00000 


-4 0.25000     host com                                    


 2 0.25000         osd.2      up  1.00000          1.00000 


 3       0 osd.3              up  1.00000          1.00000 


 4       0 osd.4              up  1.00000          1.00000 


 5       0 osd.5              up  1.00000          1.00000
```

构建新的new\_root根节点，并将这三个新的OSD加入到新的根节点下\(注意OSD和主机的物理对应关系\)：

```
ceph osd crush add-bucket new_root  root


ceph osd crush add-bucket new_mon_1 host


ceph osd crush add-bucket new_mon_2 host


ceph osd crush add-bucket new_mon_3 host


ceph osd crush move new_mon_1 root=new_root


ceph osd crush move new_mon_2 root=new_root


ceph osd crush move new_mon_3 root=new_root


ceph osd crush add osd.3 0.25 host=new_mon_1


ceph osd crush add osd.4 0.25 host=new_mon_2


ceph osd crush add osd.5 0.25 host=new_mon_3
```

此时，新的 TREE 结构如下：

```
[root@ceph ~]# ceph osd tree


ID WEIGHT  TYPE NAME          UP/DOWN REWEIGHT PRIMARY-AFFINITY 


-5 0.75000 root new_root                                        


-6 0.25000     host new_mon_1                                   


 3 0.25000         osd.3           up  1.00000          1.00000 


-7 0.25000     host new_mon_2                                   


 4 0.25000         osd.4           up  1.00000          1.00000 


-8 0.25000     host new_mon_3                                   


 5 0.25000         osd.5           up  1.00000          1.00000 


-1 0.75000 root default                                         


-2 0.25000     host ceph                                        


 0 0.25000         osd.0           up  1.00000          1.00000 


-3 0.25000     host con                                         


 1 0.25000         osd.1           up  1.00000          1.00000 


-4 0.25000     host com                                         


 2 0.25000         osd.2           up  1.00000          1.00000 
```

#### 2.编辑 CRUSH MAP

导出CRUSH MAP，并编辑添加三条新的 CRUSH RULE：

```
### 导出CRUSH MAP


ceph osd getcrushmap -o map 


crushtool -d map -o map.txt


### 在map.txt 最后添加以下内容


vim map.txt 




# rules


rule replicated_ruleset {


        ruleset 0


        type replicated


        min_size 1


        max_size 10


        step take default


        step chooseleaf firstn 0 type host


        step emit


}


>
>
>
>
>
>
>
>
>
>
>
>
>
>
 添加开始  
>
>
>
>
>
>
>
>
>
>


rule replicated_ruleset1 {


        ruleset 1


        type replicated


        min_size 1


        max_size 10


        step take default


        step chooseleaf firstn 2 type host


        step emit


}


rule replicated_ruleset2 {


        ruleset 2


        type replicated


        min_size 1


        max_size 10


        step take default


        step chooseleaf firstn 2 type host


        step emit


        step take new_root


        step chooseleaf firstn 2 type host


        step emit


}


rule replicated_ruleset3 {


        ruleset 3


        type replicated


        min_size 1


        max_size 10


        step take new_root


        step chooseleaf firstn 2 type host


        step emit


}


<
<
<
<
<
<
<
<
<
<
<
<
 添加结束  
<
<
<
<
<
<
<
<
<
<
<
<


# end crush map




### 编译 CRUSH MAP，并注入到集群中




crushtool -c map.txt -o map.bin


ceph osd setcrushmap -i map.bin
```

#### 3. 开始迁移数据

到目前为止的所有操作均不会发生数据迁移，对线上业务也就没有影响，下面我们要开始通过修改 POOL 的 CRUSH RULESET 和 副本数来实现数据的整体迁移\(这里我们取 volumes 池来介绍\)：

```
###确认当前 volumes 池使用的是 crush_ruleset 0


[root@ceph cluster]# ceph osd pool get volumes crush_ruleset 


crush_ruleset: 0




### 将 volumes 池的 crush_ruleset 设置为 1，设置完后，集群一切正常，PG均为active+clean


[root@ceph cluster]# ceph osd pool set volumes crush_ruleset 1


set pool 1 crush_ruleset to 1




### 将 volumes 池的 副本数设置为4， 设置完后，PG经过短暂 Peer，变为 active+undersized+degraded


[root@ceph cluster]# ceph osd pool set volumes size 4


set pool 1 size to 4




[root@ceph cluster]# ceph -s


    cluster 166889ab-fa7b-4a07-83da-6dfc92913a3d


     health HEALTH_WARN


            256 pgs degraded


            256 pgs undersized


            recovery 183984/368006 objects degraded (49.995%)


     monmap e47: 3 mons at {ceph=192.168.100.112:6789/0,com=192.168.100.111:6789/0,con=192.168.100.110:6789/0}


            election epoch 182, quorum 0,1,2 con,com,ceph


     osdmap e106: 6 osds: 6 up, 6 in


            flags sortbitwise,require_jewel_osds


      pgmap v23391: 768 pgs, 3 pools, 403 MB data, 92011 objects


            1523 MB used, 1532 GB / 1533 GB avail


            183984/368006 objects degraded (49.995%)


                 512 active+clean


                 256 active+undersized+degraded






[root@ceph cluster]# ceph osd pool set volumes crush_ruleset 2


set pool 1 crush_ruleset to 2




### 等到 volumes 池的所有PG均变为 active+clean 后，将 volumes 池的 crush_ruleset 设置为3， 此步骤后，volumes 池的 PG状态变为 active+remapped。但是不影响集群IO。




[root@ceph cluster]# ceph osd pool set volumes crush_ruleset 2


set pool 1 crush_ruleset to 2




set pool 1 crush_ruleset to 3




### 此时，将 volumes 池的 副本数 设置为2 ，此时 PG 状态经过 peer 之后，很快变为 active+clean ，volumes 池的两副本均落在新的节点下，并且后台在自行删除之前旧节点上的数据。




[root@ceph cluster]# ceph osd pool set volumes size  2
```

### **注意事项**

由于本次数据迁移是在生产环境上执行的，所以没有直接执行将数据从旧节点直接 `mv`到新节点，而是选择了执行步骤较为复杂的上面的方案，先 `scp` 到新节点，再 `rm`掉旧节点的数据。并且每一个步骤都是可以快速回退到上一步的状态的，对于生产环境的操作，是比较友好的。在本次方案测试过程中，遇到了如下的一些问题，需要引起充分的注意：

* Ceph 版本不一致： 由于旧的节点的 Ceph 版本为 0.94.5 ，而新节点安装了较新版本的 10.2.7， 在副本 2=&gt;4 的过程中Peer是正常的，而将池的crush\_ruleset 设置为3 ，也就是将新节点的 PG 升级为主副本后，PG由新节点向旧节点发生Peer，此时会一直卡住，PG始终卡在了 `remapped + peering`，导致该 pool 无法IO。在将**新旧节点 Ceph 版本一致**后\(旧节点升级，新节点降级\)，此现象得以消除。 为了确保此现象不会在实际操作中发生，应该在变更之前，新建一个测试pool，对其写入部分数据，再执行所有数据迁移指令，查看此过程是否顺畅，确认无误后，再对生产pool进行操作！

* 新节点 OSD 的重启问题： 如果没有添加了`osd_crush_update_on_start` 这个配置参数，那么当新节点的OSD重启后，会自动添加到默认的 `root=default` 下，然后立刻产生数据迁移，因此需要添加这个参数，保证OSD始终位于我们人为指定的节点下，并不受重启影响。这是个基本的Ceph运维常识，但是一旦遗忘了，可能造成较为严重的影响。更不用说防火墙时钟这些配置了。

* 集群性能降低：总体来说，在副本从2克隆为4这段时间\(约2-3天，取决于集群数据量）内，集群的实际IO表现降低到变更前的 25%-&gt;80% 左右，时间越往后表现越接近变更前，这虽然不会导致客户端的IO阻塞，但从客户反馈来看，可以感知到较为明显的卡顿。因此克隆时间应该选择业务量较低的节假日等。

* 新节点IP不`cluster_network`范围内：这个比较好解决，只需要增大部署目录`ceph.conf`内的`cluster_network` 或者 `public_network`的掩码范围即可，不需要修改旧节点的，当然网络还是要通的。

* 变更的回退：对于生产环境来说，尽管执行步骤几乎是严谨不会出错的，但是难免会遇到意外情况，就比如上面的版本不一致导致的 peer 卡住现象。因此我们需要制定完善的回退步骤，在意外发生的时候，能够快速将环境回退到上一步集群正常的状况，而不是在意外发生时惊出一身冷汗，双手颤抖得敲指令。。。所以，下面的表格给出了这个变更操作的每一步的回退步骤：

| 变更步骤 | 实际影响 | 回退指令 | 回退影响 |
| :--- | :--- | :--- | :--- |
| ceph osd pool set volumes crush\_ruleset 1 | 无 | ceph osd pool set volumes crush\_ruleset 0 | 无 |
| ceph osd pool set volumes size 4 | PG由active+clean，变为 active+undersized+degraded | ceph osd pool set volumes size 2 | PG很快恢复active+clean |
| ceph osd pool set volumes crush\_ruleset 2 | PG 开始 backfill，需要较长时间 | ceph osd pool set volumes crush\_ruleset 1 | PG很快恢复到active+undersized+degraded，并删除在新节点生成的数据。 |
| ceph osd pool set volumes crush\_ruleset 3 | PG 很快变为 active+remapped。 | ceph osd pool set volumes crush\_ruleset2 | PG很快恢复到 active+clean |
| ceph osd pool set volumes size 2 | PG 很快变为 active+clean，并且后台删除旧节点数据。 | ceph osd pool set volumes size 4 | PG 变为active+remapped。 |

> 为何不直接迁移数据到新节点？总体来看数据迁移过程，最耗时的地方是数据从两副本变为四副本的过程，其余过程是短暂且可以快速回退的，而如果直接将池的 crush\_ruleset 设置为 3 ，数据开始从旧节点直接backfill到新节点上，其实这么做也是可以的，但是这里我们就会遇到一个问题，一旦数据复制了一天或者一段时间后，集群出现了问题，比如性能骤降，要求必须回退到操作之前的状态，此时已经有部分PG完成了迁移，也就是说旧节点上的两副本已经删除了，那么回退到上一步后，还会发生数据从新节点向旧节点的复制，那么之前复制了多久的数据，可能就需要多久来恢复旧节点上删除的数据，这很不友好。而用了我们的方法来复制数据的话，不会存在这个问题，因为这里的方法在最后一步将池的副本设置为2之前，旧节点上的数据始终都是在的，并且不会发生任何迁移，我们可以在任意意外情况下，通过几条指令将集群恢复到变更之前的状态。

## **MON 的迁移**

### 原理介绍

相比于 OSD 的数据迁移，MON 的迁移比较省时省力一些，步骤相对简单，但是里面涉及的原理比较复杂，操作也需要细心又细心。

首先，我们的环境为典型的 Openstack+Ceph的环境，其中 Openstack 的三个组件： Nova/Cinder/Glance 均已经对接到了Ceph集群中，也就是说虚机系统盘，云硬盘，镜像都保存在Ceph中。而这三个客户端调用Ceph的方式不太一样：

* Glance ：上传下载镜像等时，需要新建一个调用 librbd 的 Client 来连接 Ceph集群。

* Cinder ：

* * 创建删除云盘时，新建一个调用 librbd 的 Client 来连接 Ceph 集群。

  * 挂载卸载云盘时，由Nova调用librbd来实现该操作。
* Nova ： 虚机\(qemu-kvm进程\)相当于一个始终在调用librbd的Client，并且进程始终都在。

我们需要知道的是，当一个 Client需要连接 Ceph 集群时，它首先通过自己的用户名和秘钥（client.cinder/client.nova...\) 来连接到`/etc/ceph/ceph.conf`配置文件指定IP的MON，认证成功后，可以获取集群的很多MAP\( monmap,osdmap,crushmap...\)，通过这些 MAP，即可向 Ceph 集群读取数据。

对于一个虚机进程\(qemu-kvm\)来说，虚机启动之初，它即获取到了集群的 monmap， 而当所连接 MON 的 IP 变化时，比如这个 MON 挂掉时，它便会尝试连接 monmap 里面的其他 IP 的 MON，如果每个MON都挂了，那么这个 Client 就不能连接上集群获取最新的 monmap，osdmap等。下面我们以一个 pid为3171的 qemu-kvm 进程来演示这一过程：

```
[root@con ~(keystone_admin)]# ps -ef|grep kvm 


qemu        3171       1 17 14:32 ?        00:31:08 /usr/libexec/qemu-kvm -name guest=instance-0000000b.........




[root@con ~(keystone_admin)]# netstat -tnp |grep 3171|grep 6789


tcp        0      0 192.168.100.110:59926   192.168.100.112:6789    ESTABLISHED 3171/qemu-kvm 
```

可以看到，这个进程连接着IP为`192.168.100.112`的 MON，而我们手动将这个IP的 MON 停掉，则会发现这个进程又连接到了剩余两个IP的MON之一上：

```
[root@con ~(keystone_admin)]# ssh 192.168.100.112 systemctl stop ceph-mon.target




[root@con ~(keystone_admin)]# netstat -tnp |grep 3171|grep 6789


tcp        0      0 192.168.100.110:48792   192.168.100.111:6789    ESTABLISHED 3171/qemu-kvm  
```

因此，如果我们每次都**增加一个MON，再删除一个MON**，那么在删除一个MON之后，之前连接到这个MON上的 Client 会自动连接到一个其他MON，并且再获取最新的monmap。那么我们增删过程就是：

* 原先有三个MON： con， com， ceph

* 增加 new\_mon\_1，变为四个： con， com， ceph， new\_mon\_1

* 删除con，变为三个：com， ceph， new\_mon\_1

* 增加 new\_mon\_2，变为四个： com， ceph， new\_mon\_1， new\_mon\_2

* 删除com，变为三个：ceph， new\_mon\_1， new\_mon\_2

* 增加 new\_mon\_3，变为四个： ceph， new\_mon\_1， new\_mon\_2， new\_mon\_3

* 删除con，变为三个：new\_mon\_1， new\_mon\_2， new\_mon\_3

#### Nova 侧的一个问题

在实际操作中，发现了一个问题，会导致虚机无法重启等问题。

当虚机挂载一个云硬盘时，Nova 会将挂载这个云盘时所连接的MON IP 写入到数据库中，而在修改完MON的IP后，新的MON IP**不会**被更新到数据库中，而虚机启动时会加载 XML 文件，这个文件由数据库对应字段生成，由于没有更新 MON IP，所以 qemu-kvm 进程在启动时，会尝试向旧的MON IP发起连接请求，当然，旧MON已经删除，导致连接不上而卡住，最终致使虚机进程启动了，但是虚机状态始终不能更新为 RUNNING。

可以通过打开客户端的`ceph.conf` 内的`debug_rbd=20/20`，查看qemu-kvm进程调用librbd时生成的log发现进程在启动时始终尝试连接旧的MON IP。

这里，我们只能手动修改数据库中记录的IP地址来确保虚机重启后能够连接上新的MON，需要注意的是，仅仅修改虚机XML文件是无法生效的，因为会被数据库内的字段覆盖而连上旧MON：

```
### 具体字段为： 


mysql =
>


nova  =
>
 block_device_mapping =
>
 connection_info




*************************** 23. row ***************************


           created_at: 2018-03-19 08:50:59


           updated_at: 2018-03-26 06:32:06


           deleted_at: 2018-03-26 09:20:02


                   id: 29


          device_name: /dev/vdb


delete_on_termination: 0


          snapshot_id: NULL


            volume_id: 39c76d96-0f95-490c-b7db-b3da6d17331b


          volume_size: NULL


            no_device: NULL


      connection_info: {"driver_volume_type": "rbd", "serial": "39c76d96-0f95-490c-b7db-b3da6d17331b", "data": {"secret_type": "ceph", "name": "volumes/volume-39c76d96-0f95-490c-b7db-b3da6d17331b", "secret_uuid": "0668cc5e-7145-4b27-8c83-6c28e1353e83", "qos_specs": null, "hosts": ["192.168.100.110", "192.168.100.111", "192.168.100.112"], "auth_enabled": true, "access_mode": "rw", "auth_username": "cinder", "ports": ["6789", "6789", "6789"]}}


        instance_uuid: 4f52191f-9645-448f-977b-80ca515387f7


              deleted: 29


          source_type: volume


     destination_type: volume


         guest_format: NULL


          device_type: disk


             disk_bus: virtio


           boot_index: NULL


             image_id: NULL
```

### 迁移指令

这里，我们使用 ceph-deploy 来增删 MON 节点，主要是为了操作的简洁和安全性着想。

首先，我们需要将新的三个MON的IP地址加入到**所有节点**的`/etc/ceph/ceph.conf`的`mon_host` 字段中。保证此时`mon_host`内是六个MON的IP地址。

```
### 添加 new_mon_1 


[root@ceph cluster]# ceph-deploy mon add new_mon_1


[root@ceph cluster]# ceph -s


    cluster 166889ab-fa7b-4a07-83da-6dfc92913a3d


     health HEALTH_OK


     monmap e48: 4 mons at {ceph=192.168.100.112:6789/0,com=192.168.100.111:6789/0,con=192.168.100.110:6789/0,new_mon_1=192.168.100.113:6789/0}




### 删除 con  删除完后，建议等待1-3min再添加新的MON，使得qemu-kvm进程可以建立新的socket链接。


[root@ceph cluster]# ceph-deploy mon destroy con


[root@ceph cluster]# ceph -s


    cluster 166889ab-fa7b-4a07-83da-6dfc92913a3d


     health HEALTH_OK


     monmap e49: 3 mons at {ceph=192.168.100.112:6789/0,com=192.168.100.111:6789/0,new_mon_1=192.168.100.113:6789/0}




### 依次添加 new_mon_2 ，删除 com，添加 new_mon_3 ， 删除 ceph：


[root@ceph cluster]# ceph-deploy mon add new_mon_2


[root@ceph cluster]# ceph-deploy mon destroy com 


### 等待1-3min，


[root@ceph cluster]# ceph-deploy mon add new_mon_3


[root@ceph cluster]# ceph-deploy mon destroy ceph




[root@con ~(keystone_admin)]# ceph -s


cluster 166889ab-fa7b-4a07-83da-6dfc92913a3d


     health HEALTH_OK


     monmap e53: 3 mons at {new_mon_1=192.168.100.113:6789/0,new_mon_2=192.168.100.114:6789/0,new_mon_3=192.168.100.115:6789/0}
```

此时，将**所有节点**`/etc/ceph/ceph.conf`内的`mon_host`字段中原先的MON删除，**只保留三个新的MON IP地址**。

#### Glance & Cinder & Nova 服务重启

无需重启 Glance 服务。

需要重启**所有计算节点**的 nova-compute 和**控制节点**的 Cinder 服务，否则会导致虚机无法创建等问题。

```
## 在计算节点


openstack-service restart nova-compute


## 在控制节点


openstack-service restart cinder
```

#### Nova

由于 nova 不会更新之前的已经挂载的磁盘所连接的MON IP 信息，这会导致虚机在重启等动作时，尝试连接到旧的已经被摧毁的MON的地址，导致动作卡住，因此这里要单独改一下数据库内的MON IP 信息：

这里我们将 192.168.100.110/111/112 改为 192.168.100.113/114/115

```
mysql 
>
>




use nova;


update  block_device_mapping set connection_info=(replace(connection_info,'192.168.100.110','192.168.100.113'))；


update  block_device_mapping set connection_info=(replace(connection_info,'192.168.100.111','192.168.100.114'))；


update  block_device_mapping set connection_info=(replace(connection_info,'192.168.100.112','192.168.100.115'))；




exit;
```

更新完数据库后，这次数据迁移算是大功告成了。为何不需要重启虚机服务呢，这里再做一些简单的介绍：

qemu-kvm 虚机进程是一个长连接的 Ceph Client，自虚机启动后，进程就和 Ceph 集群保持着连接，在我们更改 MON 后，这些 Client 会自动尝试重连 monmap 里面的其他MON，从而更新 monmap， 来获取最新的 MON 。修改`/etc/ceph/ceph.conf`不会对虚机进程产生影响，除非虚机重启等，但是，虚机可以通过更新 monmap 的方式来感知集群MON的改变。

## **参考文档**

* \[如何更改基于rbd块设备的虚机的monitor ip\] \[https://opengers.github.io/openstack/how-to-change-guest-monitor-ip-with-rbd-disk/\#%E6%9F%A5%E7%9C%8B%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%BD%93%E5%89%8D%E8%BF%9E%E6%8E%A5%E7%9A%84monitor-ip\]

* \[Ceph Monitor hardcoded IPs in Nova database\] \[https://bugzilla.redhat.com/show\_bug.cgi?id=1414124\]



