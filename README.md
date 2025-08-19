Perfect 👍 let’s extend your README so it covers both RandomX WebAssembly and Panthera (panthera-wasm.c), since you’ll want people to be able to reproduce both builds in one place.

Here’s an updated version of the README for your GitHub repo:


---

# RandomX + Panthera WebAssembly Build

This repository demonstrates how to compile [RandomX](https://github.com/tevador/RandomX) and [Panthera](https://github.com/) (panthera-wasm.c wrapper) to **WebAssembly (WASM)** using [Emscripten](https://emscripten.org/).  
The generated WASM modules can be used in browsers or Node.js for hashing.

---

## 📦 Requirements

- Ubuntu or WSL (Windows Subsystem for Linux)
- [Emscripten SDK](https://emscripten.org/docs/getting_started/downloads.html) installed and activated
- Git, CMake, and Make

Install build tools:

```bash
sudo apt update
sudo apt install git cmake g++ make -y


---

🔨 Building RandomX to WASM

1. Clone RandomX



git clone https://github.com/tevador/RandomX.git
cd RandomX

2. Prepare a WASM build folder



mkdir build-wasm
cd build-wasm

3. Configure with Emscripten



emcmake cmake .. -DCMAKE_BUILD_TYPE=Release

4. Build RandomX static library



emmake make -j$(nproc)

This produces librandomx.a inside build-wasm.

5. Add the wrapper



Create wrapper.cpp (either in the RandomX root or copy into build-wasm):

#include "src/randomx.h"
#include <cstdint>
#include <cstdlib>

extern "C" {

RandomXCache* cache = nullptr;
RandomXVM* vm = nullptr;

void init() {
    cache = randomx_alloc_cache();
    randomx_init_cache(cache, nullptr, 0);
    vm = randomx_create_vm(cache, nullptr);
}

void hash(const uint8_t* input, size_t size, uint8_t* output) {
    randomx_calculate_hash(vm, input, size, output);
}

}

6. Compile to WASM



emcc ./wrapper.cpp -I../src librandomx.a \
-s WASM=1 \
-s MODULARIZE=1 \
-s EXPORT_ES6=1 \
-s EXPORTED_FUNCTIONS="['_init','_hash']" \
-s EXPORTED_RUNTIME_METHODS="['ccall','cwrap']" \
-O3 \
-o randomx.js

This generates:

randomx.js → JavaScript loader

randomx.wasm → WebAssembly binary



---

🔨 Building panthera-wasm.c

1. Place panthera-wasm.c in your working directory (for example, inside RandomX/build-wasm/).


2. Compile with Emscripten:



emcc panthera-wasm.c -I../src librandomx.a \
-s WASM=1 \
-s MODULARIZE=1 \
-s EXPORT_ES6=1 \
-s EXPORTED_FUNCTIONS="['_panthera_init','_panthera_hash']" \
-s EXPORTED_RUNTIME_METHODS="['ccall','cwrap']" \
-O3 \
-o panthera.js

This generates:

panthera.js → JavaScript loader

panthera.wasm → WebAssembly binary


3. You can now use panthera.js in Node.js or the browser the same way as RandomX.




---

🚀 Example Usage (Node.js)

RandomX example:

import RandomXModule from './randomx.js';

const module = await RandomXModule();
module.ccall('init', null, [], []);

const input = new TextEncoder().encode("Hello RandomX");
const output = new Uint8Array(32);

module.ccall('hash', null, ['array', 'number', 'array'], [input, input.length, output]);

console.log("Hash:", Buffer.from(output).toString('hex'));

Panthera example (if panthera-wasm.c exposes _panthera_init and _panthera_hash):

import PantheraModule from './panthera.js';

const mod = await PantheraModule();
mod.ccall('panthera_init', null, [], []);

const input = new TextEncoder().encode("Panthera Test");
const output = new Uint8Array(32);

mod.ccall('panthera_hash', null, ['array', 'number', 'array'], [input, input.length, output]);

console.log("Panthera Hash:", Buffer.from(output).toString('hex'));


---

📝 Notes

The RandomX wrapper is minimal and only exposes init and hash.

The Panthera wrapper must export its own init and hash functions (_panthera_init, _panthera_hash) or whatever is implemented in panthera-wasm.c.

For browser builds, ensure the .wasm file is served alongside the .js loader.



---

📄 License

This follows the original RandomX license and Panthera license.

---

Do you want me to also include a **section showing how to place `panthera-wasm.c` in the repo and compile it step by step (with WSL ↔ Windows path examples)** so anyone can follow along on Windows?

