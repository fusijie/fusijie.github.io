---
layout: post
title: 30分钟正则表达式快速入门
date: 2016-04-28 21:54:28
comments: true
categories: Others
tags: [正则表达式]
---

正则表达式在查找，解析，替换文本是非常管用的，这些常见的文本包含但不限于代码，日志，表格，甚至于文稿。但是很多人一看到正则表达式，反应通常是

<div align="center"><img src="/images/posts/2016-04-28-learn-regex-in-30-minutes/ylmb.jpg" alt="" border="0" /><br></br></div>

其实并没有这么难，本文就是带你30分钟快速入门正则表达式。

本文改编自 [http://regexone.com/](http://regexone.com/)。

### ABCs

首先要明白的第一件事就是**本质上一切都是字符**，正则表达式就是一串用于匹配字符串的字符串。

一般情况下我们使用 ASCII 字符，也就是英文字符，数字，标点符号和一些特殊符号（%#$@!），当然 Unicode 字符也可以用作正则表达式

<!-- more -->。

先来试一试最简单的字符串直接匹配。

练习：匹配字符串

```
match      abcdefg
match      abcde
match      abc
```
答案：（正则表达式是没有唯一答案的，这和你需要的匹配程度有关）

```
regex      abc
```

### 123s

字符包括英文字母，当然也包括数字。

`\d` 通常用来表示单个数字（0~9），比如 `\d` 可以是 0~9，`\d\d` 可以是 00~99。

练习：匹配数字

```
match      abc123xyz
match      define "123"
match      var g = 123;
```
答案：(当然你也可以写`\d\d\d`，同样可以匹配123)

```
regex      123
```

### 点号

正则表达式中点号（.）是做通配符使用，也就是说点号可以表示任意单个字符（字母，数字，空格，任何符号都是）。

但是如果要准确适配点号本身，那如何表示呢？`\.`

练习：使用通配符匹配

```
match      cat.
match      896.
match      ?=+.
skip       abc1
```
答案：(关键在于最后一个字符点号要能够准确适配)

```
regex      ...\.
```

### 匹配指定字符

点号太给力了，但是给力不一定是万能的，比如我们要匹配一个电话号码，“abcd-efghijkl” 可以是一个有效是电话号码吗？

所以我们需要一种能匹配指定字符的方法，这就是`[]`。比如 `[abc]` 只匹配 a 或 b 或 c。要注意 `[]` 只匹配单个字符。

练习：匹配字符串

```
match      can
match      man
match      fan
skip       dan
skip       ran
skip       pan
```
答案：(学了下一节之后你会发现还可以用[^drp]an)

```
regex      [cmf]an
```

### 匹配指定排除的字符

有时候我们并不知道要匹配哪个字符，但是我们指定不匹配哪个字符。

这时候 `^` 就有用了。比如 `[^abc]` 匹配除 a 和 b 和 c 以外的所有字符，注意也是单个。

练习：匹配指定排除的字符

```
match      hog
match      dog
skip       bog
```
答案：（也可以使用上一节的 `[hd]og`）

```
regex      [^b]og
```

### 范围字符

如果需要指定的字符太多了，比如从 a 到 g，难道我们就要写 `[abcdefg]` 么？当然有更方便的写法 `[a-g]`。

举个栗子，`[a-z]` 表示匹配所有小写字母（注意大小写是区分的），`[0-9]` 表示匹配所有数字，`[^n-p]` 表示匹配除 n 到 p 以外的单个字符，而 `\w` 其实就是 `[A-Za-z0-9_]`。

练习：匹配指定范围字符

```
match      Ana
match      Bob
match      Cpc
skip       aax
skip       bby
skip       ccz
```
答案：

```
regex      [A-Z][n-p][a-c]
```

### 匹配重复次数

前面我们学习的是匹配字符，假如要匹配一个3位数的数字，那就是 `\d\d\d`，但是如果是10位呢？如果是100位呢？

这里就需要重复次数的语法了，`{}`。

`a{3}` 表示a重复3次，`a{2,6}`表示a重复2次到6次之间都匹配。

结合之前学习的，`[a-z]{5}`表示某个小写字母重复5次，比如 bbbbb，而 `\d{3,8}` 表示某个数字重复3到8次，比如 55555。

练习：匹配重复字符

```
match      wazzzzup
match      wazzzup
skip       wazup
```

答案：

```
regex      waz{3,4}up
```

### 另一种匹配重复次数

除了使用 `{}` 来匹配重复次数，还可以使用下面两种符号来匹配重复次数。

`*`，表示0次到多次重复，比如 `.*` 匹配任意字符串

`+`，表示1次到多次重复，比如 `a+` 匹配超过一个包含 a 的字符串

练习：匹配重复字符

```
match      aaaabcc
match      aabbbbc
match      aacc
skip       a
```

答案：（也可以是上一节的 `a{2,4}b{0,4}c{1,2}`）

```
regex      a+b*c+
```

### 可选字符

可选字符是指某个字符可以是一个也可以没有，用 `?` 表示。

举个栗子，`ab?c` 可以匹配 abc 或者 ac。

需要注意的是，如果要准确匹配问号，则需要使用 `\?`。

练习：匹配可选字符

```
match      1 file found?
match      2 files found?
match      24 files found?
skip       no files found.
```
答案：

```
regex      \d+ files? found\?
```

### 匹配空白符

空白符包含空格键（␣），制表符（\t），换行符（\n）和回车符（\r），使用 `\s` 表示。

练习：匹配空白字符

```
match      1.   abc
match      2.    abc
match      3.       abc
skip       4.abc
```
答案：

```
regex      \d\.\s+abc
```

### 开始和结束

到目前为止，我们写的正则表达式都是匹配片段文本，如果需要匹配完整的文本呢？

所谓完整，就是有开头有结尾。正则表达式中使用 `^` 和 `$` 表示。

练习：匹配行

```
match      Mission:successful
skip       Last Mission: unsuccessful
skip       Next Mission: successful upon capture of target
```
答案：

```
regex      ^Mission: successful$
```

### 匹配组

正则表达式不仅可以匹配文本，还允许我们取出部分信息做进一步的处理。

如何取出我们想要的信息呢？使用`()`。

比如，有个图片的 url，可能是这样的 `https://noexist.com/IMG123.png`，你希望取出文件名 `IMG_123`。对应的正则表达式就是 `^(IMG\d+)\.png`。

练习：匹配组

```
capture       file_record_transcript.pdf      file_record_transcript
capture       file_07241999.pdf               file_07241999
skip          testfile_fake.pdf.tmp		
```
答案：

```
regex        ^(file.+)\.pdf$
```

### 嵌套组

有的时候你想要获得多层信息，还是以图片的 url（`https://noexist.com/IMG123.png`）为例，你既想拿到完整的图片名（IMG123），也想得到图片的序号（123）。这时候嵌套组就可以派上用场了。

嵌套组使用多个嵌套的 `()` 来区分组。比如 `^(IMG(\d+))\.png$` 就可以实现上述目的。

练习：匹配嵌套组

```
capture       Jan 1987      Jan 1987       1987
capture       May 1969      May 1969       1969
capture       Aug 2011      Aug 2011       2011
```
答案：

```
regex        (\w+ (\d+))
```

### 匹配重复组

之前我们了解到 `*`，`+`，`{m,n}`，`?` 可以用在单个字符上，同样的，它也可以用在组上。

比如我们知道某个电话号码可能包含或者不包含一个区号，那么我们就可以写一个 `(\d{4})?` 来处理区号。

练习：匹配嵌套组

```
capture      1280x720      1280      720
capture      1920x1600     1920      1600
capture      1024x768      1024      768
```
答案：

```
regex        (\d+)x(\d+)
```

### 可选项匹配

之前我们学习到了精确匹配，但是有时候我们在几个选项之间犹豫，这时候就可以使用 `|` 来处理可选项的匹配。

比如 `Buy more (milk|bread|juice)` 可以匹配 Buy more milk，Buy more bread 和 Buy more juice。

练习：匹配可选文本

```
match    I love cats
match    I love dogs
skip     I love logs
skip     I love cogs

```
答案：

```
regex    I love (cats|dogs)
```

### 其他特殊字符

之前我们已经学习了 `\d（数字）`，`\s（空格）`，`\w（字母数字下划线）`。正则表达式也提供给我们一套相反的集合，用大写的 `\D（非数字）`，`\S（非空格）`，`\W（非字母数字下划线，比如标点符号）` 来表示。

另外 `\b` 用于匹配单词和非单词字符之间的边界。比如我们要获取一个单词，就可以这么写 `\w+\b`。

还有一些系统是支持用 `\0`，`\1`，`\2` 来表示组。

练习：匹配特殊字符

```
match      The quick brown fox jumped over the lazy dog.
match      There were 614 instances of students getting 90.0% or above.
match      The FCC had to censor the network for saying &$#*@!.
```

答案：自由发挥~