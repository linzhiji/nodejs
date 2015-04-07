# Buffer 缓冲器

    稳定性: 3 - 稳定

纯 Javascript 语言是 Unicode 友好性的，但是难以处理二进制数据。在处理 TCP 流和文件系统时经常需要操作字节流。Node 提供了一些列机制，用于操作、创建、以及消耗字节流。

在 Buffer 类实例化的中存储了原始数据。 Buffer 类似于一个整数数组，但 Buffer 对应了在 V8 堆（the V8 heap）外的原始存储空间分配。一旦创建了 Buffer 实例，则无法改变其大小。

`Buffer` 类是全局对象，所以它是仅有的不必使用 `require('buffer')` 。

在Buffers和Javascript String对象之间转换需要一个明确的编码方法。下面是string的不同编码。

* `'ascii'` - 7位的 ASCII 数据。这种编码方式非常迅速，它会移除最高位内容。

* `'utf8'` - 多字节编码Unicode字符。大量网页和文档使用这类编码方式。

* `'utf16le'` - 2个或4个字节, 低字节序编码Unicode字符。编码范围 (U+10000 到 U+10FFFF) 。

* `'ucs2'` - `'utf16le'`的子集。

* `'base64'` - Base64 字符编码。

* `'binary'` - 仅使用每个字符的头8位将原始的二进制信息进行编码。在需使用Buffer的情况下，应该尽量避免使用这个已经过时的编码方式。这个编码方式将会在未来版本中弃用。
* `'hex'` - 每个字节都采用 2 进制编码。

在`Buffer`中创建一个数组，需要注意以下规则：

1。 Buffer 是内存拷贝，而不是内存共享。

2。 Buffer 占用内存被解释为一个数组，而不是字节数组。比如， `new Uint32Array(new Buffer([1,2,3,4]))` 创建了4个元素的 `Uint32Array`，它的成员为 `[1,2,3,4]` ,而不是`[0x1020304]` 或 `[0x4030201]`。

注意：Node。js v0。8 只是简单的引用了`array。buffer`里的buffer,而不是之前的克隆(cloning)。

介绍一个高效的方法，`ArrayBuffer#slice()` 拷贝了一份切片，而 `Buffer#slice()` 新建了一份。

## 类: Buffer

Buffer 类是一个全局变量类型，用来直接处理2进制数据的。 它能够使用多种方式构建。

### new Buffer(size)

* `size` Number 类型

