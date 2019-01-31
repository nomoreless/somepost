---
layout: post
title: bouncycaslte加密库入门教程
---
BouncyCaslte 是一个开源密码算法库，有 Java 和 C# 两个版本，它支持大多数的加密算法，功能上与大名鼎鼎的 OpenSSL 加密库相比毫不逊色，而且更加简单易用，最重要的是它是 Java 语言的，对应 Java 开发者来说，没有比这更有吸引力的了。

# 安装包说明
BouncyCalste 分几个包，可以分别下载, 他们分别是

### bcprov-jdk15on-158.jar

这是提供者和核心包，实现多数的算法，这是我们最常用的包，很多时候，也只需要这个包就可以了。文件名的后面部分的158是版本序号，不同版本不同。

###  bcpkix-jdk15on-158.jar

提供了 PKIX/CMS/EAC/DVCS/PKCS/TSP/OPENSSL 功能的包

###  bcmail-jdk15on-158.jar

这是 S/MIME 功能的包，也就是在处理邮件加解密的时候需要

###  bcpg-jdk15on-158.jar

这是 OpenPGP 的包，可以用于进行 PGP 签名和加密 

###  bctls-jdk15on-158.jar

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

直接贴代码

``` java
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
```

编译运行，可以得到输出
``` 
Cipher = 96d0028878d58c89
```

由于处理的二进制字节，输出的时候，采用`new String(Hex.encode(cipher))`这个方法来转换成16进制的形式。

由于 DES 算法是块加密算法，也就是每次处理一块，每块的大小等于它的密钥长度，也就是8个字节。因此 BufferedBlockCipher 处理的输入的长度必须是8的倍数。 如果不是，将抛出一个异常。 大家可以手动补足8的倍数。也可以采用`PaddedBufferedBlockCipher`代替`BufferedBlockCipher`，它会自动对输入的数据进行填充处理，填充规范遵循 PKCS5/7 的标准。

在`cipherBlock.init(true, parameters);`中，第一个参数设置为true，表示将要对数据进行加密，如果要解密，把该参数设置为false即可。


