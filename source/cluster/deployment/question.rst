.. _cluster_question:

======================
集群常见故障及解决方案
======================

.. contents:: 目录


介绍
==========

本文档主要记录集群使用过程中所遇到的故障,方便后续维护和使用。

.. ========  =====
.. 故障评级   B  
.. ========  =====
.. False     False
.. True      False
.. ========  =====


.. csv-table:: 故障评级说明
   :header: "故障等级","定义", "说明"
   :widths: 20, 20, 50
   :align: center

   "P1", "重大故障","VPN 不能正常连接，集群出现大面积宕机，系统完全不可用，
   
   宕机时长大于5h并且未能正常恢复"
   "P2", "严重故障", "网络出现故障导致 VPN不能正常连接，
   
   集群未出现宕机情况在小于5个小时内恢复状态"
   "P3", "一般故障", "VPN 能正常连接，大部分主机处于正常工作状态，
   
   少数主机网络不可达或者宕机，1个小时内正常恢复"
   "P4", "轻微故障", "集群整体状态良好，连接速度慢，
   
   或者偶尔断开连接，能通过重连恢复"



故障日志
==========

1. 时间：2021年3月28日

    **故障描述**: vpn 连接不上, 但集群内主机可以正常上网。 (P3故障)

    **排查**: 集群可以正常上网说明集群的联网状态正常那么可能 vpn 服务可能异常。尝试登录 vpn 所在主机, 发现未能正常连接, 说明 vpn 所在主机可能联网状态异常。
    检查网线接口,发现出现松动😂。

    **解决方法**: 重新插拔网线即可。 

.. _2021_03_29_log:

2. 时间：2021年3月29日

    **故障描述**: vpn能正常连接，其中一台（host2）连接不上。(P3故障)

    **排查**: 说明是host2出现故障，ping 后发现不可达，有可能是host2资源被占满，或者网络连接错误导致无法响应。
    此时 host2 负载应该较小，排除第一种情况；同时 host2 主机前面板网络连线指示灯正常闪烁，说明网线也是正常连接。
    登入路由器，查看 host2 是否在线，发现 host2 并未出现在 ``Active DHCP Leases`` 中，说明 host2 未能正确获得 IP。

    **解决方法**: 重新插拔网线，使其主动发起 dhcp 请求即可。

3. 时间：2021年4月13日
   
    **故障描述**: 集群主机不能正常连如校园网，并且主机不能正常DHCP。（P1故障）
    
    **排查**：不能正常进入登录界面 aaa.uestc.edu.cn，使用 IP（10.253.0.237）后可以正常登录，但是集群依旧不能正常上网，怀疑是 DNS 错误，将主机的 DNS 手动设置后可以正		常上网。
    集群主机不能正常获得DHCP，但是主机手动设置 IP 后一切正常，进入路由器界面 DHCP 活跃主机的确没有列表，重启 lan 口、路由器以、交换机以及插拔主机网线依旧存在这个问题。暂定认为路由器故障

    **解决方法**：

    针对认证后不能正常上网可以有以下解决办法:

        (1).  初步怀疑是登录的账号比较新，从未登录过，切换使用比较活跃的账号。

        .. note::
            虽然这个现象非常不可思议😓，但是切换账号后的确生效，但是目前不能完全保证是这个原因，可以尝试下面的方法。
        
        (2). 手动在主机上设置  DNS 地址为 ``202.112.14.21``、``61.139.2.69``、``114.114.114.114`` 等（Windows、Linux 如何设置 DNS 请自行查询）
        
        (3). 直接修改路由器的 DNS 地址（``Network-> DHCP and DNS``)

        .. image:: https://i.loli.net/2021/04/27/ekj69FxfZTzpAva.png
            :align: center
            :alt: DNS servers

    针对集群主机不能正常DHCP获得 IP:

        (1). 重新插拔网线、重启交换机、重启路由器的 lan 口、重启路由器 
        
        .. warning:: 
            注意按顺序操作（安操作影响的范围排序），每一步都可以使主机重新获得 IP，不需要全部执行。
        
        (2). 如果上述步骤都未能起作用，排查是否是网线、交换机、路由器等出现故障，如果确认那么只能修理或切换设备。
        关于路由器升级以及更新固件请参考：`正在写`_

4. 时间：2021年4月25日

    **故障描述**: 集群host2主机不能正常ssh。（P3故障）

    **排查**：故障情况和 :ref:`2021年3月29日 <2021_03_29_log>` 发生故障类似，按照解决方案解决后依然存在问题。连接显示器查看问题，但是连接显示器、键盘及鼠标均未有反应。

    **解决方法**：
    
    初步估计是 host2 出现死机，只能物理重启，重启后一切正常。
    排查系统日志，从2021年4月25日07:17后就没有日志记录，向前排查都是正常日志，仅有一个 账号 在 2:00～4:30一直反复连接ssh和断开ssh，是否与死机又关系待进一步验证。
    
    
