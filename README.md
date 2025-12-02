# Teanote Helm Chart

这是一个用于部署Note笔记应用的Helm chart。

## 包含组件

- **note-api**: Web API服务，监听在8080端口
- **note-ui**: Web前端应用，监听在4000端口
- **数据库备份**: 定时备份数据库并上传到对象存储（可选，备份文件格式：backup-YYYY-MM-DD-HH_MM_SS.sql.age）

## 安装

```bash
# 添加仓库（示例）
# helm note add your-repo https://your-helm-repo.example.com

# 安装chart
helm install note ./note-chart

# 或者指定自定义values文件
helm install note ./note-chart -f custom-values.yaml
```

## 配置说明

### 主要配置选项

```yaml
# image:
#   api:
#     repository: note/api
#     tag: "dev"
#   ui:
#     repository: note/ui
#     tag: "dev"

# imagePullSecrets:
#   - name: your-registry

ingress:
  enabled: true
  annotations:
    {}
    # 使用acme 启用下面的注解
    # cert-manager.io/cluster-issuer: $[your_issuer]
  hostname: note.example.com
  # 默认不启用tls
  # tls:
  #   enabled: false

# note管理员用户名，密码
note:
  user: your_username
  password: "your_password"

# api 的日志级别，默认info
# log:
#   level: info

# 数据库，type必须是mysql
database:
  type: mysql
  host: db
  user: your_user
  password: "your_password"
  database: your_database

# redisdatabase默认为0
redis:
  host: redis
  password: "your_password"
  database: 1

## 图片上传配置
# imageUpload:
#   # 上传图片后文件名的前缀
#   # 默认值：images
#   prefix:
#     images
#   #图片上传的临时目录，默认值 /app/tmp
#   tmp_dir:
#     /app/tmp


### 存储图片的s3配置，已测minio、aliyun、aws，只需要配置其中之一

## minio
# s3:
  # endpoint: http://127.0.0.1:9000
  # bucket: bucket_name
  # access_key: your_access_key
  # secret_key: "your_secret_key"
  # 默认值：false
  # path_style:
    # true


## aliyun oss
# s3:
#   endpoint: https://oss-cn-hongkong.aliyuncs.com
#   bucket: bucket_name
#   access_key: "your_access_key"
#   secret_key: "your_secret_key"

## aws
# s3:
#   endpoint: https://s3.amazonaws.com
#   bucket: bucket_name
#   access_key: your_access_key
#   secret_key: your_secret_key
  # aws需要设置s3_region为aws的region，默认值：""
  # s3_region: ""

# 数据库备份配置（可选）
backup:
  enabled: false  # 是否启用备份
  schedule: "0 2 * * *"  # cron表达式，默认每天凌晨2点（Asia/Shanghai时区）
  retention: 5  # 保留最近的备份数量
  image:
    repository: "ubuntu"  # 备份使用的镜像
    tag: "22.04"          # 镜像标签
    pullPolicy: IfNotPresent
  age:
    publicKey: ""  # age加密公钥
  s3:
    prefix: "backup/"  # 备份文件路径前缀
```

### 自定义备份镜像

为了避免每次备份时都安装必要的工具，您可以创建一个预装了所有工具的自定义镜像。以下是一个示例Dockerfile：

```dockerfile
FROM ubuntu:22.04

# 添加时区文件
RUN apt-get update && apt-get install -y tzdata

# 安装必要的工具
RUN apt-get install -y \
    mariadb-client age wget \
    && rm -rf /var/lib/apt/lists/*

RUN  wget https://dl.min.io/client/mc/release/linux-amd64/mc -O /usr/bin/mc && \
      chmod +x /usr/bin/mc


# 设置工作目录
WORKDIR /backup

CMD ["bash"]
```

构建并推送镜像后，在values.yaml中指定您的自定义镜像：

```yaml
backup:
  image:
    repository: "your-registry/backup-tools"
    tag: "latest"
```

## 卸载

```bash
helm uninstall note
``` 

# 许可证
项目使用AGPL-3.0 许可证 ([LICENSE](LICENSE))。
