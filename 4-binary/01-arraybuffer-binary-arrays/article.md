# ArrayBuffer, arrays binarios

En desarrollo web nos encontramos con datos binarios generalmente cuando tratamos con archivos (crear, subir, descargar). Otro uso típico es el tratamiento digital de imágenes.

Todo esto es posible en JavaScript, y las operaciones binarias ofrecen un muy buen rendimiento.

De igual forma, hay un poco de confusión al respecto, ya que existen muchas clases, por ejemplo:
- `ArrayBuffer`, `Uint8Array`, `DataView`, `Blob`, `File`, etc.

Los datos binarios en JavaScript estan implementados en una forma no estandard, comparado con otros lenguajes de programación. Pero una vez que entendemos las cosas, todo se torna más fácil.
**El objeto binario básico es `ArrayBuffer` -- una referencia a un área de memoria de tamaño fijo.**

Lo creamos de la siguiente manera:
```js run
let buffer = new ArrayBuffer(16); // crea un buffer de tamaño 16
alert(buffer.byteLength); // 16
```

Esto asigna un area de memoria contigua de 16 bits y la rellena con ceros.

```warn header="`ArrayBuffer` no es un arrego de algo"
Para eliminar un poco de confusión que pueda generarse `ArrayBuffer` no tiene nada en común con `Array`:
- Tiene un tamaño fijo, no podemos aumentarlo ni disminuirlo.
- Toma exactamente ese mismo espacio en la memoria. 
- Para acceder a bytes individuales, se necesita otro objeto "view", no `buffer[index]`.
```

`ArrayBuffer` es un área de memoria. ¿Qué se almacena en él? Solo una secuencia "cruda" de bytes

**Para manipular un `ArrayBuffer`, necesitamos usar un objeto "view".**

Un objeto view no allmacena nada por si mismo. Son los "anteojos" los que dan una interpretación de los bytes almacenados en el `ArrayBuffer`.

Por ejemplo:

- **`Uint8Array`** -- trata cada byte en `ArrayBuffer` como un entero, con un valor entre 0 y 255 (es un entero de 8bits, no puede almacenar más que eso). Dicho valor es llamado "Entero de 8 bits sin signo".
- **`Uint16Array`** -- trata cada 2 bytes en `ArrayBuffer` como un número, con un valor entre 0 y 65535 . Dicho valor es llamado "Entero de 16 bits sin signo".
- **`Uint32Array`** -- trata cada 5 bytes en `ArrayBuffer` como un entero, con un valor entre 0 y 4294967295 . Dicho valor es llamado "Entero de 32 bits sin signo".
- **`Float64Array`** -- trata cada 8 bytes en `ArrayBuffer` como un número float, con un valor entre <code>5.0x10<sup>-324</sup></code> to <code>1.8x10<sup>308</sup></code>.

Por lo tanto, los datos binarios en un `ArrayBuffer` de 16 bytes se pueden interpretar como 16" números diminutos ", o 8 números más grandes (2 bytes cada uno), o 4 incluso más grandes (4 bytes cada uno), o 2 valores de punto flotante con alta precisión (8 bytes cada uno).



![](arraybuffer-views.svg)

`ArrayBuffer` es el objeto central, la raíz de todo, los datos binarios sin procesar.

Pero si vamos a escribir en él, o iterar sobre él, básicamente para casi cualquier operación, debemos usar un objeto view, por ejemplo:

```js run
let buffer = new ArrayBuffer(16); // crea un búfer de tamaño 16

*!*
let view = new Uint32Array(buffer); // trata el búfer como una secuencia de 32 enteros

alert(Uint32Array.BYTES_PER_ELEMENT); // 4 bytes por entero
*/!*

alert(view.length); // 4, almacena a todos los enteros
alert(view.byteLength); // 16, tamaño en bytes

// escribamos un valor
view[0] = 123456;

// iteraremos en los valores
for(let num of view) {
  alert(num); // 123456, then 0, 0, 0 (4 values total)
}

```

## TypedArray

