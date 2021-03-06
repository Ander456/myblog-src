---
layout: post
categories: work
title: 'LUA正则表达'
date: 2018-12-25
---

##LUA正则表达

### 规则
* x: (where x is not one of the magic characters ^$()%.[]*+-?) represents the character x itself.
* .: (a dot) represents all characters.
* %a: represents all letters.
* %c: represents all control characters.
* %d: represents all digits.
* %l: represents all lowercase letters.
* %p: represents all punctuation characters.
* %s: represents all space characters.
* %u: represents all uppercase letters.
* %w: represents all alphanumeric characters.
* %x: represents all hexadecimal digits.
* %z: represents the character with representation 0.
* %x: (where x is any non-alphanumeric character) represents the character x

### 常用函数
`string.gsub (s, pattern, repl [, n])`
* s 目标字符串
* pattern 规则（也就是上面的那些）
* repl 替换字符串 最常用的是 作为string ex: "" 或者 "%n" 这里n可以是1-9 %几的意思就是匹配到的第几组也可以理解为第几个括号里匹配到的内容
* n就是要执行几次正则匹配替换

例子：
```
s="先来最基础简单的"
x = string.gsub(s,"基础简单","复杂困难") --将"基础简单"替换成"复杂困难"
--> x = "先来最复杂困难的"

s="多次替换数字123,多次替换数字321"
x = string.gsub(s,"%d+","lalala") --将数字替换成"lalala"
--> x = "多次替换数字lalala,多次替换数字lalala"

s="本末&倒置，开始=逆转"
x = string.gsub(s,"(.+)&(.+)，(.+)=(.+)","%3%2%4%1") --一次匹配下返回括号内四个值，并且排列后替换整个字符串
--> x = "开始倒置逆转本末"

%1-%4相当于临时变量，将临时变量赋予括号内的值并且将整个字符串替换成临时排列后的变量。
%1 = 本末，%2 = 倒置，%3=开始，%4=逆转，而%0则是等于整个字符串：本末&倒置，开始=逆转
***从这里看出来了规则里的括号()就是%n的替换第几个括号就是n几***

s="hello string test world"
x = string.gsub(s,"(%w+)%s*(%w+)","%1",1) --一次匹配返回两个值，将这两个替换成第一个值，只执行一次
--> x = "hello test world"
这里最后的参数是1那么表示这次正则替换我们只执行一次，什么是执行一次呢？
这里"hello string test world"我们定义的查找规则是"(%w+)%s*(%w+)"也就是 "字符空格字符" 
那我们按照规则去s里查找一次得到的结果是找到了并且返回了"hello string"。这，对这就是一次了。如果我们不填1或者写成2那就是要全匹配完或者匹配两次，就会再从s串里接着找，这次我们找到的是"test world"。对的，这就是第二次了。然后每次执行括号里的就是%n，所以上面定义的只执行一次匹配替换，那么我们就按照上面说的执行一次的结果来：
1次的结果是"hello string"然后%1对照"(%w+)%s*(%w+)"第一个括号是 hello %2是 string
然后repl是"%1"也就是替换的字符串是%1也就是hello，那么结果有了，就是把这第一次找到的hello string 替换为hello 然后我们不进行往后的正则查找匹配替换了，到此为止。所以结果就是hello test world。

s="hello string test world"
x = string.gsub(s,"(%w+)%s*(%w+)","%0 %0",1) --匹配成功后将多复制一次，只执行一次
--> x = "hello string hello string test world"
这个同理如上，只不过 repl变成了"%0 %0"其实就是连着输出两次的意思没啥的
这里匹配查找一次的结果是:
%1=hello %2=string %0就是连起来 hello string
然后repl是"%0 %0" 那么就是 hello string hello string
所以结果就是把第一次匹配到的内容 hello string 变成了 hello string hello string
那么此时的s变成了"hello string hello string test world"
```

---

`string.match (s, pattern [, init])`
* s 目标字符串
* pattern 查询匹配规则
* 可选init表示匹配字符串的起始索引默认为1
在字符串s中匹配pattern，如果匹配失败返回nil。否则，当pattern中没有分组时，返回第一个匹配到的子串；**当pattern中有分组时，返回第一个匹配到子串的分组，多个分组就返回多个**。
也就是当有小括号()分组的时候就有多个返回值了

```
s, func = string.match("static bool getBoolean(const char *key, bool defaultValue = false);", "(.+[ *])([^ ]+)[ ]*%(")
print(s, func)
```





