ES6 加强了对 Unicode 的支持，并且扩展了字符串对象。

## 字符的 Unicode 表示法 {#字符的-Unicode-表示法}

JavaScript 允许采用`\uxxxx`形式表示一个字符，其中`xxxx`表示字符的 Unicode 码点。

```
"\u0061"
// "a"
```

但是，这种表示法只限于码点在`\u0000`~`\uFFFF`之间的字符。超出这个范围的字符，必须用两个双字节的形式表示。

```
"\uD842\uDFB7"
// "𠮷"

"\u20BB7"
// " 7"
```

上面代码表示，如果直接在`\u`后面跟上超过`0xFFFF`的数值（比如`\u20BB7`），JavaScript会理解成`\u20BB+7`。由于`\u20BB`是一个不可打印字符，所以只会显示一个空格，后面跟着一个`7`。

ES6 对这一点做出了改进，只要将码点放入大括号，就能正确解读该字符。

```
"\u{20BB7}"
// "𠮷"

"\u{41}\u{42}\u{43}"
// "ABC"

let hello = 123;
hell\u{6F} // 123

'\u{1F680}' === '\uD83D\uDE80'
// true
```

上面代码中，最后一个例子表明，大括号表示法与四字节的 UTF-16 编码是等价的。

有了这种表示法之后，JavaScript 共有6种方法可以表示一个字符。

```
'\z' === 'z'  // true
'\172' === 'z' // true
'\x7A' === 'z' // true
'\u007A' === 'z' // true
'\u{7A}' === 'z' // true
```

## codePointAt\(\)[§](http://es6.ruanyifeng.com/#docs/string#codePointAt)[⇧](http://es6.ruanyifeng.com/#docs/string) {#codePointAt}

JavaScript内部，字符以UTF-16的格式储存，每个字符固定为`2`个字节。对于那些需要`4`个字节储存的字符（Unicode码点大于`0xFFFF`的字符），JavaScript会认为它们是两个字符。

```
var s = "𠮷";

s.length // 2
s.charAt(0) // ''
s.charAt(1) // ''
s.charCodeAt(0) // 55362
s.charCodeAt(1) // 57271
```

上面代码中，汉字“𠮷”（注意，这个字不是”吉祥“的”吉“）的码点是`0x20BB7`，UTF-16编码为`0xD842 0xDFB7`（十进制为`55362 57271`），需要`4`个字节储存。对于这种`4`个字节的字符，JavaScript不能正确处理，字符串长度会误判为`2`，而且`charAt`方法无法读取整个字符，`charCodeAt`方法只能分别返回前两个字节和后两个字节的值。

ES6提供了`codePointAt`方法，能够正确处理4个字节储存的字符，返回一个字符的码点。

```
var s = '𠮷a';

s.codePointAt(0) // 134071
s.codePointAt(1) // 57271

s.codePointAt(2) // 97
```

`codePointAt`方法的参数，是字符在字符串中的位置（从0开始）。上面代码中，JavaScript将“𠮷a”视为三个字符，codePointAt方法在第一个字符上，正确地识别了“𠮷”，返回了它的十进制码点134071（即十六进制的`20BB7`）。在第二个字符（即“𠮷”的后两个字节）和第三个字符“a”上，`codePointAt`方法的结果与`charCodeAt`方法相同。

总之，`codePointAt`方法会正确返回32位的UTF-16字符的码点。对于那些两个字节储存的常规字符，它的返回结果与`charCodeAt`方法相同。

`codePointAt`方法返回的是码点的十进制值，如果想要十六进制的值，可以使用`toString`方法转换一下。

```
var s = '𠮷a';

s.codePointAt(0).toString(16) // "20bb7"
s.codePointAt(2).toString(16) // "61"
```

你可能注意到了，`codePointAt`

方法的参数，仍然是不正确的。比如，上面代码中，字符`a`在字符串`s`的正确位置序号应该是1，但是必须向

`codePointAt`方法传入2。解决这个问题的一个办法是使用`for...of`循环，因为它会正确识别32位的UTF-16字符。

```
var s = '𠮷a';
for (let ch of s) {
  console.log(ch.codePointAt(0).toString(16));
}
// 20bb7
// 61
```

`codePointAt`

方法是测试一个字符由两个字节还是由四个字节组成的最简单方法。

```
function is32Bit(c) {
  return c.codePointAt(0) > 0xFFFF;
}

is32Bit("𠮷") // true
is32Bit("a") // false
```

## String.fromCodePoint\(\) {#String-fromCodePoint}

ES5提供`String.fromCharCode`方法，用于从码点返回对应字符，但是这个方法不能识别32位的UTF-16字符（Unicode编号大于`0xFFFF`）

```
String.fromCharCode(0x20BB7)
// "ஷ"
```



