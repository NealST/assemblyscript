![AS](https://avatars1.githubusercontent.com/u/28916798?s=48) loader
======================

A convenient loader for AssemblyScript modules. Demangles module exports to a friendly object structure compatible with WebIDL and TypeScript definitions and provides some useful utility to read/write data from/to memory.

Usage
-----

```js
const loader = require("@assemblyscript/loader");
...
```

API
---

* **instantiate**<`T`>(module: `WebAssembly.Module`, imports?: `WasmImports`): `ASModule<T>`<br />
  Instantiates an AssemblyScript module using the specified imports.

* **instantiateBuffer**<`T`>(buffer: `Uint8Array`, imports?: `WasmImports`): `ASModule<T>`<br />
  Instantiates an AssemblyScript module from a buffer using the specified imports.

* **instantiateStreaming**<`T`>(response: `Response`, imports?: `WasmImports`): `Promise<ASModule<T>>`<br />
  Instantiates an AssemblyScript module from a response using the sspecified imports.

* **demangle**<`T`>(exports: `WasmExports`): `T`<br />
  Demangles an AssemblyScript module's exports to a friendly object structure. You usually don't have to call this manually as instantiation does this implicitly.

**Note:** `T` above can either be omitted if the structure of the module is unknown, or can reference a `.d.ts` as produced by the compiler with the `-d` option.

Instances of `ASModule` are automatically populated with some useful utility:

* **I8**: `Int8Array`<br />
  An 8-bit signed integer view on the memory.

* **U8**: `Uint8Array`<br />
  An 8-bit unsigned integer view on the memory.

* **I16**: `Int16Array`<br />
  A 16-bit signed integer view on the memory.

* **U16**: `Uint16Array`<br />
  A 16-bit unsigned integer view on the memory.

* **I32**: `Int32Array`<br />
  A 32-bit signed integer view on the memory.

* **U32**: `Uint32Array`<br />
  A 32-bit unsigned integer view on the memory.

* **F32**: `Float32Array`<br />
  A 32-bit float view on the memory.

* **F64**: `Float64Array`<br />
  A 64-bit float view on the memory.

* **newString**(str: `string`): `number`<br />
  Allocates a new string in the module's memory and returns its pointer. Requires `allocate_memory` to be exported from your module's entry file, i.e.:

  ```js
  import "allocator/tlsf";
  export { allocate_memory, free_memory };
  ```

* **getString**(ptr: `number`): `string`<br />
  Gets a string from the module's memory by its pointer.

Examples
--------

### Instantiating a module

```js
// From a module provided as a buffer, i.e. as returned by fs.readFileSync
const myModule = loader.instatiateBuffer(fs.readFileSync("myModule.wasm"), myImports);

// From a response object, i.e. as returned by window.fetch
const myModule = await loader.instantiateStreaming(fetch("myModule.wasm"), myImports);
```

### Reading/writing basic values to/from memory

```js
var ptrToInt8 = ...;
var value = myModule.U16[ptrToInt8]; // alignment of log2(1)=0

var ptrToInt16 = ...;
var value = myModule.U16[ptrToInt16 >>> 1]; // alignment of log2(2)=1

var ptrToInt32 = ...;
var value = myModule.U32[ptrToInt32 >>> 2]; // alignment of log2(4)=2

var ptrToFloat32 = ...;
var value = myModule.F32[ptrToFloat32 >>> 2]; // alignment of log2(4)=2

var ptrToFloat64 = ...;
var value = myModule.F64[ptrToFloat64 >>> 3]; // alignment of log2(8)=3

// Likewise, for writing
myModule.U16[ptrToInt8] = newValue;
myModule.U16[ptrToInt16 >>> 1] = newValue;
myModule.U32[ptrToInt32 >>> 2] = newValue;
myModule.F32[ptrToFloat32 >>> 2] = newValue;
myModule.F64[ptrToFloat64 >>> 3] = newValue;
```

**Note:** Make sure to reference the views as shown above as these will automatically be updated when the module's memory grows.

### Allocating/obtaining strings to/from memory

```js
// Allocating a string, i.e. to be passed to an export expecting one
var str = "Hello world!";
var ptr = module.newString(str);

// Disposing a string that is no longer needed (requires free_memory to be exported)
module.free_memory(ptr);

// Obtaining a string, i.e. as returned by an export
var ptrToString = ...;
var str = module.getString(ptrToString);
```

### Usage with TypeScript definitions produced by the compiler

```ts
import MyModule from "myModule"; // pointing at the d.ts

const myModule = loader.instatiateBuffer<MyModule>(fs.readFileSync("myModule.wasm"), myImports);
```

**Hint:** You can produce a `.d.ts` for your module with the `-d` option on the command line.