# Deploy Artifacts

受限网络环境下部署所需的**二进制产物镜像**（Docker 引擎离线包、Compose 二进制、常用镜像 tar 等）。

Git 仓库不适合存放几十上百 MB 的二进制文件（单文件 100 MiB 硬限制），所以这些产物统一以
**Release 资产**的形式发布，按「组件 + 平台」打 tag，便于脚本按需下载。

## 为什么需要这个仓库

很多生产机器（尤其境外或内网隔离环境）无法访问 Docker Hub、GitHub 直连或系统软件源，
但安装过程又需要这些大文件。把产物放在公开 Release 里，安装脚本就能：

- 有外网时按需拉取，无需人工拷贝；
- 校验 sha256 后再使用，避免拿到被替换或损坏的文件；
- 下载失败时回退到其它来源（在线安装或本地离线包）。

## 内容一览

| Tag | 内容 | 适用平台 |
|---|---|---|
| `docker-el7-v2` | Docker 引擎离线包（自足 RPM 集，含系统依赖） | CentOS 7 / RHEL 7，x86_64 |
| `docker-el9-v2` | Docker 引擎离线包（差分 RPM 闭包：只带 Docker 组件及其缺的包） | RHEL 9 系（Rocky 9 / AlmaLinux 9 / RHEL 9 / CentOS Stream 9），x86_64 |
| `docker-ubuntu-2204-v1` | Docker 引擎离线包（自足 deb 集，含系统依赖） | Ubuntu 22.04 LTS，x86_64 |
| `docker-ubuntu-2404-v1` | Docker 引擎离线包（自足 deb 集，含系统依赖） | Ubuntu 24.04 LTS，x86_64 |
| `compose-v5.1.3` | Docker Compose 独立静态二进制 | linux x86_64 |
| `images-el7-v1` | 常用容器镜像 tar（mysql / nginx / openresty / temurin-8-jre / coturn / redis，每个镜像单独一个文件） | linux amd64（与发行版无关） |

每个 Release 的说明里写明用途、sha256 与使用方式。

说明：`docker-el9-v1`（旧）是 CentOS Stream 9 上打的**全闭包**，含 systemd/glibc 等基础包，装到 Rocky/Alma/RHEL 9.4 会被 dnf 以「替换受保护包」拒绝，**请改用 `docker-el9-v2`**。

说明：`images-*` 里的 tar 是 `docker save` 产物，与宿主发行版无关，任何 amd64 Linux 上的 Docker 都可 `docker load -i <文件名>`；文件名是固定契约（安装脚本按名字找文件），不要改名。

## 下载

直连模板：

```text
https://github.com/djsaleqwn/deploy-artifacts/releases/download/<tag>/<asset>
```

境内网络直连 GitHub 下载大文件常常很慢甚至 0 字节，可以给地址加公开代理前缀（二选一）：

```text
https://ghfast.top/https://github.com/djsaleqwn/deploy-artifacts/releases/download/<tag>/<asset>
https://gh-proxy.com/https://github.com/djsaleqwn/deploy-artifacts/releases/download/<tag>/<asset>
```

示例（把结果校验后再解包）：

```bash
url=https://github.com/djsaleqwn/deploy-artifacts/releases/download/compose-v5.1.3/docker-compose-linux-x86_64-5.1.3
curl -fL --connect-timeout 5 --speed-limit 102400 --speed-time 30 -o /tmp/docker-compose "$url"
sha256sum /tmp/docker-compose
chmod +x /tmp/docker-compose
```

## 命名规则

- **Tag**：`<组件>-<平台>-v<序号>`，例如 `docker-el7-v2`、`compose-v5.1.3`、`images-el7-v1`。
- **资产文件名**：保持使用方期望的原名（例如 `docker-offline.tar.gz`），不要改，否则脚本认不出来。
- 更新产物时**发新 tag**，不要覆盖旧 tag 的资产——已经部署的机器可能依赖旧版本。

## 校验

每个资产的正确 sha256 同时写在对应 Release 说明里。下载后请先校验再使用：

```bash
sha256sum <file>   # macOS: shasum -a 256 <file>
```

## 许可与免责

本仓库只做产物分发。每个产物仍归其上游项目所有，并沿用上游许可（例如 Docker Engine 为
Apache-2.0）。不提供任何担保，请自行确认与目标环境的兼容性。

本仓库**不包含**任何源码、业务程序、密钥、凭据或客户信息。