分配一个新的 buffer 大小是  `size` 的8位字节。 注意, `size` 必须小于
[kMaxLength](smalloc。html#smalloc_smalloc_kmaxlength)，否则，将会抛出异常 `RangeError`。

### new Buffer(array)

* `array` Array

分配一个新的 buffer 使用一个8位字节 `array` 数组。

### new Buffer(buffer)

* `buffer` {Buffer}

拷贝参数 `buffer` 的数据给 `Buffer` 实例。

### new Buffer(str[, encoding])

* `str` String 类型 - 需要存入 buffer 的 string 字符串。
* `encoding` String 类型 - 编码方式, 非必传。

分配一个新的 buffer ，其中包含着传入的 `str` 字符串。
`encoding` 编码方式默认为 `'utf8'`。

### 类方法: Buffer.isEncoding(encoding)

* `encoding` {String}  用来测试给定的编码字符串

如果给定的编码 `encoding` 是有效的，返回 true，否则返回 false。

### 类方法: Buffer.isBuffer(obj)

* `obj` Object
* Return: Boolean

`obj` 如果是 `Buffer` 返回 true, 否则返回false。

### 类方法: Buffer.byteLength(string[, encoding])

* `string` String 类型
* `encoding` String 类型, 可选, 默认: 'utf8'
* Return: Number 类型

将会返回这个字符串真实byte长度。 encoding 编码默认是： `utf8`. 这个和 `String.prototype.length` 是不一样的，因为那个方法返回这个字符串中字符的数量。

例如:

    str = '\u00bd + \u00bc = \u00be';

    console.log(str + ": " + str。length + " characters, " +
      Buffer。byteLength(str, 'utf8') + " bytes");

    // ½ + ¼ = ¾: 9 characters, 12 bytes

### 类方法: Buffer.concat(list[, totalLength])

* `list` {Array} 用来连接的数组
* `totalLength` {Number 类型} 数组里所有对象的总buffer大小

返回一个buffer对象，它是由传入buffer数组中所有buffer对象拼接在一起的。

如果传入的数组没有内容，或者 totalLength 是0，那将返回一个长度为0的buffer。

如果数组长度为0，返回数组第一个成员。

如果数组长度大于0，将会创建一个新的Buffer实例。

如果没有提供 totalLength 参数，会从buffer数组中计算出来，这样会增加一个额外的循环来计算它，所以提供一个准确的 totalLength 参数将会更快。

### 类方法: Buffer.compare(buf1, buf2)

* `buf1` {Buffer}
* `buf2` {Buffer}

和 [`buf1。compare(buf2)`](#buffer_buf_compare_otherbuffer)一样。 用来对数组排序:

    var arr = [Buffer('1234'), Buffer('0123')];
    arr.sort(Buffer.compare);


### buf.length

* Number 类型

这个 buffer 的 bytes 数。注意这未必是这 buffer 里面内容的大小。`length` 的依据是 buffer 对象所分配的内存数值，它不会随着这个 buffer 对象内容的改变而改变。

    buf = new Buffer(1234);

    console.log(buf.length);
    buf.write("some string", 0, "ascii");
    console.log(buf.length);

    // 1234
    // 1234

`length`的不能改变，如果改变`length`将会导致不可以预期的结果。如果想要改变buffer的长度，需要使用`buf.slice`来创建新的 buffer。

    buf = new Buffer(10);
    buf.write("abcdefghj", 0, "ascii");
    console.log(buf.length); // 10
    buf = buf.slice(0,5);
    console.log(buf.length); // 5

### buf.write(string[, offset][, length][, encoding])

* `string` String 类型 - 写入到 buffer 里
* `offset` Number 类型, 可选参数， 默认值: 0
* `length` Number 类型, 可选参数， 默认值: `buffer.length - offset`
* `encoding` String 类型， 可选参数， 默认值: 'utf8'

根据参数 offset 偏移量和指定的 encoding 编码方式，将参数 string 数据写入buffer。 offset偏移量 默认是 0, encoding编码方式默认是 `utf8`。 length长度是将要写入的字符串的bytes大小。 返回number类型，表示多少8位字节流被写入了。如果buffer 没有足够的空间来放入整个string，它将只会写入部分的字符串。 length 默认是 buffer.length - offset。 这个方法不会出现写入部分字符。

    buf = new Buffer(256);
    len = buf.write('\u00bd + \u00bc = \u00be', 0);
    console.log(len + " bytes: " + buf.toString('utf8', 0, len));

### buf.writeUIntLE(value, offset, byteLength[, noAssert])
### buf.writeUIntBE(value, offset, byteLength[, noAssert])
### buf.writeIntLE(value, offset, byteLength[, noAssert])
### buf.writeIntBE(value, offset, byteLength[, noAssert])

* `value` {Number 类型} Bytes to be written to buffer
* `offset` {Number 类型} `0 <= offset <= buf.length`
* `byteLength` {Number 类型} `0 < byteLength <= 6`
* `noAssert` {Boolean} 默认值: false
* Return: {Number 类型}

将`value` 写入到 buffer 里， 它的时由`offset` 和 `byteLength` 决定的，支持48位计算，例如:

    var b = new Buffer(6);
    b.writeUIntBE(0x1234567890ab, 0, 6);
    // <Buffer 12 34 56 78 90 ab>

将 `noAssert` 设置为 `true` ，不再验证 `value` 和 `offset` 的有效性。 默认是 `false`。

### buf.readUIntLE(offset, byteLength[, noAssert])
### buf.readUIntBE(offset, byteLength[, noAssert])
### buf.readIntLE(offset, byteLength[, noAssert])
### buf.readIntBE(offset, byteLength[, noAssert])

* `offset` {Number 类型} `0 <= offset <= buf.length`
* `byteLength` {Number 类型} `0 < byteLength <= 6`
* `noAssert` {Boolean} 默认值: false
* Return: {Number 类型}

支持48为位以下的数字读取。 例如:

    var b = new Buffer(6);
    b.writeUint16LE(0x90ab, 0);
    b.writeUInt32LE(0x12345678, 2);
    b.readUIntLE(0, 6).toString(16);  // 指定为 6 bytes (48 bits)
    // 输出: '1234567890ab'

将 `noAssert` 设置为 true 后， `offset` 不再验证是否超过 buffer 的长度，默认为 `false`。

### buf.toString([encoding][, start][, end])

* `encoding` String 类型， 可选参数， 默认值: 'utf8'
* `start` Number 类型， 可选参数， 默认值: 0
* `end` Number 类型， 可选参数， 默认值: `buffer.length`

根据 encoding参数（默认是 'utf8'）返回一个解码的 string 类型。还会根据传入的参数 start (默认是0) 和 end (默认是 buffer.length)作为取值范围。

    buf = new Buffer(26);
    for (var i = 0 ; i < 26 ; i++) {
      buf[i] = i + 97; // 97 is ASCII a
    }
    buf.toString('ascii'); // 输出: abcdefghijklmnopqrstuvwxyz
    buf.toString('ascii',0,5); // 输出: abcde
    buf.toString('utf8',0,5); // 输出: abcde
    buf.toString(undefined,0,5); // encoding defaults to 'utf8', 输出 abcde

查看上面 `buffer.write()` 例子。


### buf.toJSON()

返回一个 JSON表示的Buffer实例。`JSON.stringify`将会默认调用来字符串序列化这个Buffer实例。

例如:

    var buf = new Buffer('test');
    var json = JSON。stringify(buf);

    console.log(json);
    // '{"type":"Buffer","data":[116,101,115,116]}'

    var copy = JSON。parse(json, function(key, value) {
        return value && value。type === 'Buffer'
          ? new Buffer(value。data)
          : value;
      });

    console.log(copy);
    // <Buffer 74 65 73 74>

### buf[index]

<!--type=property-->
<!--name=[index]-->

获取或者设置在指定index索引位置的8位字节。这个值是指单个字节，所以这个值必须在合法的范围，16进制的0x00 到0xFF，或者0 到255。


例如: 拷贝一个 ASCII 编码的 string 字符串到一个 buffer, 一次一个 byte 进行拷贝:

    str = "node。js";
    buf = new Buffer(str.length);

    for (var i = 0; i < str.length ; i++) {
      buf[i] = str.charCodeAt(i);
    }

    console.log(buf);

    // node。js

### buf.equals(otherBuffer)

* `otherBuffer` {Buffer}

如果 `this` 和 `otherBuffer` 拥有相同的bytes，返回 true。

### buf.compare(otherBuffer)

* `otherBuffer` {Buffer}

返回一个数字，表示 `this` 是否`otherBuffer`在它之前，之后或相同。

### buf.copy(targetBuffer[, targetStart][, sourceStart][, sourceEnd])

* `targetBuffer` Buffer object - Buffer to copy into
* `targetStart` Number 类型， 可选参数， 默认值: 0
* `sourceStart` Number 类型， 可选参数， 默认值: 0
* `sourceEnd` Number 类型， 可选参数， 默认值: `buffer.length`

进行buffer的拷贝，源和目标可以是重叠的。 `targetStart` 目标开始偏移 和 `sourceStart` 源开始偏移 默认都是 0. `sourceEnd` 源结束位置偏移默认是源的长度 `buffer.length` 。

例如:创建2个Buffer，然后把将buf1的16位到19位 拷贝到 buf2中，并且从buf2的第8位开始拷贝。

    buf1 = new Buffer(26);
    buf2 = new Buffer(26);

    for (var i = 0 ; i < 26 ; i++) {
      buf1[i] = i + 97; // 97 is ASCII a
      buf2[i] = 33; // ASCII !
    }

    buf1。copy(buf2, 8, 16, 20);
    console.log(buf2。toString('ascii', 0, 25));

    // !!!!!!!!qrst!!!!!!!!!!!!!

例如: 在同一个buffer中，从一个区域拷贝到另一个区域

    buf = new Buffer(26);

    for (var i = 0 ; i < 26 ; i++) {
      buf[i] = i + 97; // 97 is ASCII a
    }

    buf.copy(buf, 0, 4, 10);
    console.log(buf.toString());

    // efghijghijklmnopqrstuvwxyz


### buf.slice([start][, end])

* `start` Number 类型， 可选参数， 默认值: 0
* `end` Number 类型， 可选参数， 默认值: `buffer.length`

返回一个新的buffer，这个buffer将会和老的buffer引用相同的内存地址，只是根据 `start `(默认是 `0` ) 和 `end` (默认是 `buffer.length` ) 偏移和裁剪了索引。 负的索引是从buffer尾部开始计算的。

**修改这个新的buffer实例slice切片，也会改变原来的buffer!**

例如: 创建一个ASCII 字母的 Buffer，对它slice切片，然后修改源Buffer上的一个byte。

    var buf1 = new Buffer(26);

    for (var i = 0 ; i < 26 ; i++) {
      buf1[i] = i + 97; // 97 is ASCII a
    }

    var buf2 = buf1。slice(0, 3);
    console.log(buf2。toString('ascii', 0, buf2。length));
    buf1[0] = 33;
    console.log(buf2。toString('ascii', 0, buf2。length));

    // abc
    // !bc

### buf.readUInt8(offset[, noAssert])

* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false
* Return: Number 类型

从这个buffer对象里，根据指定的偏移量，读取一个 unsigned 8 bit integer整形。

设置参数 `noAssert` 为true表示忽略验证 `offset` 偏移量参数。 这意味着 `offset` 可能会超出buffer的末尾。默认是 `false`。

例如:

    var buf = new Buffer(4);

    buf[0] = 0x3;
    buf[1] = 0x4;
    buf[2] = 0x23;
    buf[3] = 0x42;

    for (ii = 0; ii < buf.length; ii++) {
      console.log(buf.readUInt8(ii));
    }

    // 0x3
    // 0x4
    // 0x23
    // 0x42

### buf.readUInt16LE(offset[, noAssert])
### buf.readUInt16BE(offset[, noAssert])

* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false
* Return: Number 类型

从这个buffer对象里，根据指定的偏移量，使用特殊的 endian字节序格式读取一个 unsigned 16 bit integer。

设置参数 `noAssert` 为true表示忽略验证 `offset` 偏移量参数。 这意味着 `offset` 可能会超出buffer的末尾。默认是 `false`。

例如:

    var buf = new Buffer(4);

    buf[0] = 0x3;
    buf[1] = 0x4;
    buf[2] = 0x23;
    buf[3] = 0x42;

    console.log(buf.readUInt16BE(0));
    console.log(buf.readUInt16LE(0));
    console.log(buf.readUInt16BE(1));
    console.log(buf.readUInt16LE(1));
    console.log(buf.readUInt16BE(2));
    console.log(buf.readUInt16LE(2));

    // 0x0304
    // 0x0403
    // 0x0423
    // 0x2304
    // 0x2342
    // 0x4223

### buf.readUInt32LE(offset[, noAssert])
### buf.readUInt32BE(offset[, noAssert])

* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false
* Return: Number 类型

从这个buffer对象里，根据指定的偏移量，使用指定的 endian字节序格式读取一个 unsigned 32 bit integer。

设置参数 `noAssert`为true表示忽略验证offset偏移量参数。 这意味着 `offset` 可能会超出buffer的末尾。默认是 `false`。

例如:

    var buf = new Buffer(4);

    buf[0] = 0x3;
    buf[1] = 0x4;
    buf[2] = 0x23;
    buf[3] = 0x42;

    console.log(buf.readUInt32BE(0));
    console.log(buf.readUInt32LE(0));

    // 0x03042342
    // 0x42230403

### buf.readInt8(offset[, noAssert])

* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false
* Return: Number 类型

从这个buffer对象里，根据指定的偏移量，读取一个 signed 8 bit integer。

设置参数 `noAssert` 为true表示忽略验证offset偏移量参数。 这意味着 offset 可能会超出buffer的末尾。默认是 `false`。

和 `buffer.readUInt8` 一样的返回，除非buffer中包含了有作为2的补码的有符号值。

### buf.readInt16LE(offset[, noAssert])
### buf.readInt16BE(offset[, noAssert])

* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false
* Return: Number 类型

从这个 buffer 对象里，根据指定的偏移量，使用特殊的 endian 格式读取一个 signed 16 bit integer。

设置参数 `noAssert`为 true 表示忽略验证offset偏移量参数。 这意味着 offset 可能会超出buffer 的末尾。默认是 `false`。

和 `buffer.readUInt16` 一样返回，除非buffer中包含了有作为2的补码的有符号值。

### buf.readInt32LE(offset[, noAssert])
### buf.readInt32BE(offset[, noAssert])

* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false
* Return: Number 类型

从这个buffer对象里，根据指定的偏移量，使用指定的 endian字节序格式读取一个 signed 32 bit integer。

设置参数 `noAssert` 为true表示忽略验证 `offset` 偏移量参数。 这意味着 `offset` 可能会超出buffer的末尾。默认是 `false`。

和 `buffer.readUInt32` 一样返回，除非buffer中包含了有作为2的补码的有符号值。

### buf.readFloatLE(offset[, noAssert])
### buf.readFloatBE(offset[, noAssert])

* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false
* Return: Number 类型

从这个buffer对象里，根据指定的偏移量，使用指定的 endian字节序格式读取一个 32 bit float。

设置参数 `noAssert` 为true表示忽略验证 `offset` 偏移量参数。 这意味着 `offset` 可能会超出buffer的末尾。默认是 `false`。

例如:

    var buf = new Buffer(4);

    buf[0] = 0x00;
    buf[1] = 0x00;
    buf[2] = 0x80;
    buf[3] = 0x3f;

    console.log(buf.readFloatLE(0));

    // 0x01

### buf.readDoubleLE(offset[, noAssert])
### buf.readDoubleBE(offset[, noAssert])

* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false
* Return: Number 类型

从这个buffer对象里，根据指定的偏移量，使用指定的 endian字节序格式读取一个 64 bit double。

设置参数 `noAssert` 为 true 表示忽略验证 `offset` 偏移量参数。 这意味着 `offset` 可能会超出buffer的末尾。默认是 `false`。

例如:

    var buf = new Buffer(8);

    buf[0] = 0x55;
    buf[1] = 0x55;
    buf[2] = 0x55;
    buf[3] = 0x55;
    buf[4] = 0x55;
    buf[5] = 0x55;
    buf[6] = 0xd5;
    buf[7] = 0x3f;

    console.log(buf.readDoubleLE(0));

    // 0。3333333333333333

### buf.writeUInt8(value, offset[, noAssert])

* `value` Number 类型
* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false

根据传入的 `offset` 偏移量将 `value` 写入buffer。注意：`value` 必须是一个合法的unsigned 8 bit integer。

设置参数 `noAssert` 为 true 表示忽略验证 `value` 和 `offset` 参数。 这意味着 `value` 可能过大，或者 `offset` 可能会超出buffer的末尾造成value被丢弃。 这个参数除了你非常有把握，否则不应该使用它。默认是 `false`。

例如:

    var buf = new Buffer(4);
    buf.writeUInt8(0x3, 0);
    buf.writeUInt8(0x4, 1);
    buf.writeUInt8(0x23, 2);
    buf.writeUInt8(0x42, 3);

    console.log(buf);

    // <Buffer 03 04 23 42>

### buf.writeUInt16LE(value, offset[, noAssert])
### buf.writeUInt16BE(value, offset[, noAssert])

* `value` Number 类型
* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false

根据传入的 `offset` 偏移量和指定的 endian 格式将 `value` 写入 buffer。注意：`value` 必须是一个合法的 unsigned 16 bit integer。

设置参数 `noAssert` 为 true 表示忽略验证 `value` 和 `offset` 参数。 这意味着 `value` 可能过大，或者 `offset` 可能会超出buffer的末尾造成 value 被丢弃。 这个参数除了你非常有把握，否则不应该使用它。默认是 `false`。

例如:

    var buf = new Buffer(4);
    buf.writeUInt16BE(0xdead, 0);
    buf.writeUInt16BE(0xbeef, 2);

    console.log(buf);

    buf.writeUInt16LE(0xdead, 0);
    buf.writeUInt16LE(0xbeef, 2);

    console.log(buf);

    // <Buffer de ad be ef>
    // <Buffer ad de ef be>

### buf.writeUInt32LE(value, offset[, noAssert])
### buf.writeUInt32BE(value, offset[, noAssert])

* `value` Number 类型
* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false

根据传入的 `offset` 偏移量和指定的 endian 格式将 `value` 写入buffer。注意：`value` 必须是一个合法的 unsigned 32 bit integer。

设置参数 `noAssert` 为 true 表示忽略验证 `value` 和 `offset` 参数。 这意味着`value` 可能过大，或者offset可能会超出buffer的末尾造成 `value` 被丢弃。 这个参数除了你非常有把握，否则不应该使用它。默认是 `false`。

例如:

    var buf = new Buffer(4);
    buf.writeUInt32BE(0xfeedface, 0);

    console.log(buf);

    buf.writeUInt32LE(0xfeedface, 0);

    console.log(buf);

    // <Buffer fe ed fa ce>
    // <Buffer ce fa ed fe>

### buf.writeInt8(value, offset[, noAssert])

* `value` Number 类型
* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false

根据传入的 `offset` 偏移量将 `value` 写入 buffer 。注意：`value` 必须是一个合法的 signed 8 bit integer。

设置参数 `noAssert` 为 true 表示忽略验证 `value` 和 offset 参数。 这意味着 `value` 可能过大，或者 offset 可能会超出 buffer 的末尾造成 `value` 被丢弃。 这个参数除了你非常有把握，否则不应该使用它。默认是 `false`。

和 `buffer.writeUInt8` 一样工作，除非是把有2的补码的 `signed integer` 有符号整形写入buffer。

### buf.writeInt16LE(value, offset[, noAssert])
### buf.writeInt16BE(value, offset[, noAssert])

* `value` Number 类型
* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false

根据传入的 `offset` 偏移量和指定的 endian 格式将 `value` 写入 buffer。注意：`value` 必须是一个合法的 signed 16 bit integer。

设置参数 `noAssert` 为 true 表示忽略验证 `value` 和 `offset` 参数。 这意味着 `value` 可能过大，或者 offset 可能会超出 buffer 的末尾造成 `value` 被丢弃。 这个参数除了你非常有把握，否则不应该使用它。默认是 `false` 。

和 `buffer.writeUInt16*` 一样工作，除非是把有2的补码的 signed integer 有符号整形写入`buffer`。

### buf.writeInt32LE(value, offset[, noAssert])
### buf.writeInt32BE(value, offset[, noAssert])

* `value` Number 类型
* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false

根据传入的 `offset` 偏移量和指定的 endian 格式将 `value` 写入 buffer。注意：`value` 必须是一个合法的 signed 32 bit integer。

设置参数 `noAssert` 为 true 表示忽略验证 `value` 和 `offset` 参数。 这意味着 `value` 可能过大，或者 `offset` 可能会超出 buffer 的末尾造成 `value` 被丢弃。 这个参数除了你非常有把握，否则不应该使用它。默认是 `false`。

和 `buffer.writeUInt32*` 一样工作，除非是把有2的补码的 signed integer 有符号整形写入buffer。

### buf.writeFloatLE(value, offset[, noAssert])
### buf.writeFloatBE(value, offset[, noAssert])

* `value` Number 类型
* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false

根据传入的 `offset` 偏移量和指定的 endian 格式将 `value` 写入 buffer 。注意：当 `value` 不是一个 32 bit float 类型的值时，结果将是不确定的。

设置参数 `noAssert` 为 true 表示忽略验证 `value` 和 `offset` 参数。 这意味着 value可能过大，或者 offset 可能会超出 buffer 的末尾造成 `value` 被丢弃。 这个参数除了你非常有把握，否则不应该使用它。默认是 `false`。

例如:

    var buf = new Buffer(4);
    buf.writeFloatBE(0xcafebabe, 0);

    console.log(buf);

    buf.writeFloatLE(0xcafebabe, 0);

    console.log(buf);

    // <Buffer 4f 4a fe bb>
    // <Buffer bb fe 4a 4f>

### buf.writeDoubleLE(value, offset[, noAssert])
### buf.writeDoubleBE(value, offset[, noAssert])

* `value` Number 类型
* `offset` Number 类型
* `noAssert` Boolean, 可选参数， 默认值: false

根据传入的 `offset` 偏移量和指定的 endian 格式将 `value` 写入 buffer。注意：`value` 必须是一个有效的 64 bit double 类型的值。

设置参数 `noAssert` 为 true 表示忽略验证 `value` 和 `offset` 参数。 这意味着 `value` 可能过大，或者 offset 可能会超出 buffer 的末尾造成`value`被丢弃。 这个参数除了你非常有把握，否则不应该使用它。默认是 `false`。

例如:

    var buf = new Buffer(8);
    buf.writeDoubleBE(0xdeadbeefcafebabe, 0);

    console.log(buf);

    buf.writeDoubleLE(0xdeadbeefcafebabe, 0);

    console.log(buf);

    // <Buffer 43 eb d5 b7 dd f9 5f d7>
    // <Buffer d7 5f f9 dd b7 d5 eb 43>

### buf.fill(value[, offset][, end])

* `value`
* `offset` Number 类型， Optional
* `end` Number 类型， Optional

使用指定的 `value` 来填充这个buffer。如果 `offset` (默认是 0) 并且 `end` (默认是 `buffer.length`) 没有明确给出，就会填充整个buffer。

    var b = new Buffer(50);
    b.fill("h");

## buffer.iNSPECT_MAX_BYTES

* Number 类型， 默认值: 50


设置当调用`buffer.inspect()`方法后，多少bytes将会返回。这个值可以被用户模块重写。

注意这个属性是`require('buffer')`模块返回的。这个属性不是在全局变量Buffer中，也不在buffer的实例里。

## 类: SlowBuffer

返回一个不被池管理的 `Buffer`。

大量独立分配的 Buffer 容易带来的垃圾，为了避免这个情况，小于 4KB 的空间都是切割自一个较大的独立对象。这种策略既提高了性能也改善了内存使用率。V8 不需要跟踪和清理很多 `Persistent` 对象。

当开发者需要将池中一小块数据保留一段时间，比较好的办法是用 SlowBuffer 创建一个不被池管理的 Buffer 实例，并将相应数据拷贝出来。

    // need to keep around a few small chunks of memory
    var store = [];

    socket。on('readable', function() {
      var data = socket。read();
      // allocate for retained data
      var sb = new SlowBuffer(10);
      // copy the data into the new allocation
      data。copy(sb, 0, 0, 10);
      store。push(sb);
    });

请谨慎使用，仅作为开发者经常观察到他们的应用中过度的内存保留时的最后手段。
