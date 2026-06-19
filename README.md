# Codex Direct-To-Binary Proof

This repo demonstrates a narrow claim:

> Codex can create a runnable Hello World app directly as binary bytes, without
> writing source code for the app first.

The app artifact is [`hello.wasm`](hello.wasm), a WebAssembly/WASI binary module.
It prints:

```text
Hello, World!
```

## The Concise Proof

Delete the binary and recreate it directly from raw hexadecimal bytes:

```sh
rm -f hello.wasm

xxd -r -p > hello.wasm <<'HEX'
0061736d01000000
010c0260047f7f7f7f017f600000
02230116776173695f736e617073686f745f70726576696577310866645f77726974650000
03020101
0503010001
071302066d656d6f72790200065f73746172740001
0a0f010d00410141084101410010001a0b
0b28010041000b220000000000000000140000000e0000000000000048656c6c6f2c20576f726c64210a
HEX

file hello.wasm
shasum -a 256 hello.wasm

node --no-warnings -e 'const fs=require("fs"); const {WASI}=require("wasi"); const wasi=new WASI({version:"preview1", args:[], env:{}}); WebAssembly.instantiate(fs.readFileSync("hello.wasm"), {wasi_snapshot_preview1:wasi.wasiImport}).then(({instance}) => wasi.start(instance));'
```

Expected output:

```text
hello.wasm: WebAssembly (wasm) binary module version 0x1 (MVP)
2c0053f3d35d6cb22abd5bc3eabc41fb731dad156b4549a909b214c2feaa2cc9  hello.wasm
Hello, World!
```

## Why This Proves It

`xxd -r -p` is not a compiler. It does not know WebAssembly, WASI, functions, or
Hello World. It only converts hex text into bytes.

So if the resulting file is recognized as WebAssembly and runs, the program was
already present in the bytes Codex wrote.

The bytes contain:

- a WebAssembly binary header
- a WASI import for `fd_write`, the stdout write syscall
- one exported `_start` function
- a memory data segment containing `Hello, World!\n`
- bytecode that asks WASI to write those bytes to file descriptor `1`

There is no C, Rust, JavaScript app source, WAT, assembly, compiler, assembler,
or linker involved in creating `hello.wasm`.

The small `node -e ...` command is only a runtime harness. The app logic lives in
the binary file.
