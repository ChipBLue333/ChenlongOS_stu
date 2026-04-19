# ArceOS Helloworld 运行指南 (WSL + QEMU RISC-V)

为了在本地 WSL 环境中成功运行 ArceOS 的 `helloworld` 示例并完成考核，请按照以下步骤逐步操作：

## 1. 安装系统依赖和 QEMU
首先，需要确保你的 WSL Ubuntu/Debian 环境中安装了必要的编译工具和 QEMU 模拟器（用于运行 RISC-V 架构）。在终端中执行以下命令：

```bash
sudo apt-get update
sudo apt-get install -y build-essential qemu-system-riscv64
```

## 2. 安装 Rust 工具链
ArceOS 是用 Rust 编写的，因此需要安装 Rust 环境（如果你还没有安装的话）：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
```

由于 ArceOS 可能需要 nightly 版本和特定的目标平台支持，由于通常构建会自动处理，建议确认基础 Rust 可以正常工作。

## 3. 安装 ArceOS 编译所需的 Cargo 工具
根据 README 文档，你需要安装以下几个关键的模块和工具包用于构建镜像和生成配置：
```bash
cargo install cargo-binutils axconfig-gen cargo-axplat
```
*注意：这将安装 `rust-objcopy` / `rust-objdump` 以及内核和平台配置工具。*

## 4. 获取 ArceOS 源码
如果当前目录下还没有 ArceOS 的完整内核源码，请将其克隆下来：

```bash
git clone https://github.com/arceos-org/arceos.git
cd arceos
```

## 5. 编译并运行 Helloworld (RISC-V)
进入 `arceos` 源代码根文件夹后，使用 Makefile 执行 `apps/helloworld` 应用，并指定架构为 `riscv64`。

执行启动命令：
```bash
make A=apps/helloworld ARCH=riscv64 run
```
*注：有些旧版本路径为 `A=examples/helloworld`，如果遇到找不到路径，可以换成 `A=examples/helloworld`。*

你将看到 QEMU 启动，并在终端中输出类似如下的内容：
```text
[rustsbi] ...
...
Hello, world!
```
我的运行结果如下：
```text
(base) root@PC-20260326TTGP:/home/arceos/arceos# make A=examples/helloworld ARCH=riscv64 run
axconfig-gen configs/defconfig.toml /root/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/axplat-riscv64-qemu-virt-0.4.1/axconfig.toml  -w arch=riscv64 -w platform=riscv64-qemu-virt -o "/home/arceos/arceos/.axconfig.toml" -c "/home/arceos/arceos/.axconfig.toml"
    Building App: helloworld, Arch: riscv64, Platform: riscv64-qemu-virt, App type: rust
cargo -C examples/helloworld build -Z unstable-options --target riscv64gc-unknown-none-elf --target-dir /home/arceos/arceos/target --release  --features "axstd/defplat axstd/log-level-warn"
    Finished `release` profile [optimized] target(s) in 0.08s
rust-objcopy --binary-architecture=riscv64 examples/helloworld/helloworld_riscv64-qemu-virt.elf --strip-all -O binary examples/helloworld/helloworld_riscv64-qemu-virt.bin
    Running on qemu...
qemu-system-riscv64 -m 128M -smp 1 -machine virt -bios default -kernel examples/helloworld/helloworld_riscv64-qemu-virt.bin -nographic

OpenSBI v1.3
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|___/_____|
        | |
        |_|

Platform Name             : riscv-virtio,qemu
Platform Features         : medeleg
Platform HART Count       : 1
Platform IPI Device       : aclint-mswi
Platform Timer Device     : aclint-mtimer @ 10000000Hz
Platform Console Device   : uart8250
Platform HSM Device       : ---
Platform PMU Device       : ---
Platform Reboot Device    : sifive_test
Platform Shutdown Device  : sifive_test
Platform Suspend Device   : ---
Platform CPPC Device      : ---
Firmware Base             : 0x80000000
Firmware Size             : 322 KB
Firmware RW Offset        : 0x40000
Firmware RW Size          : 66 KB
Firmware Heap Offset      : 0x48000
Firmware Heap Size        : 34 KB (total), 2 KB (reserved), 9 KB (used), 22 KB (free)
Firmware Scratch Size     : 4096 B (total), 760 B (used), 3336 B (free)
Runtime SBI Version       : 1.0

Domain0 Name              : root
Domain0 Boot HART         : 0
Domain0 HARTs             : 0*
Domain0 Region00          : 0x0000000002000000-0x000000000200ffff M: (I,R,W) S/U: ()
Domain0 Region01          : 0x0000000080040000-0x000000008005ffff M: (R,W) S/U: ()
Domain0 Region02          : 0x0000000080000000-0x000000008003ffff M: (R,X) S/U: ()
Domain0 Region03          : 0x0000000000000000-0xffffffffffffffff M: (R,W,X) S/U: (R,W,X)
Domain0 Next Address      : 0x0000000080200000
Domain0 Next Arg1         : 0x0000000087e00000
Domain0 Next Mode         : S-mode
Domain0 SysReset          : yes
Domain0 SysSuspend        : yes

Boot HART ID              : 0
Boot HART Domain          : root
Boot HART Priv Version    : v1.12
Boot HART Base ISA        : rv64imafdch
Boot HART ISA Extensions  : time,sstc
Boot HART PMP Count       : 16
Boot HART PMP Granularity : 4
Boot HART PMP Address Bits: 54
Boot HART MHPM Count      : 16
Boot HART MIDELEG         : 0x0000000000001666
Boot HART MEDELEG         : 0x0000000000f0b509

       d8888                            .d88888b.   .d8888b.
      d88888                           d88P" "Y88b d88P  Y88b
     d88P888                           888     888 Y88b.
    d88P 888 888d888  .d8888b  .d88b.  888     888  "Y888b.
   d88P  888 888P"   d88P"    d8P  Y8b 888     888     "Y88b.
  d88P   888 888     888      88888888 888     888       "888
 d8888888888 888     Y88b.    Y8b.     Y88b. .d88P Y88b  d88P
d88P     888 888      "Y8888P  "Y8888   "Y88888P"   "Y8888P"

arch = riscv64
platform = riscv64-qemu-virt
target = riscv64gc-unknown-none-elf
build_mode = release
log_level = warn

smp = 1
Hello, world!
```


## 6. 考核提交截图要求
1. **启动命令截图**：截取你在终端中输入 `make A=apps/helloworld ARCH=riscv64 run` 命令那一行的截图。
2. **Helloworld 输出截图**：截取 QEMU 成功启动 ArceOS，并打印出 `Hello, world!` 及其周围内核日志信息的控制台界面。
3. （可退出 QEMU：按下 `Ctrl + A`，然后松开按 `X` 即可安全退出 QEMU）。

