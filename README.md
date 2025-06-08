# Docker Images Pusher

使用Github Action将国外的Docker镜像转存到阿里云私有仓库，供国内服务器使用，免费易用<br>
- 支持DockerHub, gcr.io, k8s.io, ghcr.io等任意仓库<br>
- 支持最大40GB的大型镜像<br>
- 使用阿里云的官方线路，速度快<br>
- 支持多架构镜像构建<br>

## 使用方式

### 配置阿里云
登录阿里云容器镜像服务<br>
https://cr.console.aliyun.com/<br>
启用个人实例，创建一个命名空间（**ALIYUN_NAME_SPACE**）

访问凭证–>获取环境变量<br>
用户名（**ALIYUN_REGISTRY_USER**)<br>
密码（**ALIYUN_REGISTRY_PASSWORD**)<br>
仓库地址（**ALIYUN_REGISTRY**）<br>

### Fork本项目
Fork本项目<br>
#### 启动Action
进入您自己的项目，点击Action，启用Github Action功能<br>
#### 配置环境变量
进入Settings->Secret and variables->Actions->New Repository secret<br>
将上一步的**四个值**<br>
ALIYUN_NAME_SPACE,ALIYUN_REGISTRY_USER，ALIYUN_REGISTRY_PASSWORD，ALIYUN_REGISTRY<br>
配置成环境变量

### 添加镜像
打开images.txt文件，使用多架构组语法添加镜像：

#### 多架构镜像配置语法
```
# 多架构镜像组（最终镜像名：caddy:2-alpine）
MULTI_ARCH_GROUP:caddy:2-alpine
--platform=linux/amd64 caddy:2-alpine
--platform=linux/arm64 caddy:2-alpine

# PHP 多架构镜像组（最终镜像名：serversideup-php:8.4-fpm）
MULTI_ARCH_GROUP:serversideup/php:8.4-fpm
--platform=linux/amd64 serversideup/php:8.4-fpm
--platform=linux/arm64 serversideup/php:8.4-fpm
```

#### 命名规则
- 源镜像地址中的 `/` 会被替换为 `-`
- 保留 `:` 作为标签分隔符
- 例如：`serversideup/php:8.4-fpm` → `serversideup-php:8.4-fpm`
- 例如：`caddy:2-alpine` → `caddy:2-alpine`

#### 优势
- 同一个镜像名支持多个架构
- Docker 会自动选择匹配当前系统的架构
- 符合 Docker 官方多架构最佳实践
- 镜像仓库更整洁，无需架构前缀
- 保持与源镜像的对应关系

### 使用镜像
回到阿里云，镜像仓库，点击任意镜像，可查看镜像状态。(可以改成公开，拉取镜像免登录)

在国内服务器pull镜像, 例如：<br>
```
docker pull registry.cn-hangzhou.aliyuncs.com/shrimp-images/caddy:2-alpine
docker pull registry.cn-hangzhou.aliyuncs.com/shrimp-images/serversideup-php:8.4-fpm
docker pull registry.cn-hangzhou.aliyuncs.com/shrimp-images/serversideup-php:8.4-fpm-nginx
```

### 测试运行
```
docker run -d \
  --name caddy-test \
  -p 8080:80 \
  -p 8443:443 \
  registry.cn-hangzhou.aliyuncs.com/llcache/caddy:2-alpine
```

### 定时执行
修改/.github/workflows/docker.yaml文件<br>
添加 schedule即可定时执行(此处cron使用UTC时区)