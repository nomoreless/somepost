---
layout: post
title: bouncycaslte加密库入门教程(一)
---
BouncyCaslte 是一个开源密码算法库，有 Java 和 C# 两个版本，它支持大多数的加密算法，功能上与大名鼎鼎的 OpenSSL 加密库相比毫不逊色，而且更加简单易用，最重要的是它是 Java 语言的，对应 Java 开发者来说，没有比这更有吸引力的了。

# 安装包说明
BouncyCalste 分几个包，可以分别下载, 他们分别是

- bcprov-jdk15on-XXX.jar

这是提供者和核心包，实现多数的算法，这是我们最常用的包，很多时候，也只需要这个包就可以了。文件名的后面XXX部分是版本序号，目前是158，不同版本不同。

- bcpkix-jdk15on-XXX.jar

提供了 PKIX/CMS/EAC/DVCS/PKCS/TSP/OPENSSL 功能的包

-  bcmail-jdk15on-XXX.jar

这是 S/MIME 功能的包，也就是在处理邮件加解密的时候需要

-  bcpg-jdk15on-XXX.jar

这是 OpenPGP 的包，可以用于进行 PGP 签名和加密 

-  bctls-jdk15on-XXX.jar

这个是 TLS 的包，可以用来实现服务器和客户端之间的安全传输 

在本篇文章中，我们主要介绍算法的使用，所以，只需要 bcprov-jdk15on-158.jar 这个包就可以, 在pom.xml中添加以下文件即可

``` xml
<!-- https://mvnrepository.com/artifact/org.bouncycastle/bcprov-jdk15on -->
    <dependency>
        <groupId>org.bouncycastle</groupId>
        <artifactId>bcprov-jdk15on</artifactId>
        <version>1.58</version>
    </dependency>
```


# DES加密解密
DES （Data Encryption Stander，数据加密标准）对称加密算法曾经是在工业上用的最多的算法，但是由于其密钥长度只有64比特，在现代计算机面前，已经被攻破，在实际应用中已不建议采用。不过，DES 算法是对称加密的代表，下面就从 DES 算法开始。

加密

``` java
public void encrypt() {
        // 1. 先定义密钥和待加密的明文
        String key = "12345678";
        String plain = "12345678";
        byte cipher[] = new byte[8];

        // 2. 创建engine
        DESEngine engine = new DESEngine();

        // 3. 创建一个paramters
        DESParameters parameters = new DESParameters(key.getBytes());

        // 4. 创建一个CipherBlock
        BufferedBlockCipher cipherBlock = new BufferedBlockCipher(engine);

        cipherBlock.init(true, parameters);
        cipherBlock.processBytes(plain.getBytes(), 0, 8, cipher, 0);

        System.out.println("Cipher = " + new String(Hex.encode(cipher)));
}
```

编译运行，可以得到输出
``` 
Cipher = 96d0028878d58c89
```

由于处理的二进制字节，输出的时候，采用`new String(Hex.encode(cipher))`这个方法来转换成16进制的形式。

由于 DES 算法是块加密算法，也就是每次处理一块，每块的大小等于它的密钥长度，也就是8个字节。因此 BufferedBlockCipher 处理的输入的长度必须是8的倍数。 如果不是，将抛出一个异常。 大家可以手动补足8的倍数。也可以采用`PaddedBufferedBlockCipher`代替`BufferedBlockCipher`，它会自动对输入的数据进行填充处理，填充规范遵循 PKCS5/7 的标准（参见下文的说明）。

在`cipherBlock.init(true, parameters);`中，第一个参数设置为`true`，表示将要对数据进行加密，如果要解密，把该参数设置为false即可。

解密
{% highlight java %}
public void decrypt() {
        // 1. 先定义密钥和待加密的明文
        String key = "12345678";
        String cipherHex = "96d0028878d58c89";
        byte plain[] = new byte[8];

        // 2. 创建engine
        DESEngine engine = new DESEngine();

        // 3. 创建一个paramters
        DESParameters parameters = new DESParameters(key.getBytes());

        // 4. 创建一个CipherBlock
        BufferedBlockCipher cipherBlock = new BufferedBlockCipher(engine);

        cipherBlock.init(false, parameters);
        cipherBlock.processBytes(Hex.decode(cipherHex), 0, 8, plain, 0);

        System.out.println("Plain = " + new String(Hex.encode(plain)));
}
{% endhighlight %}
可以看到，解密的代码和加密的几乎一模一样。只是在`cipherBlock.init(true, parameters);`中，第一个参数设置为`false`，表示将要对数据进行解密.

编译运行，可以得到输出
```
Plain = 3132333435363738
```
刚好是12345678的16进制表示。

# PKCS5/7填充规则

PKCS #7 填充字符串由一个字节序列组成，每个字节填充该填充字节序列的长度。

假定块长度为 8，数据长度为 9，
数据： FF FF FF FF FF FF FF FF FF
PKCS7 填充： FF FF FF FF FF FF FF FF FF 07 07 07 07 07 07 07

简单地说, PKCS5和PKCS7有如下相同的特点:
1)填充的字节都是一个相同的字节
2)该字节的值,就是要填充的字节的个数

如果要填充8个字节,那么填充的字节的值就是0×8;
要填充7个字节,那么填入的值就是0×7;

正是这种即使恰好是8个字节也需要再补充字节的规定，可以让解密的数据很确定无误的移除多余的字节。

在PKCS5Padding中，明确定义Block的大小是8位，而在PKCS7Padding定义中，对于块的大小是不确定的，可以在1-255之间。 

# 总结
- DES 密钥长度 8个字节(即64比特)
- DES加密解密使用同一个密钥
- DES加密解密使用相同的函数，由`init`中的第一个参数决定是加密还是解密
- DES加密的数据长度必须是8的倍数，否则需要手动填充或者采用`PaddedBufferedBlockCipher`