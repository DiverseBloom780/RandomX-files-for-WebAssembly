RandomX WebAssembly (WASM)

This repository contains a WebAssembly build of RandomX, the Proof-of-Work algorithm originally developed for Monero and other cryptocurrencies.

It provides two output files:

randomx.wasm → the compiled RandomX library in WebAssembly

randomx.js → the JavaScript glue code to load the WASM module

The build is based on the official RandomX project (BSD-3-Clause License).

📦 Usage
In HTML
<script type="module">
  import initRandomX from './randomx.js';

  const run = async () => {
    const RandomX = await initRandomX();

    // Initialize with a key (seed)
    const key = new TextEncoder().encode("test-key");
    RandomX.ccall("init", null, ["array", "number"], [key, key.length]);

    // Hash input
    const input = new TextEncoder().encode("hello world");
    const outputPtr = RandomX._malloc(32);
    RandomX.ccall("hash", null, ["array", "number", "number"], [input, input.length, outputPtr]);

    // Copy result back into JS
    const result = new Uint8Array(RandomX.HEAPU8.buffer, outputPtr, 32);
    console.log("Hash:", Array.from(result).map(b => b.toString(16).padStart(2, "0")).join(""));

    RandomX._free(outputPtr);
  };

  run();
</script>

In React (Vite, CRA, Next.js, etc.)

Import the module the same way in a component:

import { useEffect } from "react";
import initRandomX from "./randomx.js";

export default function App() {
  useEffect(() => {
    (async () => {
      const RandomX = await initRandomX();
      console.log("RandomX WASM loaded:", RandomX);
    })();
  }, []);

  return <h1>RandomX WebAssembly Example</h1>;
}

⚡ Building From Source

To reproduce this build:

# Clone RandomX
git clone https://github.com/tevador/RandomX
cd RandomX

# Make a build directory
mkdir build-wasm && cd build-wasm

# Ensure emsdk is loaded
source ~/emsdk/emsdk_env.sh

# Configure with Emscripten
emcmake cmake .. -DCMAKE_BUILD_TYPE=Release
emmake make randomx -j$(nproc)

# Compile wrapper
emcc ../wrapper.cpp \
  -I../src \
  librandomx.a \
  -s WASM=1 \
  -s MODULARIZE=1 \
  -s EXPORT_ES6=1 \
  -s EXPORTED_FUNCTIONS="['_init','_hash']" \
  -s EXPORTED_RUNTIME_METHODS="['ccall','cwrap']" \
  -O3 \
  -o randomx.js

📜 License

This project is licensed under the BSD 3-Clause License, the same as the original RandomX.


javascriptconst importObject = {
    a: {
        a: () => {},
        b: () => {},
        c: () => {},
        d: () => {},
        e: () => {},
        f: () => {}
    }
};
This provides stub implementations for all six function imports (a.a through a.f) that your WASM expects.
