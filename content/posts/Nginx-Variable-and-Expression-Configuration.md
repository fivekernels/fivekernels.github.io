---
title: "Nginx 配置 变量和表达式"
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

- $arg_\<PARAMETER\>: 客户端GET请求中 \<PARAMETER\>字段的值

#### 客户端证书

参考：<https://blog.csdn.net/tuzongxun/article/details/91477954>

nginx内部解析证书后，会把相关信息放到内置的变量中

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

### 比较

**数值比较:**

&emsp;&emsp;比较两个变量/变量与字符串是否相等

|  符号  |  说明   |
| :----: | :----: |
| =      | 相等   |
| !=    | 不相等  |

**正则比较:**

&emsp;&emsp;变量与正则表达式的模式是否匹配

|  符号  |  说明   |
| :----: | :----: |
| ~ | 匹配，区分大小写 |
| ~* | 匹配，不区分大小写 |
| !~ | 不匹配，区分大小写 |
| !~* | 不匹配，不区分大小写 |

&emsp;&emsp;Nginx 在匹配正则时会生成对应表达式中括号被匹配字符的变量，从左至右依次为：$1|$2|$3……可供后续程序中使用匹配内容。例如字符串"ID=user1_acdefg9876543GHI"，使用正则表达式"(ID=([-_a-zA-Z0-9]+))"进行匹配，所生成的变量：

```code
$1 = ID=user1_acdefg9876543GHI;
$2 = user1_acdefg9876543GHI;
```
