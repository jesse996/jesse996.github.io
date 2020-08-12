# WebAssembly


# SSVM 简介

SSVM 是适用于云，AI 和区块链应用程序的高性能，企业级 WebAssembly（WASM）虚拟机。包括以下用例：

-   Node.js 应用程序中 Rust 函数的高性能和安全运行时
-   针对 ONNX AI 模型的硬件优化的运行时
-   适用于领先的区块链平台的智能合约运行时引擎

# 一个简单的 Web App

**功能**：base64 的编码和解码

### 首先在 rust 的 lib 文件中编写实际的用来编码和解码的代码：

```rust
use base64::{decode, encode};
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn base64Encode(s: &str) -> String {
    encode(s)
}

#[wasm_bindgen]
pub fn base64Decode(s: &str) -> String {
    String::from_utf8(decode(s).unwrap()).unwrap()
}

```

### 然后在 node 中调用 rust 代码：

```js
const {
    base64Encode,
    base64Decode,
} = require('../pkg/ssvm_nodejs_starter_lib.js')

const http = require('http')
const url = require('url')
const hostname = '0.0.0.0'
const port = 3000

const server = http.createServer((req, res) => {
    const queryObject = url.parse(req.url, true).query
    if (queryObject['encodeStr']) {
        res.end(base64Encode(queryObject['encodeStr']) + '\n')
    } else if (queryObject['decodeStr']) {
        console.log(queryObject['decodeStr'])
        res.end(base64Decode(queryObject['decodeStr']) + '\n')
    } else {
        res.end(
            `Please use command curl http://${hostname}:${port}/?encodeStr=string
or
http://${hostname}:${port}/?decodeStr=string \n`
        )
    }
})

server.listen(port, hostname, () => {
    console.log(`Server running at http://${hostname}:${port}/`)
})
```

