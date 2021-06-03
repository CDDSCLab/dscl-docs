# gitlab 服务部署

## 撰写目的

方便大家协作开发，利用 gitlab-ce 开源组件搭建了实验室内部的 [git 服务](http://git.dscl.team)。 本文文档主要用于介绍服务的部署和备份。

当前该服务部署在 master 服务器上，路径为：/home/dscl/gitlab

## 从备份中恢复
部署文件放在 [gitlab-docs](http://git.dscl.team/publicservervice/dscl-gitlab) 仓库中。
1. 在 `.env` 中文件中,设置 `app_root` 路径.

2. 启动gitlab
`docker-compose up -d`
需要等待一段时间，才能启动成功,可以使用 `docker log <container name>` 查看.

3. 导入备份文件

    目前每次备份的文件在 [dscl-gitlab-backup](http://cddsclab.f3322.net:9000/minio/backup/gitlab/)，按照时间命名。
    1. 导入备份文件到 `data/backups`中，注意文件权限
    2. 将`config/gitlab.rb` 和 `config/gitlab-secrets.json` 放到项目目录下的 `config`中
    3. 进入 gitlab container 命令行, 设置备份文件的权限
`chown git:git <backup_file>`
    4. 关闭相关服务
        ```shell
        gitlab-ctl stop unicorn
        gitlab-ctl stop puma
        gitlab-ctl stop sidekiq
        ```
    5. 导入备份
    ```bash
    gitlab-backup restore BACKUP=<backup_file>
    ```
    **注意，文件名字不要包含后缀`_gitlab_backup.tar`**

    上述详细的步骤请参考[gitlab-restore](https://docs.gitlab.com/ee/raketasks/backup_restore.html#restore-for-docker-image-and-gitlab-helm-chart-installations)

## 备份整个gitlab
### 如何备份
进入 gitlab 所在docker，执行：
``` shell
gitlab-backup create
```
备份文件会存储在 `data/backups` 下面。

如果配置了 s3 存储，那么文件会自动保存到 s3 上面, 具体配置信息参考 [gitlab 备份到 s3](http://git.dscl.team/publicservervice/dscl-gitlab/-/blob/master/gitlab.rb#L553)

更多信息参考 [gitlab 备份与恢复](https://docs.gitlab.com/ee/raketasks/backup_restore.html)


### 定时备份（optional）
可以参考 cron 的使用，定时触发备份命令，具体如何设置不再详细阐述。
