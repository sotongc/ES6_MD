#字符串的扩展

1. 字符的Unicode表示法
2. codePointAt();
3. String.fromCodePoint();
4. 字符串的遍历器接口
5. at()
6. normalize()
7. includes(),startsWith(),endsWith()
8. repeat()
9. padStart(),padEnd()
10. 模板字符串
11. 实例：模板编译
12. 标签模板
13. String.raw()
14. 模板字符串的限制

ES6加强了对Unicode的支持，并且扩展了字符串对象。
--------------------------------------

##1. 字符的Unicode表示法

JavaScript 允许采用`\uxxxx`形式表示一个字符，其中"xxxx"表示字符的码点。

但是，这种表示法只限于`\u0000`--`uFFFF`之间的字符。超出这个范围的字符，必须用两个双字节的表达形式。

```javascript
"\u0061"//"a"
"\uD842\uDFB7"//"𠮷"
"\u20BB7"//" 7"
```
ES6做出了一些改进，只要将码点放入大括号，就能正确解读该字符。

```javascript
"\u{20BB7}"
// "𠮷"

"\u{41}\u{42}\u{43}"
// "ABC"

let hello = 123;
hell\u{6F} // 123

'\u{1F680}' === '\uD83D\uDE80'
// true
```
最后一个例子表示，大括号表示法与四字节的UTF-16编码是等价的。

##2. codePointAt()

