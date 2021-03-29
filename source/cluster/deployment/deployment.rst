.. _deployment:

==========
集群部署
==========

.. contents:: Table of Contents

.. section-numbering::

撰写目的
===============
为了方便同学们方便的使用和管理集群资源，现将实验室集群部署的流程整理成文档，希望大家不断完善，以优化集群使用体验。

网络基本情况
==============

首先说明一下学校校园网的网络结构及实验室网络设备情况。我们学校是中国教育与科研计算机网 ``CERNET`` 西南地区网络中心、四川省网网络中心，也是中国下一代互联网 ``CERNET2`` 西南地区核心节点和中国教育科研网格 China Grid 节点。而我们实验室的网络是直接接入学校校园网，未利用路由器将整个实验室形成内部局域网，这样的好处是每个同学都会分配到一个独立的教育网 IP 资源，并且带宽不会受到路由器质量的影响。但是这样的一个坏处是我们不能够方便自主的管理我们实验室各个设备的网络连接方式，并且鉴于学校的网络需要认证登录，也就是说每个设备都需要有一个账号资源，这显然是很难做到的。


技术原理与部署
==============

网路连接
--------------
实验室目前共有5台公共机器，用于服务实验室的科研任务。为了方便管理这5台机器，并且为了解决账号资源问题，我们通过一个小型路由器将他们接入，连接图示如下：

.. image:: https://i.loli.net/2021/03/29/98SdcteoMVXuk4m.png
    :width: 200
    :align: center
    
其中USTC-Network为校园网，路由器（三层设备）wan口接入校园网，具有校园网 IP。交换机（二层设备）接入路由器lan口，用于扩展路由器 lan 口接口。集群的各个主机连接到交换机接口，并从路由器自动获得 IP，从而将各个设备连接起来，使其处于同一个局域网（冲突域）中。至此的部署配置就完成了集群的网络连接过程，在后续接入设备时，请按照上面所描述的网络结构进行连接。


其中路由器选择小米路由器AC2100，并刷入 openwrt 系统，有关具体硬件和系统信息请参考 `mi_router_ac2100_wiki`_ 。

.. _`mi_router_ac2100_wiki`: https://openwrt.org/toh/xiaomi/xiaomi_mi_router_ac2100

.. image:: https://i.loli.net/2021/03/29/MfEOSmQToVlGnzg.png
    :align: center
    :alt: Mi Router AC2100

交换机为TP-LINK TL-SF1016（百兆）系列交换机。

.. image:: https://i.loli.net/2021/03/29/1GUxkzBjEAvFp5I.png
    :align: center
    :width: 400
    :alt: TL-SF1016 Switch

注意事项：

1. 集群连接的交换机（标有集群）尽量只供集群内部使用，不另外接其余主机，减少维护成本和路由器负载。
2. 校园网连接的交换机（标有校园网），可以接入其他电脑，但是不能接入具有 dhcp 功能的路由器（如需要请接入 wan 口，不能将路由器的lan口接入交换机）避免和校园网 dhcp 功能冲突，避免网络异常。具体的案例见 :ref:`集群故障及解决方法 <cluster_question>`.

网络认证
-----------
完成集群网络连接后，还需要进行校园网认证登陆(aaa.uestc.edu.cn)，有以下两种方法：

1. 手动认证，打印机电脑是连接如集群网络，并具有显示设备，所以直接用其浏览器打开认证网址，填入账号域名，认证即可。
2. 自动化认证，校园网因为是动态分配 IP，所以 IP 会随时变化，为了使得这一过程自动化，编写了一个自动登陆程序 `uestc_network_manager`_ ，详细见        README。可部署在集群内的主机上，实现断网自动认证。

.. _`uestc_network_manager`: https://github.com/ehds/uestc_network_manager


VPN搭建
-----------

