# Agent Instructions — 本仓库是什么 / 怎么装

**仓库用途**：一键安装 REA（Reverse Engineer Anything）+ Ghidra 逆向分析工具链，
并解决 arm64 Linux 上 Ghidra 反编译器缺失需现场编译的问题（本仓库已预编译好）。

## 一句话安装

```bash
sudo bash install-rea-ghidra.sh
```

（等效的一行远程执行：`curl -fsSL https://raw.githubusercontent.com/Jack-kd/rea-quick-install/main/install-rea-ghidra.sh | sudo bash`）

## 这个脚本会做什么

1. 检测架构（x86_64 / aarch64），映射 Ghidra 平台目录
2. 检查/安装 JDK 21（apt，已有则跳过）
3. 检查/安装 rea-agents（npm 全局，已有则跳过）
4. 下载 Ghidra 12.1.4（543MB，断点续传，多镜像源，zip 缓存于 /var/cache/rea-install）
5. 安装反编译器：aarch64 用仓库内预编译包 `rea-ghidra-arm64-decompiler-12.1.4.tar.gz`
   （缺失时自动从仓库 raw 拉取）；x86_64 官方包自带，跳过
6. 给 `rea` 挂环境变量包装脚本（GHIDRA_INSTALL_DIR / JAVA_HOME）
7. 运行 `rea doctor` 验证

## 验证标准

`rea doctor` 输出中 ghidra / ghidra-native_decompiler / ghidra-version 等项为 healthy。

## 常见参数（完整见 README.md）

- `--ghidra-zip <路径>`：用本地已下载的 Ghidra zip，跳过下载
- `--decompiler-tgz <路径>`：指定本地预编译反编译器包
- `--compile`：强制源码编译反编译器（备用）
- `--skip-rea`：只装 Ghidra
- `--install-dir <路径>`：自定义安装目录
- `--no-cache`：装完不保留 zip 缓存

## 约束与注意事项

- 仅支持 **Linux**（x86_64 / aarch64）；Windows/macOS 不适用（需 WSL 等）
- 需要 root（脚本写 /opt、全局 npm、apt）
- 需要 Node.js 22.19+/24.11+/26+ 与 npm（脚本会检查，缺了会报错提示）
- 脚本幂等：已装好的部分自动跳过，重跑很快
- 若下载失败：残留文件保留在 /var/cache/rea-install/，可断点重跑

## APK / Android 分析能力（装完即有）

脚本会自动下载 JADX 桥接（`jadx-headless-mcp`，REA_JADX_MCP_JAR）并配置进 `rea` 包装脚本。
装完即可对 APK 做静态逆向（不运行目标代码）：

- `rea inspect-android-package <apk>` — 包信息、权限、Manifest、类/资源统计
- `rea inspect-android-class <apk> <完整类名>` — 类的成员清单（方法/字段/内部类）
- `rea inspect-android-method <apk> <类名> <方法名>` — 反编译单个方法为 Java 代码

APK 分析入口：先 `inspect-android-package` 拿清单，再按需深入类/方法。