El término común de todas estas views (`Uint8Array`, `Uint32Array`, etc) es [TypedArray](https://tc39.github.io/ecma262/#sec-typedarray-objects). Ellas comparten el mismo set de propiedades y métodos

Nótese que no hay un constructor llamado `TypedArray`, solo es un termino "umbrella" común para representar una de estas views en `ArrayBuffer`: `Int8Array`, `Uint8Array` y así sucesivamente, pronto aparecerá la lista completa. 

Cuando vea algo como `new TypedArray`, significa cualquiera de` new Int8Array`, `new Uint8Array`, etc.

Los arreglos escritas se comportan como arreglos regulares: tienen índices y son iterables.

Un constructor de arreglo con tipo (ya sea `Int8Array` o` Float64Array`, no importa) se comporta de manera diferente dependiendo de los tipos de argumentos.

Hay 5 variantes de argumentos:
```js
new TypedArray(buffer, [byteOffset], [length]);
new TypedArray(object);
new TypedArray(typedArray);
new TypedArray(length);
new TypedArray();
```

1. Si se proporciona un argumento `ArrayBuffer`, la instancia se crea sobre él. Ya usamos esa sintaxis.

     Opcionalmente, podemos proporcionar `byteOffset` para comenzar desde (0 por defecto) y la` longitud` (hasta el final del búfer por defecto), entonces la instancia cubrirá solo una parte del `búfer`.

2. Si se proporciona un "Array", o cualquier objeto similar a un arreglo, crea un arreglo con tipo de la misma longitud y copia el contenido.

     Podemos usarlo para rellenar previamente el arreglo con los datos: 
    ```js run
    *!*
    let arr = new Uint8Array([0, 1, 2, 3]);
    */!*
    alert( arr.length ); // 4, created binary array of the same length
    alert( arr[1] ); // 1, filled with 4 bytes (unsigned 8-bit integers) with given values
    ```
    
3. Si se proporciona otro `TypedArray`, hace lo mismo: crea un arreglo con tipo de la misma longitud y copia los valores. Los valores se convierten al nuevo tipo en el proceso, si es necesario. 
    ```js run
    let arr16 = new Uint16Array([1, 1000]);
    *!*
    let arr8 = new Uint8Array(arr16);
    */!*
    alert( arr8[0] ); // 1
    alert( arr8[1] ); // 232, trata de copiar 1000, pero no puede guardar 1000 dentro de 8 bits (como se habia explicado más arriba)
    ```

4. Para un argumento numérico `length` - crea el conjunto de tipos para contener esa cantidad de elementos. Su longitud de bytes será "longitud" multiplicada por el número de bytes en un solo elemento
  `TypedArray.BYTES_PER_ELEMENT`:
    ```js run
    let arr = new Uint16Array(4); // create typed array for 4 integers
    alert( Uint16Array.BYTES_PER_ELEMENT ); // 2 bytes per integer
    alert( arr.byteLength ); // 8 (size in bytes)
    ```

5. Sin argumentos, crea un arreglo con tipo de longitud cero.

Podemos crear un `TypedArray` directamente, sin mencionar` ArrayBuffer`. Pero una vista no puede existir sin un `ArrayBuffer` subyacente, por lo que se crea automáticamente en todos estos casos excepto en el primero (cuando se proporciona).

Para acceder al `ArrayBuffer`, hay propiedades:
- `arr.buffer` - hace referencia al` ArrayBuffer`.
- `arr.byteLength` - la longitud del` ArrayBuffer`.

Entonces, siempre podemos pasar de una vista a otra:
```js
let arr8 = new Uint8Array([0, 1, 2, 3]);

// another view on the same data
let arr16 = new Uint16Array(arr8.buffer);
```


Aquí está la lista de matrices escritas:

- `Uint8Array`,` Uint16Array`, `Uint32Array` - para números enteros de 8, 16 y 32 bits.
   - `Uint8ClampedArray` - para enteros de 8 bits, los" fija "en la asignación (ver más abajo).
- `Int8Array`,` Int16Array`, `Int32Array` - para números enteros con signo (puede ser negativo).
- `Float32Array`,` Float64Array` - para números de coma flotante firmados de 32 y 64 bits.

`` `warn header =" No `int8` o tipos similares de un solo valor"
Tenga en cuenta que, a pesar de los nombres como `Int8Array`, no hay un tipo de valor único como` int` o `int8` en JavaScript.

Eso es lógico, ya que `Int8Array` no es u de estos valores individuales, sino más bien una vista de` ArrayBuffer`.
''
### Out-of-bounds behavior

What if we attempt to write an out-of-bounds value into a typed array? There will be no error. But extra bits are cut-off.

For instance, let's try to put 256 into `Uint8Array`. In binary form, 256 is `100000000` (9 bits), but `Uint8Array` only provides 8 bits per value, that makes the available range from 0 to 255.

For bigger numbers, only the rightmost (less significant) 8 bits are stored, and the rest is cut off:

![](8bit-integer-256.svg)

So we'll get zero.

For 257, the binary form is `100000001` (9 bits), the rightmost 8 get stored, so we'll have `1` in the array:

![](8bit-integer-257.svg)

In other words, the number modulo 2<sup>8</sup> is saved.

Here's the demo:

```js run
let uint8array = new Uint8Array(16);

let num = 256;
alert(num.toString(2)); // 100000000 (binary representation)

uint8array[0] = 256;
uint8array[1] = 257;

alert(uint8array[0]); // 0
alert(uint8array[1]); // 1
```

`Uint8ClampedArray` is special in this aspect, its behavior is different. It saves 255 for any number that is greater than 255, and 0 for any negative number. That behavior is useful for image processing.

## TypedArray methods

`TypedArray` has regular `Array` methods, with notable exceptions.

We can iterate, `map`, `slice`, `find`, `reduce` etc.

There are few things we can't do though:

- No `splice` -- we can't "delete" a value, because typed arrays are views on a buffer, and these are fixed, contiguous areas of memory. All we can do is to assign a zero.
- No `concat` method.

There are two additional methods:

- `arr.set(fromArr, [offset])` copies all elements from `fromArr` to the `arr`, starting at position `offset` (0 by default).
- `arr.subarray([begin, end])` creates a new view of the same type from `begin` to `end` (exclusive). That's similar to `slice` method (that's also supported), but doesn't copy anything -- just creates a new view, to operate on the given piece of data.

These methods allow us to copy typed arrays, mix them, create new arrays from existing ones, and so on.



## DataView

[DataView](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView) is a special super-flexible "untyped" view over `ArrayBuffer`. It allows to access the data on any offset in any format.

- For typed arrays, the constructor dictates what the format is. The whole array is supposed to be uniform. The i-th number is `arr[i]`.
- With `DataView` we access the data with methods like `.getUint8(i)` or `.getUint16(i)`. We choose the format at method call time instead of the construction time.

The syntax:

```js
new DataView(buffer, [byteOffset], [byteLength])
```

- **`buffer`** -- the underlying `ArrayBuffer`. Unlike typed arrays, `DataView` doesn't create a buffer on its own. We need to have it ready.
- **`byteOffset`** -- the starting byte position of the view (by default 0).
- **`byteLength`** -- the byte length of the view (by default till the end of `buffer`).

For instance, here we extract numbers in different formats from the same buffer:

```js run
// binary array of 4 bytes, all have the maximal value 255
let buffer = new Uint8Array([255, 255, 255, 255]).buffer;

let dataView = new DataView(buffer);

// get 8-bit number at offset 0
alert( dataView.getUint8(0) ); // 255

// now get 16-bit number at offset 0, it consists of 2 bytes, together interpreted as 65535
alert( dataView.getUint16(0) ); // 65535 (biggest 16-bit unsigned int)

// get 32-bit number at offset 0
alert( dataView.getUint32(0) ); // 4294967295 (biggest 32-bit unsigned int)

dataView.setUint32(0, 0); // set 4-byte number to zero, thus setting all bytes to 0
```

`DataView` is great when we store mixed-format data in the same buffer. For example, when we store a sequence of pairs (16-bit integer, 32-bit float), `DataView` allows to access them easily.

## Summary

`ArrayBuffer` is the core object, a reference to the fixed-length contiguous memory area.

To do almost any operation on `ArrayBuffer`, we need a view.

- It can be a `TypedArray`:
    - `Uint8Array`, `Uint16Array`, `Uint32Array` -- for unsigned integers of 8, 16, and 32 bits.
    - `Uint8ClampedArray` -- for 8-bit integers, "clamps" them on assignment.
    - `Int8Array`, `Int16Array`, `Int32Array` -- for signed integer numbers (can be negative).
    - `Float32Array`, `Float64Array` -- for signed floating-point numbers of 32 and 64 bits.
- Or a `DataView` -- the view that uses methods to specify a format, e.g. `getUint8(offset)`.

In most cases we create and operate directly on typed arrays, leaving `ArrayBuffer` under cover, as a "common denominator". We can access it as `.buffer` and make another view if needed.

There are also two additional terms, that are used in descriptions of methods that operate on binary data:
- `ArrayBufferView` is an umbrella term for all these kinds of views.
- `BufferSource` is an umbrella term for `ArrayBuffer` or `ArrayBufferView`.

We'll see these terms in the next chapters. `BufferSource` is one of the most common terms, as it means "any kind of binary data" -- an `ArrayBuffer` or a view over it.

Here's a cheatsheet:

![](arraybuffer-view-buffersource.svg)