完成上述步骤后，集群内部主机已经具备网络访问能力，可通过校园网接入 Internet。但是仅能在其所处的局域网内登陆（ssh等）这些设备，无法通过其他网络（有线接入校园网或这校园WIFI）访问，原因是所处的网络链路不可达（集群中除了路由器 IP 以外都是保留地址，直接使用保留地址访问主机没有路由表可达）。为了解决这个问题，通常采取的办法就是使用 VPN，VPN 可以看成一个隧道，将局域网的设备和外部设备进行联通，具体原理参考《计算机网络》或其他资料，在这不再赘述。

在本环境中，采用 `SoftEther`_ 开发的VPN组件，比较方便配置，具体原理参考 `Remote-Access-VPN-to-LAN`_。

.. image:: https://i.loli.net/2021/03/29/rnJdglbsSU587IQ.png
    :align: center
    :width: 300

如上图所示，router 中搭建 VPN 后，远端的客户端可通过 VPN 建立的隧道完成对集群节点的访问。
具体搭建步骤如下（目前仅介绍 router 和 vpn 分离方式）：

1. 下载 softethervpn 服务组件，地址 `vpnserver`_ ， 选择对应平台下载，放到集群的某一节点（以 ubuntu 为例）。

2. 启动 vpnserver，以 root 方式

    .. code-block:: shell

        sudo vpnserver start


3. 转发对应端口，因为主机位于局域网，所以需要设置路由器转发相应端口。

    (1). 进入路由器 ``network/firewall`` 管理界面

    .. image:: https://i.loli.net/2021/03/29/aVO6WCqvcsjRDL8.png
        :align: center
        
    ``zones`` 开启，``wan->lan`` 设置为 *accept*，才能开启端口转发

    (2). 转发与 VPN 相关的接口到 vpnserver 的主机上（本例为192.168.2.125）

    .. image:: https://i.loli.net/2021/03/29/JZXCcqAbt2N3O79.png
        :align: center

    其中 ``L2TP``，``IPsec`` 和 ``IKE`` 是必须开的选项用于 VPN 的客户端连接，992，5555和1194为任选其一开发（或者全开都口）用于vpnserver的管理，下文会提到。

    (3). 搭建完毕后，首次运行需要设置 vpnserver，需要下载辅助工具 vpnserver-manager，下载地址同上 vpnserver-manager，选择相应平台（以 windows 为例）
    安装后，配置连接:

    .. image:: https://i.loli.net/2021/03/29/CYVylO2UgNkHq7J.png
        :align: center

    主机名为路由器ip地址（可以为局域网地址：192.168.2.1，也可以为域名：cddsclab.f3322.net 或 vpn.dscl.team,如果你是在局域网外可以使用域名)，端口选择上面开放的管理端口（本例为5555）

    (4). 连接完成后就出现：

    .. image:: https://i.loli.net/2021/03/29/HbIeY2ZJ4dh1Bfz.png
        :align: center

    (5). 开始配置 vpnserver（首次配置，迁移配置参考步骤 :ref:`(6) <back_from_config>`）

    首次配置，需要建立 VPN 虚拟 HUB，这个就是上面提及的隧道，并且每个虚拟 HUB 是独立的。

    .. image:: https://i.loli.net/2021/03/29/hXMvjHLUNwJOl5r.png
        :align: center

    点击管理虚拟HUB可以新增用户等：这部分可以咨询

    .. image:: https://i.loli.net/2021/03/29/ErXuRw3BWJ147H2.png
        :align: center

    同时为了 vpn 能够顺利连接，并且客户端能够分配到路由器的 IP，还需要给虚拟HUB配置本地网桥，选择本地网桥设置：

    .. image:: https://i.loli.net/2021/03/29/iSRWOLF7NEGjtqD.png
        :align: center

    虚拟 HUB 选择刚刚的 VPN，LAN 适配器选择当前 vpnserver 所在主机的物理网卡名称（此处为 enp2s0）
    至此，一个完整的VPN配置就完成了，用户可以按照 `如何连接VPN`_ 进行连接操作。
    

