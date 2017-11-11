# Unicode in Python

tag: python, unicode



## Python 2.x

### unicode 和 str 关系

- 字符串（string）是 unicode 进行编码（encode）得到的。
- unicode 是 string 进行解码（decode）出来的。


```
str ---(decode)---> unicode
unicode ---(encode)---> str
```

**换句话说，unicode 是比较纯粹的，str 则是能用各种方式编码出来的。**

### 构造

```
'hello'  ==> 字符串
u'hello' ===>unicode
```

### 转换

str to unicode:

```py
# 1
s = unicode('中文', 'utf-8')
# 2
('中文').decode('utf-8')
```


unicode to str:

```py
u'中文'.encode('utf-8')
```

### 建议

1. 入口处，全部转成unicode, 池里全部使用unicode处理，出口处，再转成目标编码。
2. 用 `chardet` 进行编码识别。
3. 字符串中混进了其它东西：`.decode('utf8', 'replace')`，`.decode('utf8', 'ignore')`
4. 跨平台用命令行读取编码：`msg.encode(sys.stdin.encoding)`


---



## Python 3.x

Python 2.x 的默认编码是 ASCII，而 3.x 是 UTF-8。

3.x 这样就避免了不少编码的问题。

### bytes

Python 3 中增加了一种 `bytes` 对象（`b‘\xb6\xfe\xbd\xf8\xd6\xc6\xca\xfd\xbe\xdd’`），专门用来表示编码后的（二进制）数据。

### bytes 和 str 的转换

这里和 Python 2.x 有点混淆：

因为这时 str 已经默认是 unicode 了！

```
str ---(encode)---> bytes
bytes ---(decode)---> str
```

具体的作法是：

```py
bytes1 = str1.encode('gbk')
str2 = bytes1.decode('gbk')
```

参考：

- Unicode and other: http://blog.jobbole.com/18269/
- Python 2.x unicode: https://www.v2ex.com/t/287727
- Python 2.x unicode: http://wklken.me/posts/2013/08/31/python-extra-coding-intro.html
- Python 3.x bytes: http://www.crifan.com/summary_python_string_encoding_decoding_difference_and_comparation_python_2_x_str_unicode_vs_python_3_x_bytes_str/
- Python 3.x cool feature: http://www.kissuki.com/blog/2011/11/15/whats-new-in-python-3
