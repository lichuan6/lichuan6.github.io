# Anchor helloworld

<!-- toc -->

- [Install Rust](#install-rust)
- [Install solana](#install-solana)
- [Install Anchor](#install-anchor)
- [Init anchor project](#init-anchor-project)
- [Build anchor project - but failed ❌](#build-anchor-project---but-failed-%E2%9D%8C)
- [Try to fix the build error](#try-to-fix-the-build-error)
- [Write anchor program](#write-anchor-program)
- [Write anchor test](#write-anchor-test)
- [Anchor build](#anchor-build)
- [Anchor test](#anchor-test)
- [Run solana-test-validator](#run-solana-test-validator)
- [Anchor deploy](#anchor-deploy)
- [What happend when deploy](#what-happend-when-deploy)
- [Program log](#program-log)
- [Airdrop](#airdrop)

<!-- tocstop -->

# Install Rust

```bash
# install rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
export PATH="$HOME/.cargo/bin:$PATH"
```

# Install solana

```bash
# install solana
# NOTE: Using 1.18.4 will cause `anchor build` error, so we choose v1.18.8
# sh -c "$(curl -sSfL https://release.solana.com/v1.18.4/install)"
sh -c "$(curl -sSfL https://release.solana.com/v1.18.8/install)"
```

# Install Anchor

Anchor is a framework for Solana development

```bash
# install anchor
cargo install --git https://github.com/coral-xyz/anchor avm --locked --force
avm install latest
avm use latest
```

# Init anchor project

```bash
# init new project
anchor init helloworld
```

# Build anchor project - but failed ❌

Now, let's build the anchor project.

```bash
# error
❯ anchor build
warning: some crates are on edition 2021 which defaults to `resolver = "2"`, but virtual workspaces default to `resolver = "1"`
note: to keep the current resolver, specify `workspace.resolver = "1"` in the workspace root's manifest
note: to use the edition 2021 resolver, specify `workspace.resolver = "2"` in the workspace root's manifest
error: package `solana-program v1.18.8` cannot be built because it requires rustc 1.75.0 or newer, while the currently active rustc version is 1.72.0-dev
Either upgrade to rustc 1.75.0 or newer, or use
cargo update -p solana-program@1.18.8 --precise ver
where `ver` is the latest version of `solana-program` supporting rustc 1.72.0-dev
```

Oops! We have an error!

# Try to fix the build error

I tried to run the following commands but no luck.

```bash
# To check the version of rustc from Solana Tools, runs:
# see https://solana.stackexchange.com/questions/7077/anchor-build-says-cannot-be-built-because-it-requires-rustc-1-68-0-or-newer-bu
❯ cargo-build-sbf --version
solana-cargo-build-sbf 1.18.4
platform-tools v1.39
rustc 1.72.0

# Upgrade your Solana tools
solana-install init 1.18.8

# To check the version of rustc from Solana Tools, runs:
❯ cargo-build-sbf --version
solana-cargo-build-sbf 1.18.8
platform-tools v1.41
```

After struggling with the error message, I found there was no `rust` in `platform-tools` directory, only an uncompressed tar ball.

```bash
# anchor build error again
~/.local/share/solana/install/releases/1.18.8/solana-release/bin/sdk/sbf/dependencies/platform-tools
drwxr-xr-x    - hana 28 Mar 10:36 llvm
.rw-r--r-- 291M hana 28 Mar 10:36 platform-tools-osx-x86_64.tar.bz2
.rw-r--r--  240 hana 27 Feb 04:59 version.md
```

Instinctively, I extract `platform-tools-osx-x86_64.tar.bz2` file manually.

```bash
# Manually extract platform-tools-osx-x86_64.tar.bz2
cd ~/.local/share/solana/install/releases/1.18.8/solana-release/bin/sdk/sbf/dependencies/platform-tools
tar xf platform-tools-osx-x86_64.tar.bz2
```

After the file is extracted, I run `anchor build` again and it succeeds! 🎉

# Write anchor program

Now, it's time to write some `Rust` code(and `TypeScript`)!

Modify `programs/helloworld/lib.rs` to print message to `initialize` function.

```rust
use anchor_lang::prelude::*;

declare_id!("4d9Riew7cGvjPUzpjDnDSkXdt7qzW67qT84Ef1wpwEMx");

#[program]
pub mod helloworld {
    use super::*;

    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        msg!("Hello world, from solana smart contract");
        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize {}
```

# Write anchor test

Anchor will generate a test case for `helloworld` project.

Below is the generated test case in `tests/helloworld.ts` file.

```typescript
import * as anchor from "@coral-xyz/anchor";
import { Program } from "@coral-xyz/anchor";
import { Helloworld } from "../target/types/helloworld";

describe("helloworld", () => {
  // Configure the client to use the local cluster.
  anchor.setProvider(anchor.AnchorProvider.env());

  const program = anchor.workspace.Helloworld as Program<Helloworld>;

  it("Is initialized!", async () => {
    // Add your test here.
    const tx = await program.methods.initialize().rpc();
    console.log("Your transaction signature", tx);
  });
});
```

# Anchor build

Now, we can build the anchor project.

```bash
❯ anchor build
warning: virtual workspace defaulting to `resolver = "1"` despite one or more workspace members being on edition 2021 which implies `resolver = "2"`
note: to keep the current resolver, specify `workspace.resolver = "1"` in the workspace root's manifest
note: to use the edition 2021 resolver, specify `workspace.resolver = "2"` in the workspace root's manifest
note: for more details see https://doc.rust-lang.org/cargo/reference/resolver.html#resolver-versions
   Compiling proc-macro2 v1.0.79
   Compiling unicode-ident v1.0.12
   Compiling version_check v0.9.4
   Compiling serde v1.0.197
   Compiling typenum v1.17.0
   Compiling syn v1.0.109
   Compiling thiserror v1.0.58
   Compiling serde_json v1.0.115
   Compiling semver v1.0.22
   Compiling anyhow v1.0.81
   Compiling subtle v2.5.0
   Compiling generic-array v0.14.7
   Compiling itoa v1.0.11
   Compiling ryu v1.0.17
   Compiling cfg-if v1.0.0
   Compiling quote v1.0.35
   Compiling syn v2.0.55
   Compiling unicode-segmentation v1.11.0
   Compiling cpufeatures v0.2.12
   Compiling libc v0.2.153
   Compiling heck v0.3.3
   Compiling rustc_version v0.4.0
   Compiling bs58 v0.5.1
   Compiling proc-macro-error-attr v1.0.4
   Compiling autocfg v1.2.0
   Compiling once_cell v1.19.0
   Compiling proc-macro-error v1.0.4
   Compiling hashbrown v0.14.3
   Compiling equivalent v1.0.1
   Compiling ahash v0.7.8
   Compiling ahash v0.8.11
   Compiling winnow v0.5.40
   Compiling toml_datetime v0.6.5
   Compiling feature-probe v0.1.1
   Compiling indexmap v2.2.6
   Compiling bv v0.11.1
   Compiling jobserver v0.1.28
   Compiling solana-frozen-abi-macro v1.18.8
   Compiling cc v1.0.90
   Compiling rustversion v1.0.14
   Compiling zerocopy v0.7.32
   Compiling cfg_aliases v0.1.1
   Compiling borsh v1.4.0
   Compiling memoffset v0.9.1
   Compiling num-traits v0.2.18
   Compiling solana-frozen-abi v1.18.8
   Compiling hashbrown v0.11.2
   Compiling hashbrown v0.13.2
   Compiling toml_edit v0.21.1
   Compiling arrayvec v0.7.4
   Compiling lazy_static v1.4.0
   Compiling bs58 v0.4.0
   Compiling borsh-derive-internal v0.10.3
   Compiling borsh-schema-derive-internal v0.10.3
   Compiling borsh-schema-derive-internal v0.9.3
   Compiling proc-macro-crate v3.1.0
   Compiling borsh-derive-internal v0.9.3
   Compiling arrayref v0.3.7
   Compiling log v0.4.21
   Compiling either v1.10.0
   Compiling keccak v0.1.5
   Compiling constant_time_eq v0.3.0
   Compiling itertools v0.10.5
   Compiling getrandom v0.2.12
   Compiling base64 v0.13.1
   Compiling blake3 v1.5.1
   Compiling solana-program v1.18.8
   Compiling serde_derive v1.0.197
   Compiling thiserror-impl v1.0.58
   Compiling syn_derive v0.1.8
   Compiling bytemuck_derive v1.6.0
   Compiling solana-sdk-macro v1.18.8
   Compiling num-derive v0.4.2
   Compiling anchor-derive-space v0.29.0
   Compiling borsh-derive v1.4.0
   Compiling bytemuck v1.15.0
   Compiling serde_bytes v0.11.14
   Compiling bincode v1.3.3
   Compiling toml v0.5.11
   Compiling block-buffer v0.10.4
   Compiling crypto-common v0.1.6
   Compiling digest v0.10.7
   Compiling sha2 v0.10.8
   Compiling sha3 v0.10.8
   Compiling proc-macro-crate v0.1.5
   Compiling anchor-syn v0.29.0
   Compiling borsh-derive v0.9.3
   Compiling borsh-derive v0.10.3
   Compiling borsh v0.10.3
   Compiling borsh v0.9.3
   Compiling anchor-derive-accounts v0.29.0
   Compiling anchor-attribute-account v0.29.0
   Compiling anchor-derive-serde v0.29.0
   Compiling anchor-attribute-access-control v0.29.0
   Compiling anchor-attribute-program v0.29.0
   Compiling anchor-attribute-error v0.29.0
   Compiling anchor-attribute-event v0.29.0
   Compiling anchor-attribute-constant v0.29.0
   Compiling anchor-lang v0.29.0
   Compiling helloworld v0.1.0 (/Users/hana/tmp/helloworld/programs/helloworld)
warning: unused variable: `ctx`
 --> programs/helloworld/src/lib.rs:9:23
  |
9 |     pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
  |                       ^^^ help: if this is intentional, prefix it with an underscore: `_ctx`
  |
  = note: `#[warn(unused_variables)]` on by default

warning: `helloworld` (lib) generated 1 warning (run `cargo fix --lib -p helloworld` to apply 1 suggestion)
    Finished release [optimized] target(s) in 3m 30s
+ wget https://github.com/Snaipe/Criterion/releases/download/v2.3.2/criterion-v2.3.2-osx-x86_64.tar.bz2 -O criterion-v2.3.2-osx-x86_64.tar.bz2 --progress=dot:giga --retry-connrefused --read-timeout=30
--2024-03-28 11:00:29--  https://github.com/Snaipe/Criterion/releases/download/v2.3.2/criterion-v2.3.2-osx-x86_64.tar.bz2
Resolving github.com (github.com)... 20.205.243.166
Connecting to github.com (github.com)|20.205.243.166|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/30111969/33613116-1c60-11e7-8934-6166ffd90477?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240328%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240328T030030Z&X-Amz-Expires=300&X-Amz-Signature=4122d55871f80d3351bb33cb84c3a81a36f7e9b28b2acec18bba5ecaf72b73e8&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=30111969&response-content-disposition=attachment%3B%20filename%3Dcriterion-v2.3.2-osx-x86_64.tar.bz2&response-content-type=application%2Foctet-stream [following]
--2024-03-28 11:00:30--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/30111969/33613116-1c60-11e7-8934-6166ffd90477?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAVCODYLSA53PQK4ZA%2F20240328%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20240328T030030Z&X-Amz-Expires=300&X-Amz-Signature=4122d55871f80d3351bb33cb84c3a81a36f7e9b28b2acec18bba5ecaf72b73e8&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=30111969&response-content-disposition=attachment%3B%20filename%3Dcriterion-v2.3.2-osx-x86_64.tar.bz2&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.111.133, 185.199.108.133, 185.199.110.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.111.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 180979 (177K) [application/octet-stream]
Saving to: ‘criterion-v2.3.2-osx-x86_64.tar.bz2’

     0K                                    100%  587K=0.3s

2024-03-28 11:00:31 (587 KB/s) - ‘criterion-v2.3.2-osx-x86_64.tar.bz2’ saved [180979/180979]

+ tar --strip-components 1 -jxf criterion-v2.3.2-osx-x86_64.tar.bz2
+ ./platform-tools/rust/bin/rustc --version
+ ./platform-tools/rust/bin/rustc --print sysroot
+ set +e
+ rustup toolchain uninstall solana
info: uninstalling toolchain 'solana'
info: toolchain 'solana' uninstalled
+ set -e
+ rustup toolchain link solana platform-tools/rust
+ exit 0
```

# Anchor test

```bash
anchor test
warning: virtual workspace defaulting to `resolver = "1"` despite one or more workspace members being on edition 2021 which implies `resolver = "2"`
note: to keep the current resolver, specify `workspace.resolver = "1"` in the workspace root's manifest
note: to use the edition 2021 resolver, specify `workspace.resolver = "2"` in the workspace root's manifest
note: for more details see https://doc.rust-lang.org/cargo/reference/resolver.html#resolver-versions
warning: unused variable: `ctx`
 --> programs/helloworld/src/lib.rs:9:23
  |
9 |     pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
  |                       ^^^ help: if this is intentional, prefix it with an underscore: `_ctx`
  |
  = note: `#[warn(unused_variables)]` on by default

warning: `helloworld` (lib) generated 1 warning (run `cargo fix --lib -p helloworld` to apply 1 suggestion)
    Finished release [optimized] target(s) in 0.24s

Found a 'test' script in the Anchor.toml. Running it as a test suite!

Running test suite: "/Users/hana/tmp/helloworld/Anchor.toml"

yarn run v1.22.21
warning package.json: No license field
$ /Users/hana/tmp/helloworld/node_modules/.bin/ts-mocha -p ./tsconfig.json -t 1000000 'tests/**/*.ts'
(node:87299) [DEP0040] DeprecationWarning: The `punycode` module is deprecated. Please use a userland alternative instead.
(Use `node --trace-deprecation ...` to show where the warning was created)


  helloworld
Your transaction signature 2TWQV8ftHjohurqV2WtFqKCn7VwseRjBfTwKPBLYqa8x3A54joXjDt1s6gWfMFMi8JNEbB4XuAvHnTthCGn8n5oq
    ✔ Is initialized! (575ms)


  1 passing (577ms)

✨  Done in 4.07s.
```

🎉🎉🎉

# Run solana-test-validator

The program is built successfully, we can deploy it in localnet.

Before deploying in localnet, you have to ensure solana local validate is up and running.

Run `solana-test-validator` to start solana local validator:

```bash
❯ solana-test-validator
--faucet-sol argument ignored, ledger already exists
Ledger location: test-ledger
Log: test-ledger/validator.log
⠐ Initializing...                                                                                                                                         Waiting for fees to stabilize 1...
Identity: DB867gXPUPZXZ1epZe1u6M6sms1Ybe7ni5v5wK6iyvPh
Genesis Hash: 7CtbpyodydkFzXTyELyBb1GTrZQFXSvus4mHrDVDBX4z
Version: 1.18.8
Shred Version: 10839
Gossip Address: 127.0.0.1:1024
TPU Address: 127.0.0.1:1027
JSON RPC URL: http://127.0.0.1:8899
⠤ 02:00:48 | Processed Slot: 13441 | Confirmed Slot: 13441 | Finalized Slot: 13409 | Full Snapshot Slot: 13404 | Incremental Snapshot Slot: - | Transactions: 13614 |
```

NOTE: You may need to turn off proxy when you encounter error like `localhost:8899 connect refused`.

```bash
Genesis Hash: 7CtbpyodydkFzXTyELyBb1GTrZQFXSvus4mHrDVDBX4z
Version: 1.18.8
Shred Version: 10839
Gossip Address: 127.0.0.1:1024
TPU Address: 127.0.0.1:1027
JSON RPC URL: http://127.0.0.1:8899
WebSocket PubSub URL: ws://127.0.0.1:8900
  RPC connection failure: error sending request for url (http://127.0.0.1:8899/): connection closed before message completed
Identity: DB867gXPUPZXZ1epZe1u6M6sms1Ybe7ni5v5wK6iyvPh
Genesis Hash: 7CtbpyodydkFzXTyELyBb1GTrZQFXSvus4mHrDVDBX4z
  RPC connection failure: error sending request for url (http://127.0.0.1:8899/): connection closed before message completed
Identity: DB867gXPUPZXZ1epZe1u6M6sms1Ybe7ni5v5wK6iyvPh
Genesis Hash: 7CtbpyodydkFzXTyELyBb1GTrZQFXSvus4mHrDVDBX4z
```

# Anchor deploy

The program is built successfully, we can deploy it in localnet.

Before deploying in localnet, you have to ensure solana local validate is up and running.

Config `Anchor.toml` to use `Localnet` in `provider`.

```toml
[toolchain]

[features]
seeds = false
skip-lint = false

[programs.localnet]
helloworld = "4d9Riew7cGvjPUzpjDnDSkXdt7qzW67qT84Ef1wpwEMx"

[registry]
url = "https://api.apr.dev"

[provider]
cluster = "Localnet" # <-- 🙋 use localnet
# cluster = "devnet"
# cluster = "testnet"
wallet = "/Users/hana/.config/solana/id.json"

[scripts]
test = "yarn run ts-mocha -p ./tsconfig.json -t 1000000 tests/**/*.ts"
```

Before deploying anchor program, we can run `solana balance` to check the balance.

```bash
❯ solana balance
500000002 SOL
```

Run `anchor deploy` to deploy `helloworld` program to local net.

```bash
❯ anchor deploy
Deploying cluster: http://localhost:8899
Upgrade authority: /Users/hana/.config/solana/id.json
Deploying program "helloworld"...
Program path: /Users/hana/tmp/helloworld/target/deploy/helloworld.so...
⠂  27.4% | Resent 130 transactions...               [block height 1381; re-sign in 300 blocks]
Program Id: 4d9Riew7cGvjPUzpjDnDSkXdt7qzW67qT84Ef1wpwEMx

Deploy success
```

After deploy, we can check the balance again.

```
❯ solana balance
500000000.740320265 SOL
```

# What happend when deploy

In order to check how much sol is needed to deploy the program, we can change `provider` to `testnet` and run `anchor deploy`. As we do not have any sol in testnet, `anchor deploy` will output the error: `Error: Account Hupie9P3m69DYALjhjcxwqpTWAoAr36NUXHzcHr7jPpc has insufficient funds for spend (1.25762328 SOL) + fee (0.000915 SOL)`

Below is attempt to deploy helloworld program to testnet.

```bash
❯ anchor deploy
Deploying cluster: https://api.testnet.solana.com
Upgrade authority: /Users/hana/.config/solana/id.json
Deploying program "helloworld"...
Program path: /Users/hana/tmp/helloworld/target/deploy/helloworld.so...
========================================================================
Recover the intermediate account's ephemeral keypair file with
`solana-keygen recover` and the following 12-word seed phrase:
========================================================================
high goddess rib arrange execute right dragon dutch pact edit razor ugly
========================================================================
To resume a deploy, pass the recovered keypair as the
[BUFFER_SIGNER] to `solana program deploy` or `solana program write-buffer'.
Or to recover the account's lamports, pass it as the
[BUFFER_ACCOUNT_ADDRESS] argument to `solana program close`.
========================================================================
Error: Account Hupie9P3m69DYALjhjcxwqpTWAoAr36NUXHzcHr7jPpc has insufficient funds for spend (1.25762328 SOL) + fee (0.000915 SOL)
There was a problem deploying: Output { status: ExitStatus(unix_wait_status(256)), stdout: "", stderr: "" }.
```

# Program log

Go to `.anchor/program-logs/` to inspect program's log.

```bash
❯ ct .anchor/program-logs//4d9Riew7cGvjPUzpjDnDSkXdt7qzW67qT84Ef1wpwEMx.helloworld.log
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
Streaming transaction logs mentioning 4d9Riew7cGvjPUzpjDnDSkXdt7qzW67qT84Ef1wpwEMx. Confirmed commitment
Transaction executed in slot 8:
  Signature: 2TWQV8ftHjohurqV2WtFqKCn7VwseRjBfTwKPBLYqa8x3A54joXjDt1s6gWfMFMi8JNEbB4XuAvHnTthCGn8n5oq
  Status: Ok
  Log Messages:
    Program 4d9Riew7cGvjPUzpjDnDSkXdt7qzW67qT84Ef1wpwEMx invoke [1]
    Program log: Instruction: Initialize
    Program log: Hello world, from solana smart contract
    Program 4d9Riew7cGvjPUzpjDnDSkXdt7qzW67qT84Ef1wpwEMx consumed 324 of 200000 compute units
    Program 4d9Riew7cGvjPUzpjDnDSkXdt7qzW67qT84Ef1wpwEMx success
```

# Airdrop

You can go to solona faucet to claim sol token.

https://faucet.solana.com/

After deploy in testnet, we can check transaction in solona exploer.

https://explorer.solana.com/address/Hupie9P3m69DYALjhjcxwqpTWAoAr36NUXHzcHr7jPpc?cluster=testnet

You can also inspect the transaction.

https://explorer.solana.com/tx/43YmCZWeUHDkvtqGf6tidbTi3wiynzBc64TR1jmrcqhY5kzSqSSc3A8p6Q4mKkG8hzdy7uNzkbQ6Q2ngzA5rsP4f?cluster=testnet

![solana helloworld](https://github.com/lichuan6/i/blob/main/solana/helloworld.png?raw=true)