.. _back_from_config:

    (6). 从备份文件中恢复配置
    
    当 vpnserver 所在的节点变更时，需要重新配置 vpnserver，为了方便这一流程可以直接导入之前备份好的配置文件。点击编辑配置

        1). 备份配置文件

        .. image:: https://i.loli.net/2021/03/29/wt5XTusrR1vmMUE.png
            :align: center

        点击保存到文件即可保存当前配置

        2). 导入备份文件

        选择导入文件并应用选择之前备份的文件
        注意，导入文件后，虚拟 HUB 的用户信息都还存在，但是网桥信息需要做修改，因为切换设备后，物理网卡环境发生变化，需要按照（5）步骤中的本地网桥配置进行设置。至此，恢复之前的配置就完成了。

以上，主要说明了如何利用 softethervpn 的整个步骤，主要分为端口转发，vpnserver 部署和 vpnserver 管理三个大的步骤，每一步都非常至关重要，所以特地在每个步骤中说明了该操作的原理和具体细节，希望能够有所帮助。

.. _`SoftEther`: https://www.softether.org/
.. _`Remote-Access-VPN-to-LAN`: https://www.softether.org/4-docs/2-howto/1.VPN_for_On-premise/2.Remote_Access_VPN_to_LAN
.. _`vpnserver`: https://www.softether-download.com/cn.aspx?product=softether
.. _`如何连接VPN`: https://docs.qq.com/doc/DTnhkdFVnTUpBTlFz

动态DNS
-----------
完成上述步骤后，vpn 已经具备基本工作能力，但是由于上面提到校园网 IP 经常变动，所以连接集群路由的IP也会经常改变，为了方便，需要利用域名来标识路由，并利用动态绑定脚本对 IP 和域名进行绑定。

具体原理就是利用DDNS脚本，随时检查 IP 和域名对应关系，当 IP 发生改变，则向域名注册商发起修改请求，保证域名和 IP 的对应关系。

配置步骤如下：

    （1). 申请域名，本例采用免费的域名服务商 `pubyun`_ .

        注册账号->申请动态域名->配置

        .. image:: https://i.loli.net/2021/03/29/IxXDQhJpZbfKWRq.png
            :align: center

        主要是配置更新密码，后面会使用到。

    (2). 进入路由器管理界面 ``services/ddns`` 界面，如果没有该选项进入 ``system/software`` 安装luci-app-ddns即可，有关 openwrt 如何安装插件请自行查阅，此处不再赘述。

    (3). 新建一个解析服务

    .. image:: https://i.loli.net/2021/03/29/SR1YxZ5LibvdjPy.png
            :align: center

    (4). 配置相关信息

        .. image:: https://i.loli.net/2021/03/29/BWso6zrpeGl738Y.png
            :align: center

    按照上面表格填入相关信息，例如在此我们选择来 ``3322.org`` 作为域名解析商，域名为 ``cddsclab.f3322.net``，用户名为root，密码为刚刚设置的密码即可。
    还需要配置，监听的 IP 接口，因为我们要动态绑定校园网 IP，所以需要选择 ``IP address sourece`` 为 ``Network``，``Network`` 选择为 ``wan`` 口即可

        .. image:: https://i.loli.net/2021/03/29/y4NXZsn85Bi7udw.png  
            :align: center

    注意域名解析可能存在一定的延迟，可以检查域名管理界面查看 IP 是否正常更新。其余配置可保持默认即可，当然可以根据具体情况进行配置。

.. _`pubyun`: http://www.pubyun.com


定时任务
-----------

为了保持路由器的状态处于较优状态，设定了定时重启任务，``system/scheduled task``

    .. code-block:: bash

        #  reboot the route at 4:30 am for every day
        30 4 * * * sleep 70 && touch /etc/banner && reboot

``sleep 70`` 的作用防止重启的时间过快从而导致反复重启，所以需要先睡眠60秒以上。

具体编写格式参考 `openwrt_cron`_

.. _`openwrt_cron`: https://openwrt.org/docs/guide-user/base-system/cron



