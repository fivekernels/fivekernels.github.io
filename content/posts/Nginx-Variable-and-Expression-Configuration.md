---
title: "Nginx 语法 变量和表达式"
date: 2022-01-25T21:17:27+08:00
tags: [ "Nginx", "SSL", "Clint Certificate"]
categories: [ "Nginx", "SSL" ]
draft: true
---

## Nginx 变量

### 赋值

- 变量名以 $ 开头：$hello
- 变量仅有一种类型：字符串
- 每行语句以分号结尾
- 使用 set 指令对变量赋值：

```nginx
set $hello "hello";
```

### 取值

- 使用变量时将变量写在双引号内，将会自动将其替换为实际内容
- 写在单引号内的变量不会被替换

```nginx
print "this is $hello world.";
```

- 变量后紧跟构成变量名的字符（字母、数字、下划线等），使用大括号消除歧义

```nginx
print "this is ${hello}world.";
```

详细信息：

<https://blog.csdn.net/u014296316/article/details/80973530>

<http://blog.sina.com.cn/s/blog_6d579ff40100wi7p.html>

### 拼接

```nginx
set $allUrl "${host}${request_uri}";
```

### 内置变量

#### url 相关

- $http_host
- $server_name
- $request_uri
- $uri<>
- $arg_&lt;*PARAMETER*>: 客户端GET请求中 &lt;*PARAMETER*>字段的值

#### 客户端证书

参考：[CSDN涂宗勋：nginx获取ca证书信息并传递到java后端使用](https://blog.csdn.net/tuzongxun/article/details/91477954)

&emsp;&emsp;nginx内部解析证书后，会把相关信息放到内置的变量中

- $ssl_client_cert：证书内容
- $ssl_client_serial：证书序列号
- $ssl_client_s_dn：证书subject

&emsp;&emsp;若要将客户端信息传递到后端，可以在 Nginx 配置文件中的 location 内加入 header 配置：

```nginx
location / {
    ......
    proxy_set_header X-SSL-Client-Cert $ssl_client_cert;
    proxy_set_header X-SSL-serial $ssl_client_serial;
    proxy_set_header cert-subject  $ssl_client_s_dn;
}
```

其中，X-SSL-Client-Cert、X-SSL-serial、cert-subject为自定义变量名，供后端代码（java等）获取header值使用。

```java
public class CaController {
    @GetMapping("/test")
    public String caTest(@RequestHeader("X-SSL-serial") String serial, @RequestHeader("X-SSL-Client-Cert") String cert, @RequestHeader("cert-subject") String subject) {
        return serial + " \r" + cert + " \r" + subject;
    }
}
```

## 表达式

### 正则表达式 regex

| 字符 | 描述 |
| :----: | :---- |
| ^ | 匹配输入字符串的起始位置 |
| $ | 匹配输入字符串的结束位置 |
| \d | 匹配数字 |
| \\ | 将后面接着的字符标记为一个特殊字符或一个原义字符或一个向后引用。如"\\n"匹配一个换行符，而"\\$"则匹配"$" |
| \[c\] | 匹配单个字符c |
| \[a\-z\] |匹配 a\-z 任意一个小写字母 |
| \. | 匹配除换行符"\\n"之外的任何单个字符；若要匹配包括"\\n"在内的任意字符，使用如"\[\.\\n\]"等模式。|
| \{n\} | 重复n次 |
| \{n,\} | 重复n次或更多次 |
| ? | 匹配前面的字符0次或1次。如"do\(es\)?"能匹配"do"或者"does"，"?"等价于"\{0,1\}" |
| \* | 匹配前面的字符0次或多次。如"ol\*"能匹配"o"及"ol"、"oll"|
| \+ | 匹配前面的字符1次或多次。如"ol\+"能匹配"ol"及"oll"、"oll"，但不能匹配"o"|
| (pattern) | 匹配括号内pattern并可以在后面获取对应的匹配，常用$0\.\.\.$9属性获取小括号中的匹配内容，要匹配圆括号字符需要Content |

### 比较

**数值比较:**

&emsp;&emsp;比较两个变量/变量与字符串是否相等

|  符号  |  说明   |
| :----: | :----: |
| =      | 相等   |
| \!=    | 不相等  |

**正则比较:**

&emsp;&emsp;变量与正则表达式的模式是否匹配

|  符号  |  说明   |
| :----: | :----: |
| ~ | 匹配，区分大小写 |
| ~\* | 匹配，不区分大小写 |
| \!~ | 不匹配，区分大小写 |
| \!~\* | 不匹配，不区分大小写 |

&emsp;&emsp;Nginx 在匹配正则时会生成对应表达式中括号被匹配字符的变量，从左至右依次为：$1|$2|$3……可供后续程序中使用匹配内容。例如字符串"ID=user1_acdefg9876543GHI"，使用正则表达式"\(ID=\(\[\-_a\-zA\-Z0\-9\]\+\)\)"进行匹配，所生成的变量：

```code
$1 = ID=user1_acdefg9876543GHI;
$2 = user1_acdefg9876543GHI;
```

### rewrite

&emsp;&emsp;将URL重定向。基本过程：（参考：[博客园Dy1an：【04】Nginx：rewrite / if / return / set 和变量](https://www.cnblogs.com/Dy1an/p/11240223.html)）

1. 用户请求到达某个server，如果满足server内rewrite的正则匹配，那么rewrite将会对用户请求URI重写。
1. 重写完成后直接在该server内部去匹配location。
1. 当匹配到location后，如果location内部又有rewrite，那执行rewrite；之后再次在这个server内部去匹配location，直到请求返回。
1. 这个过程不是无限的，nginx对于这样的跳转就支持10次，如果过多甚至死循环，则会报500错误。

&emsp;&emsp;基本语法：

```nginx
rewrite regex replacement [flag];
```

## Examples

### Example 1

&emsp;&emsp;在正则比较中使用变量（参考：[腾讯云社区问答：如何在nginx“if”正则表达式中使用变量？](https://cloud.tencent.com/developer/ask/55209)）

以下语句似乎不起作用

```nginx
set $chk == "need"; 
set $me "kevin"; 
if ($uri ~ "by-{$me}") { set $chk ""; }
if ($chk == "need") { ... }
```

Solution [*待验证*]

```nginx
set $chk == "need"; 
set $me "kevin"; 
if ($uri ~ /by-([^-]+)/) { set $by $1; }
if ($by = $me) {set $chk "";}
if ($chk == "need") { ... }
```

### Example 2

&emsp;&emsp;去掉指定的url参数（去掉url中的start=...，保留其他参数）：（参考[segmentfault sPeng](https://segmentfault.com/q/1010000005143925)）

Solution1 [*待验证*]

```nginx
location ~* filename(\d+)\.html$ {
    root /var/www/html;
    index index.html;
    if ($query_string ~ ^(.*)&start=(\d+)&(.*)) {
        set $a $1;
        set $b $2;
        set $c $3;
        rewrite ^ /filename1?${a}&${c}? break;
    }
}
```

Solution2 [*待验证*]

```nginx
rewrite ^(.*?)start=\d+\&?(.*)$ $1$2 last;
```
