# 正则表达式
[TOC]

## 1 介绍
- 在线工具：https://regexper.com,可以下载到本地运行

## 2 定义
```javascript
var exp = / exp /;
var exp = new RegExp(exp);  //exp中的\需要转义
```

## 3 分类

### 1 特殊字符
- \f:换页
- \n:换行
- \r:回车
- \t:制表符
- \v:水平制表符

### 2 修饰符
- g:全局匹配
- i:忽略大小写
- m:多行匹配

### 3 字符类
- [] :或者。[abc]-->a或者b或者c
- [^]:反向。[abc]-->除了a或者b或者c

### 4 范围类
- [a-z]:在a-z范围内的
- [a-zA-Z]:---> /[a-z]/i;在a-z或A-Z范围内的
- [a-zA-Z0-9]:在a-z或A-Z或0-9范围内的

### 5 预定义类
- . :---> [^/r/n];除了回车换行之外的
- \d:---> [0-9];数字
- \D:
- \s:---> [\t\n\x0B\f\r];空白符
- \S:
- \w:---> [a-zA-Z_0-9];word--数字字母下划线
- \W:

### 6 边界
- ^ :以……开头
- $ :以……结尾
- \b:单词边界
- \B:非单词边界

### 7 量词
- {m}:出现m次
- {n,m}:出现n~m次
- {n,}:至少出现n次
- {0,m}:至多出现m次
- ? :出现0次或1次
- \+:出现1次或多次
- \*:出现任意次

### 8 贪婪模式和非贪婪模式
- 贪婪模式：默认尽可能多次匹配字符，结果为X7
```javascript
var str = '1234567';
str.replace(/\d{3,6}/,'X');
```
- 非贪婪模式：尽可能少的匹配字符，结果为X4567
```javascript
var str = '1234567';
str.replace(/\d{3,6}?/,'X');
```

### 9 分组和或
- /abc{3}/---只会匹配c三次
- /(abc){3}/---匹配abc三次
- /a(b|c)d/---匹配abd或acd

### 10 引用分组
- 忽略分组：(?:b2)

```javascript
var str = 'a1-b2-c3-d4';
console.log('分组：');
var regexp = /(a1)-(b2)-(c3-d4)/;
var result = str.replace(regexp, '$3-$2-$1');
console.log(result);

console.log('忽略分组：');
var regexp1 = /(a1)-(?:b2)-(c3-d4)/;

var result1 = str.replace(regexp1, '$2-$1');
console.log(result1);
```

### 11 前瞻（JS不支持后顾）
` 匹配到exp之后查看断言部分，但不匹配，abc中c为b的前瞻 `
- 正向前瞻：exp(?=assert)
- 负向前瞻：exp(?!assert)

```javascript
//abc中c是b的前瞻断言
var str = 'a1-b2b-c3-d4d-';
console.log('str=' + str);
console.log('正向前瞻：');
var regexp = /[a-z]\d(?=-)/g; //匹配到之后且后面是—的
var result = str.replace(regexp, 'X');

console.log(result);

console.log('负向前瞻：');
var regexp1 = /[a-z]\d(?!-)/g;
var result1 = str.replace(regexp1, 'X');

console.log(result1);
```