5. 时间：2021年8月2日

    **故障描述**: 主楼 **停电检修** 6h后vpn服务不能使用，host3(TiTan主机)挂载点失效，导致docker服务不能使用。（P2故障）

    **排查**：本次故障是因为停电关机后发生的故障，需要重新启动vpn服务和重新挂载磁盘设备。

    **解决方法**：
    
    (1). 重新启动vpn服务： ``sudo vpnserver start``。

    .. note:: 
        2022-01-23补充：已经将上述命令写入 ``/usr/local/lib/systemd/system/vpn-302.service`` service 中并 ``systemctl enable`` 了。

    (2). 有两个硬盘分区的mount没有写入 ``/etc/fstab`` 文件，需要重新mount。

        * 最新的配置中已经把挂载点写入到 ``/etc/fstab`` 文件中，不需要手动操作了，

        * nvme拓展盘： ``sudo mount /dev/nvme0n1p3 /nvme-storage``
         
        * docker磁盘： ``sudo mount /dev/sda1 /disk2`` 另外还有一个 ``/disk`` 文件夹应该是不使用的，可以在docker的配置文件 ``/etc/docker/daemon.json`` 中反推需要mount的文件夹在哪。mount之后记得要重启docker服务 ``sudo systemctl restart docker`` 。
    
    .. note::
        2022-01-23补充：已经将 mount 持久化到了 ``/etc/fstab`` 文件中，mount 时注意用 UUID 而不是 ``/dev/sda1`` 后者会随着磁盘插入位置的不同而变化,其步骤为:
        
        获取 UUID：``sudo blkid /dev/sda3`` 。

        写入 ``etc/fstab`` 文件 ``UUID=7af536ea-446c-4f89-84b3-5573cfafdc42 /disk2 ext4 defaults`` 。

        mount前可以通过下列命令进行检查mount情况： ``df -h`` 查看已挂载设备, ``fdisk -l`` 查看所有设备, ``findmnt`` 根据设备查找mount点。
         
    (3). master路由表需要重新配置才能在连接vpn的情况下直接ssh master 或 ssh master2.
    
         * 删除绑定vpn的网口路由表： ``sudo route del -net 192.168.2.0 netmask 255.255.255.0 enp2s0``


6. 时间：2022年1月22日

    **故障描述**: 主楼 **停电检修** 前先把服务器关机导致服务器没有来电自启动，host3, host4, host5, host6不能访问。（P3故障）

    **排查**：进入 BIOS 相关设置发现，上述机器设置的是 ``Last State``, 断电前的状态是 ``Power Off``， 通电后仍然保持 ``Power Off``，手动开机即可。

    **解决方法**：
    
    手动开机。

    .. note:: 
        补充进入 BIOS 的方法，先使用显示器连接服务器，开机后注意观察屏幕提示关注如何进入 BIOS，现在是设置成 ``Last State`` 是否需要设置成 ``Power On`` 需要进一步考虑。
        下面是已经探索过的路径：

        host3等: 按 ``del`` -> ``BIOS`` -> ``Advanced`` -> ``APM`` -> ``Restore AC Power Loss`` 设置为 ``Last State`` 或 ``Power On``。

        host4， host5等: 按 ``F12`` -> ``BIOS`` -> ``Advanced`` -> ``Boot Feature`` -> ``Restore AC Power Loss`` 设置为 ``Last State`` 或 ``Power On``。


7. 时间：2022年1月22日

    **故障描述**: 主楼 **无通知停电** 导致 host3 BIOS 重设，开机显示 **Invalid Partition Table**。（P3故障）

    **排查**：想起来之前有同学处理过这个情况，应该是 BIOS 恢复默认设置后没法正确识别启动项。直接搜索这个报错发现了一个 `参考链接 <https://zhuanlan.zhihu.com/p/364016838>`_ 。

    **解决方法**：

    (1). 参照第六次故障注解部分进入 BIOS， 进入 ``Advanced`` -> ``CSM Configuration`` -> ``Boot option filter`` 设置为 ``UEFI only`` 。


    (2). 进入 ``Boot`` -> ``Boot Option #1`` 设置为 ``Ubuntu``。

    相关图片：

    .. image:: https://s2.loli.net/2022/11/09/Aer7OhVMnSPdksH.jpg
        :align: center
        :alt: invalid partition table

    .. image:: https://s2.loli.net/2022/11/09/ExPhcvFI21pBKwS.jpg
        :align: center
        :alt: advanced csm entry

    .. image:: https://s2.loli.net/2022/11/09/pUqms8WFjftKRwA.jpg
        :align: center
        :alt: uefi only
    
    .. image:: https://s2.loli.net/2022/11/09/VZgPu7USRekxfJl.jpg
        :align: center
        :alt: boot proority

