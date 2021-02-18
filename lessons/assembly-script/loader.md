---
path: "/assembly-script/loader"
section: "AssemblyScript"
order: "3F"
title: "AssemblyScript Loader"
description: ""

---
```js
// assembly/index.ts
export function fizzbuzz(n: i32): String | null {
  if (n % 15 === 0) {
    return 'fizzbuzz';
  }

  if (n % 3 === 0) {
    return 'fizz';
  }

  if (n % 5 === 0) {
    return 'buzz';
  }

  return null;
}
```

```js
// index.html
<script>
    const WL = new WasmLoader();
    WL.wasm('/build/untouched.wasm')
    .then(instance => {
      const { fizzbuzz } = instance;

      document.write(fizzbuzz(3));
    });
</script>
```

Running `fizzbuzz(3)` outputs a number and not a string. Remember that Web Assembly only deals in numbers so the AssemblyScript creates a form of our strings (see: untouched.wat) and passes them into memory. The number being returned is a pointer to the memory address of the string being returned. Fortunately AssemblyScript includes a [loader](https://www.assemblyscript.org/loader.html#loader) that lets us allocate and read from memory.



Let's import the loader into the page:
```html
// index.html
<script src="https://cdn.jsdelivr.net/npm/@assemblyscript/loader/umd/index.js"></script>
```


In `loader.js` file, replace the `WebAssembly` method calls with `loader`.


```js
// js/loader.js
class WasmLoader {
    constructor() {
       this._imports = {
            env: {
                abort() {
                    throw new Error('Abort called from wasm file');
                }
            }
        };
    }

    async wasm(path, imports = this._imports) {
        console.log(`fetching ${path}`);

        if (!loader.instantiateStreaming) {
            return this.wasmFallback(path, imports);
        }

        const { instance } = await loader.instantiateStreaming(fetch(path), imports);

        return instance?.exports;
    }

    async wasmFallback(path, imports) {
        console.log('using fallback');
        const response = await fetch(path);
        const bytes = await response?.arrayBuffer();
        const { instance } = await loader.instantiate(bytes, imports);

        return instance?.exports;
    }
}