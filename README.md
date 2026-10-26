# sbi

`sbi` 是 shuyra 的 `no_std` RV64 S-mode SBI ABI 封装基础库：用
`ScallBuilder<Ext>` 绑定 EID 与 FID，由唯一碰汇编的 `call_raw` 发起
`ecall`，并把 `a0` 错误码转换为 `SError`、把 `a1` 作为返回值。

它是基础库，不是 Unit；这里没有 UnitFile、ELF、server，也没有自己的
固件。仓库根的 `SBI.bin` 是单独的 SBI 固件，不属于本 crate，本目录没有
复制、移动或替换它。来源 `SBI.bin` 的 SHA-256 保持
`c53d58aa9ab17f95fa1af159f0bf05fad5c5622ebc0f1084463435fa6e5752b6`。

## 独立构建

本目录是独立 Cargo 工作区，根包 `sbi` 为 `0.1.0`、edition 2024，
`[workspace]` 使用 `resolver = "3"`。工具链由 `rust-toolchain.toml`
固定为 `nightly-2026-09-04`。

```sh
cd sbi
cargo build --locked --target riscv64gc-unknown-none-elf
cargo build --locked --release --target riscv64gc-unknown-none-elf
cargo build --locked --offline --target riscv64gc-unknown-none-elf
cargo build --locked --offline --release --target riscv64gc-unknown-none-elf
```

目标只有 RV64 S-mode。`call_raw` 是无条件 RISC-V `ecall` 汇编：
`a7 = EID`、`a6 = FID`、`a0..a5 = 参数`，全部参数寄存器声明为
`inlateout`（会被写），`options(nostack)`；返回后 `a0` 是错误码、
`a1` 是值。`SError` 保持 0 / -1..-6 / `Unknown` 的映射，失败经
`erra` 的 `annotate("s-mode environment call")` 返回。

x86 宿主不受支持：本目录没有 `cfg` 分支、宿主 stub 或假的宿主成功路径。
`[lib]` 保持 `test = false`、`bench = false`、`doctest = false`，本 crate
没有 owned 测试。`cargo test` 即使返回 0 也只执行 0 个 owned 用例，不能
当作 ABI 验证。

## 依赖

根包只保留来源中的两项依赖：

- `erra 0.2.0`：registry，`default-features = false`；
- `fack 0.2.0`：registry，`default-features = false`。

`fack` 的传递依赖 `fack-core`、`fack-macro`、`fack-codegen`、
`proc-macro2`、`quote`、`syn`、`unicode-ident` 的版本和 checksum 均取自
来源根 `Cargo.lock`，本次提取没有升级。没有 `[patch]`、`[replace]`、
本地 Git 依赖或 cache 投影。

锁的 seed、剪枝和版本对照见 `PROVENANCE.md`。

## 验证边界

本目录当前只验证 RV64 目标上的 debug/release 构建（在线准备缓存后重复
`--locked --offline`）。未运行 QEMU，未运行宿主机测试，未验证任何
SBI 扩展的运行时行为，也不宣称 PMU、CPPC、NaCl 等未被本树使用的扩展
已被覆盖。`sbi` wrapper 的实际运行时覆盖以后由 kernel consumer 的真实
QEMU 回归承担。

## 许可证

来源项目没有声明许可证，也没有跟踪的 `LICENSE`、`COPYING` 或 `NOTICE`。
本目录不添加许可证文本或 SPDX 元数据，不假设来源未授予的许可证。
