---
title: Java使用MD5和BASE64
date: 2016-07-17 23:00
tags:
  - MD5
  - BASE64
  - 加密
  - 解密
  - 编码
  - 解码
---

# 0X00 简介
最近经常要在代码中使用到BASE64编码和MD5加密，所以把笔记贴在这里方便自己查找。
在配置postfix邮件服务器的时候发现，收到的邮件正文都是使用BASE64编码过的，所以才了解了一下这种编码。
MD5则是加密常用手段。虽说MD5细究不算加密算法，但是可以用作加密。

# 0X01 BASE64编码
>Base64是一种基于64个可打印字符来表示二进制数据的表示方法。由于2的6次方等于64，所以每6个比特为一个单元，对应某个可打印字符。三个字节有24个比特，对应于4个Base64单元，即3个字节需要用4个可打印字符来表示。它可用来作为电子邮件的传输编码。在Base64中的可打印字符包括字母A-Z、a-z、数字0-9，这样共有62个字符，此外两个可打印符号在不同的系统中而不同。一些如uuencode的其他编码方法，和之后binhex的版本使用不同的64字符集来代表6个二进制数字，但是它们不叫Base64。  -----------维基百科

代码需要 `import sun.misc.BASE64Encoder;`
```java
public static String encodeing(String str){
	byte[] b = null;
    String s = null;
    try{
        b = str.getBytes("utf-8");
    }catch (Exception e){
        e.printStackTrace();
    }
    if (b != null){
        s = new BASE64Encoder().encode(b);
    }
    return s;
}
```

# 0X02 BASE64解码
代码需要`import sun.misc.BASE64Decoder;`
```java
public static String decoding(String str){
        byte[] b = null;
        String result = null;
        if (str != null){
            BASE64Decoder decoder = new BASE64Decoder();
            try{
                b = decoder.decodeBuffer(str);
                result = new String(b, "utf-8");
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return result;
    }
```

# 0X03 MD5加密
>MD5消息摘要算法（英语：MD5 Message-Digest Algorithm），一种被广泛使用的密码散列函数，可以产生出一个128位（16字节）的散列值（hash value），用于确保信息传输完整一致。MD5由罗纳德·李维斯特设计，于1992年公开，用以替换MD4算法。这套算法的程序在 RFC 1321 中被加以规范。
将数据（如一段文字）运算变为另一固定长度值，是散列算法的基础原理。
1996年后被证实存在弱点，可以被加以破解，对于需要高度安全性的数据，专家一般建议改用其他算法，如SHA-1。2004年，证实MD5算法无法防止碰撞，因此无法适用于安全性认证，如SSL公开密钥认证或是数字签名等用途。     -----------维基百科

虽说MD5已经被证明不安全，不过用作实验性的登陆验证还是没有问题的。（其实好多好多网站的密码都是MD5的，不信可以去社工库里看看）

代码需要`import java.security.MessageDigest;`

```java
 public static String getMd5(String text){
        try{
            MessageDigest md = MessageDigest.getInstance("MD5");
            md.update(text.getBytes());
            byte b[] = md.digest();
            int i;
            StringBuffer buf = new StringBuffer("");
            for (int offset = 0; offset < b.length; offset++){
                i = b[offset];
                if (i < 0){
                    i += 256;
                }
                if (i < 16){
                    buf.append("0");
                }
                buf.append(Integer.toHexString(i));
            }
            return buf.toString();                      // 32位加密
            //return buf.toString().substring(8, 24);   // 16位加密
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }
    }
```
