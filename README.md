# rea-quick-install — REA + Ghidra 一键快速安装

在 **arm64 (aarch64) Linux** 设备上装 REA + Ghidra 的反编译器时，官方 Ghidra 发布包
没有该平台二进制，现场编译要 **15–20 分钟**。本仓库预编译好了反编译器，整个安装只需
**3–5 分钟**（网络快时），重装只需 30 秒。

## 一键安装

```bash
curl -fsSL https://raw.githubusercontent.com/Jack-kd/rea-quick-install/main/install-rea-ghidra.sh | sudo bash
```

也可以克隆到本地再跑：

```bash
git clone https://github.com/Jack-kd/rea-quick-install.git
sudo bash rea-quick-install/install-rea-ghidra.sh
```

装完 `rea doctor` 应显示 ghidra 相关检查全部 healthy。

## 支持的系统

- 架构：**x86_64**（官方包自带反编译器，无需预编译包）/ **aarch64**（使用本仓库预编译包）
- 系统：glibc 发行版（Ubuntu / Debian / Kali / Fedora 等）
- 依赖：Node.js 22.19+/24.11+/26+ 与 npm；JDK 21 自动安装（apt）

## 特性

| 特性 | 说明 |
|---|---|
| **断点续传** | 下载 Ghidra（543MB）中断后重跑脚本会从断点继续，不重新下载 |
| **zip 缓存复用** | 已下载的 zip 缓存在 `/var/cache/rea-install/`，重装/重跑直接复用（秒过） |
| **多源下载** | 本仓库 Release → GitHub 官方 → ghfast/ghproxy/gh-proxy 加速镜像，逐个尝试 |
| **免编译** | arm64 反编译器（decompile + sleigh）预编译包自动拉取并校验 SHA-256 |
| **幂等** | 已装好的部分自动跳过；重跑快路径约 30 秒 |
| **环境变量固化** | 自动给 `rea` 挂包装脚本（`GHIDRA_INSTALL_DIR`/`JAVA_HOME`），适配不加载 .bashrc 的容器 shell |

## 常用参数

| 参数 | 说明 |
|---|---|
| `--ghidra-zip <路径>` | 使用本地已下载的 Ghidra zip，跳过下载 |
| `--decompiler-tgz <路径>` | 使用本地预编译反编译器包 |
| `--compile` | 强制从源码编译反编译器（并行 make，备用） |
| `--skip-rea` | 只装 Ghidra |
| `--install-dir <路径>` | 自定义安装目录（默认 `/opt/ghidra_12.1.4_PUBLIC`） |
| `--no-cache` | 装完不保留 zip 缓存 |

## 耗时估算

| 环节 | 时间 |
|---|---|
| 拉取脚本+预编译包（~2MB） | 几秒 |
| npm 装 rea-agents | ~30 秒 |
| 装 JDK 21 | ~1–2 分钟（已有则跳过） |
| 下载 Ghidra 543MB | 30 秒 ~ 10 分钟（取决于网络；可断点续传） |
| 解压 + 复制反编译器 | ~1 分钟 |
| 验证 `rea doctor` | ~30 秒 |

## 内容

- `install-rea-ghidra.sh` — 一键安装脚本（幂等）
- `rea-ghidra-arm64-decompiler-12.1.4.tar.gz` — 预编译的 arm64 反编译器（SHA-256 见 checksums）
- Release `v1.0` 附带了 Ghidra 官方发布包 `ghidra_12.1.4_PUBLIC_20260921.zip`（543MB，离线保底源）

## 构建信息（预编译包）

- 构建环境：Ubuntu 24.04 arm64（glibc 2.39，g++ 13.3），与 Ghidra 12.1.4 官方源码一致
- 兼容性：动态链接标准 aarch64 glibc 库，适用于常见 glibc 发行版；musl（Alpine）不适用
- 重新编译：`install-rea-ghidra.sh --compile`（并行 make，比首次快 3–4 倍）

## 更新 REA 后

`npm install -g rea-agents` 会重建 `rea` 符号链接覆盖包装脚本；重跑一遍本脚本即可恢复（秒级）。
