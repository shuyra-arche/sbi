# Provenance

## Source snapshot

- Repository: <https://github.com/anreonyr/sqware>
- Commit: `c970afaf7dae1145633278253351be729b625717`
- Published source branch at that commit: `codex/shuyra-resource-extraction`
- Published pull request: <https://github.com/anreonyr/sqware/pull/32>
  ("Consume published resource and remove local implementation"). The PR page
  was fetched and its rendered HTML contains the commit above and the branch
  name above.
- Current extraction branch: `codex/shuyra-sbi-extraction` (local source
  worktree). This is where this snapshot was taken; it is not claimed to be a
  separate public sbi pull request.
- Extracted path: `crates/sbi`
- Snapshot method: `git archive` of `crates/sbi` at the commit above, plus
  `git show` of the source-root `rust-toolchain.toml` and `Cargo.lock`. The
  source worktree was clean and at `HEAD =
  c970afaf7dae1145633278253351be729b625717` when the snapshot was taken and
  when it was rechecked after validation.

The source commit tracks exactly six files under `crates/sbi`: one
`Cargo.toml` and five Rust files under `src/`. It has no owned tests, no
fixtures, and no cross-directory source projection.

## File mapping

| Upstream at `c970afaf` | Here | State |
| --- | --- | --- |
| `crates/sbi/Cargo.toml` | `Cargo.toml` | rewritten as standalone root, see below |
| `crates/sbi/src/ecall.rs` | `src/ecall.rs` | byte-identical |
| `crates/sbi/src/eid.rs` | `src/eid.rs` | byte-identical |
| `crates/sbi/src/extension.rs` | `src/extension.rs` | byte-identical |
| `crates/sbi/src/fid.rs` | `src/fid.rs` | byte-identical |
| `crates/sbi/src/lib.rs` | `src/lib.rs` | byte-identical |
| source-root `rust-toolchain.toml` | `rust-toolchain.toml` | byte-identical |
| source-root `Cargo.lock` | `Cargo.lock` | copied then minimally pruned, see below |
| none | `.gitignore`, `README.md`, `PROVENANCE.md` | new provider metadata |

All five `src/*.rs` files are byte-for-byte identical to the source commit.
No `tests/` directory is copied because the source has none. The source
root's `Cargo.toml`, `.cargo/` QEMU runner configuration, programs,
`SBI.bin`, and `SBI.elf` are not copied.

The standalone root `Cargo.toml` keeps the package name `sbi`, both original
dependencies with their exact requirements (`erra 0.2.0` and `fack 0.2.0`,
each `default-features = false`), and `[lib] test = false`, `bench = false`,
`doctest = false`. It replaces only the inherited
`version.workspace = true` / `edition.workspace = true` with explicit
`version = "0.1.0"` / `edition = "2024"`, and adds an independent
`[workspace]` with `resolver = "3"`. It contains no `[patch]`, `[replace]`,
local Git dependency, or cache projection.

This crate is an RV64 S-mode ABI wrapper. It is not a Unit and produces no
UnitFile, ELF, or server. The repository-root `SBI.bin` is separate firmware,
not a member of this crate; it was not copied, moved, or replaced. The source
firmware SHA-256 remains
`c53d58aa9ab17f95fa1af159f0bf05fad5c5622ebc0f1084463435fa6e5752b6`.

## Dependency resolution

The root package's direct dependencies are:

- `erra` `0.2.0` from `registry+https://github.com/rust-lang/crates.io-index`
  with default features disabled;
- `fack` `0.2.0` from the same registry with default features disabled.

The actual `cargo metadata --locked --offline` resolution contains ten
packages:

```text
sbi 0.1.0 (path)
erra 0.2.0 (registry)
fack 0.2.0 (registry)
fack-core 0.2.0 (registry)
fack-macro 0.2.0 (registry)
fack-codegen 0.2.0 (registry)
proc-macro2 1.0.107 (registry)
quote 1.0.47 (registry)
syn 2.0.119 (registry)
unicode-ident 1.0.24 (registry)
```

Every registry package keeps the exact version and checksum that were already
present in the source-root `Cargo.lock`; no provider or transitive version was
changed or upgraded:

| Package | Version | Checksum |
| --- | --- | --- |
| `erra` | 0.2.0 | `bc36b9bc9f9fd011eb63c44cdba19d47c4e77f4759cddb0c448b1d23d27abc1e` |
| `fack` | 0.2.0 | `79bf0abbd0418af0e15118cce2ebafbc49d877867f4eeb2fa3120bbf3ca94765` |
| `fack-codegen` | 0.2.0 | `43eb35ab3b590c556df60c7df949560c7df1fbf45bb78ba03246e8218a836410` |
| `fack-core` | 0.2.0 | `3cb29ce2b3ca6df89e4f6b7e0ab461223ee77ee997e21f13469b7474e378e83a` |
| `fack-macro` | 0.2.0 | `e3bb69083fa40ad0cbddacc560efd420208969c7924745c28da0f94e5a836ac0` |
| `proc-macro2` | 1.0.107 | `985e7ec9bb745e6ce6535b544d84d6cd6f7ad8bd711c398938ae983b91a766d9` |
| `quote` | 1.0.47 | `1fbf4db142a473a8d80c26bbf18454ed458bf8d26c8219c331daecfdbd079001` |
| `syn` | 2.0.119 | `872831b642d1a07999a962a351ed35b955ea2cfc8f3862091e2a240a84f17297` |
| `unicode-ident` | 1.0.24 | `e6e4313cd5fcd3dad5cafa179702e2b244f760991f45397d14d4ebf38247da75` |

## Lock infrastructure

The source-root `Cargo.lock` tracked at `c970afaf` has 100 packages and
SHA-256
`2af97d1b738c5d01c75d67855f8d03c33403ef529bcb3fe77395ac3db60b0a05`.
It was copied as the seed, then an actual
`cargo metadata --format-version 1 --offline` run against the fixed standalone
manifest pruned it to the 10 reachable packages above. The resulting
`sbi/Cargo.lock` has SHA-256
`e2b941b5e62fc509ae1051332526364e6ca09e92b64f213a5a646b661753907b`.

The only textual difference inside retained entries is that `fack-codegen`
and `fack-macro` now name `"syn"` without a version suffix. The source lock had
two `syn` package entries (`syn` 2.0.119 and `syn` 3.0.4 used by unrelated
packages), so its dependency lists used the qualified `"syn 2.0.119"`. The
pruned graph retains only `syn` 2.0.119, with the same version and checksum.

The lock hash was identical before and after the locked online builds, before
and after the `--locked --offline` reruns, and after the locked metadata run.
No lock file was ignored.

## Verification records

Provider evidence is under `evidence/pr10/provider/`. It records raw cargo
exit codes and separate stdout/stderr for the locked RISC-V debug and release
builds and the lock-file-preserving offline reruns, the locked metadata run,
the dependency tree, source/lock/package hashes, and the unchanged firmware
hash. It also records the source worktree's clean status and HEAD.

No QEMU run, host build, host test, consumer migration, Git initialization,
commit, push, repository creation, or pull request was performed. The crate has
no owned tests; a zero-test `cargo test` is not reported as ABI coverage.

## Attribution

The source commit `c970afaf7dae1145633278253351be729b625717` was authored and
committed by:

```text
anreonyr <anreonyr@gmail.com>
```

The unique author in the real history of `crates/sbi` is:

```text
anreonyr <anreonyr@gmail.com>
```

The last commit touching the extracted path is
`730b463a21ad5d33baf79d22a4a4d68a44231a65` (2026-09-19, same author). The
wider source repository also contains commits by
`anreonyr <88045926+anreonyr@users.noreply.github.com>`,
`anreonyr <anreonyr@users.noreply.github.com>`,
`anreonyr <anreonyr@local>`, and `Serial Device <anreonyr@gmail.com>`.
The original history remains at the source URL above. This snapshot does not
rewrite or synthesize commit history.

## Licensing

The source project declares no license. At commit
`c970afaf7dae1145633278253351be729b625717`, no tracked `LICENSE`,
`COPYING`, or `NOTICE` file matched the project, and the `sbi` manifest has no
license field. This extraction adds no license text or SPDX metadata; do not
assume a license that the source does not grant.
