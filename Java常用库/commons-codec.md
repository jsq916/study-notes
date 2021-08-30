[toc]

# commons-codec的使用

## 基础知识

common-codec:处理常用的编码方法的工具类包 例如DES、SHA1、MD5、Base64等

### 依赖

```
	<dependency>
      <groupId>commons-codec</groupId>
      <artifactId>commons-codec</artifactId>
      <version>1.15</version>
    </dependency>
```



## 编码

1. 是将bytes进行编码
2. 需要指定字符串的编码格式

```
Base64 base64 = new Base64();
System.out.println(base64.encode("abcd".getBytes()));
System.out.println("Base64 编码后：" + base64.encodeToString("abcd".getBytes("UTF-8")));
System.out.println("Base64 编码后：" + base64.encodeToString("abcd".getBytes("GBK")));
结果：
[B@7cc355be
Base64 编码后：YWJjZA==
Base64 编码后：YWJjZA==
```

## 解码

1. 可将字符串进行解码
2. 不对的会乱码

```
Base64 base64 = new Base64();
byte[] decode = base64.decode("YWJjZA==");
System.out.println(new String(decode));
System.out.println(new String(base64.decode("abcd".getBytes())));
结果：
abcd
i�
```

