.. _gitlab_deployment:

===============
gitlab 服务部署
===============

.. contents:: 目录

撰写目的
===============

方便大家协作开发，利用 gitlab-ce 开源组件搭建了实验室内部的 `git 服务`_。
本文文档主要用于介绍服务的部署和备份。

当前该服务部署在 master 服务器上，路径为：`/home/dscl/gitlab`

.. _`git 服务`: http://git.dscl.team/

备份和部署
===========

1. 在 `.env` 中文件中,设置 `app_root` 路径.

2. 启动gitlab

.. code-block:: shell

    docker-compose up -d

需要等待一段时间，才能启动成功,可以使用 `docker log <container name>` 查看.

3. 导入备份文件

    目前每次备份的文件在 `dscl-gitlab-backup`_

    1. 导入备份文件到 ``data/backups`` 中，注意文件权限

    2. 将 ``config/gitlab.rb`` 和 ``config/gitlab-secrets.json`` 放到项目目录下的 ``config`` 中

    3. 进入 gitlab container 命令行, 设置备份文件的权限
    ``chown git:git <backup_file>``

    4. 关闭相关服务

    .. code-block:: shell

        gitlab-ctl stop unicorn
        gitlab-ctl stop puma
        gitlab-ctl stop sidekiq


4. 导入备份

.. code-block:: shell

    gitlab-backup restore BACKUP=<backup_file>

**注意，文件名字不要包含后缀 _gitlab_backup.tar**

上述详细的步骤请参考 `gitlab-restore`_


.. _`dscl-gitlab-backup`: http://cddsclab.f3322.net:9000/minio/backup/gitlab/
.. _`gitlab-restore`: https://docs.gitlab.com/ee/raketasks/backup_restore.html#restore-for-docker-image-and-gitlab-helm-chart-installations