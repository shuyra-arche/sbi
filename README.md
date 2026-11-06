# sbi

Shuyra 的 `no_std` RV64 S-mode SBI 调用封装库，不包含固件或 Unit server。

`src/extension.rs`、`eid.rs`、`fid.rs` 定义扩展与调用号；`ecall.rs` 提供
`ScallBuilder<Ext>` 和汇编调用。`src/lib.rs` 导出各扩展的调用类型。
`call_raw` 将 EID/FID 放入 `a7/a6`，以 `a0..a5` 传参；返回的 `a0` 是错误码，
`a1` 是结果。参数寄存器声明为可写，调用不使用栈。

实现使用无条件 RISC-V 汇编，x86 宿主不受支持。根包没有测试入口；实际 SBI
扩展行为需要由 Kernel 在相应固件和目标机器上验证。

## 工具链与维护

使用 `rust-toolchain.toml` 固定的 `nightly-2026-09-04`，首次运行需要 Rustup
下载工具链及 RV64 目标。依赖版本以 Cargo 清单和锁文件为准。
下列命令从仓库根目录执行；宿主测试不代表真实内核运行验证。

```sh
cargo build --locked --target riscv64gc-unknown-none-elf
cargo build --locked --release --target riscv64gc-unknown-none-elf
```

## 贡献与来源

维护与 PR 规则见 [Shuyra 贡献规范](https://github.com/shuyra-arche/vanllia/blob/main/CONTRIBUTING.md)。
来源、作者与许可情况见 [PROVENANCE.md](PROVENANCE.md)。
