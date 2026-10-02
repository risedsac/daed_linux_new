# eBPF 依赖升级说明

基于官方 daed 2.1.1 完整源码包修改。
此仓库提供可浏览的源码，以及单独构建的 `daed 2.1.1-2` Arch Linux 软件包。

修改 wing/go.mod、wing/go.sum、wing/dae-core/go.mod、wing/dae-core/go.sum：

- github.com/cilium/ebpf：v0.20.0 → v0.22.0。
- golang.org/x/sys：v0.41.0 → v0.43.0。
- golang.org/x/sync：v0.19.0 → v0.20.0。
- 同步所需的间接依赖与校验值。

没有修改业务逻辑或 eBPF C 源码。主程序及 trace 的 Go 绑定和 eBPF 对象已重新生成。

## 验证

- `make ebpf`：成功生成小端、大端的主程序和 x86 trace 程序。
- dae 核心和 dae-wing 后端：`go build -mod=readonly -buildvcs=false -tags=trace` 成功。
- 两个验证二进制的 `go version -m` 均显示 github.com/cilium/ebpf v0.22.0。
- 现有 TestValidateRequiredBpfMapsLoaded、TestTunePlaceholderBpfMaps、
  TestConnStateStructSizes、TestCustomizeBpfMapSpecs 均通过。
- 未运行需要内核 BPF 权限的加载测试或实际代理联网测试。
- 前端及完整 daed 已构建成功，生成 Arch Linux x86_64 软件包 `daed 2.1.1-2`。

## 构建完整 daed

仓库不包含下载缓存、node_modules 或 Git 子模块元数据；所需源码均已包含。
需要网络及 Go 1.26 或更高版本、pnpm、clang、make，建议安装 llvm-strip。
在解压后的源码根目录执行：

```bash
upgrade_root="$(pwd)"
GOFLAGS="-buildvcs=false" make VERSION=2.1.1 GOMODCACHE="$upgrade_root/.go-mod"
```

该命令生成源码根目录的 daed 二进制。将模块缓存放在源码根目录，避免 Go 在 wing 模块内扫描依赖缓存。

## 单独应用补丁

在原 daed 2.1.1 完整源码的根目录执行：

```bash
patch -p1 < /path/to/daed_linux_new/patches/daed-ebpf-0.22.0.patch
```

然后重新构建，确保重新生成 eBPF 对象及 Go 绑定。
v0.22.0 发布说明：https://github.com/cilium/ebpf/releases/tag/v0.22.0
