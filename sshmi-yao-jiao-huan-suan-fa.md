[ssh秘钥交换详解与实现 diffie-hellman-group-exchange-sha](https://www.cnblogs.com/wchrt/p/4534463.html)

ssh的DH秘钥交换是一套复合几种算法的秘钥交换算法。在RFC4419中称为**diffie-hellman-groupX-exchange-shaX** 的算法（也有另一种单纯的 **rsaX-shaX** 交换算法）。本文就以**diffie-hellman-group-exchange-sha256**为例，详尽地讲解整个完整的秘钥交换过程。

    笔者在RFC上和网上看了很久，也只是做了一个大致了解，对实现的帮助不大。实际在实现过程中，有太多的细节需要注意，在很多细节的分歧中，需要自己抱着勇气去测试。（原谅我不看openssh源码和使用openssl库，我只想全部自己实现整个ssh）。在**diffie-hellman-group-exchange-sha256**中，数据的类型非常地重要，因为涉及到hash运算，一定要区别好整数与字符串还有进制。一个小的不同都会导致hash的结果大不一样，hash错了以后的工作都是徒劳。

**diffie-hellman-group-exchange-sha256**的整个过程中一共要用到的秘钥交换算法有：**diffie-hellman、sha256、ssh-rsa**（或其他算法协商的host key，不是单纯的rsa，虾米告诉我ssh用的是**RSASSA-PKCS1-v1\_5 scheme**标准）。而要支持这些加密算法，又需要很多基础算法:多进制大整数、高精度运算、快速模幂、离散算法...（还好自己有一定的acm基础，本来当初是打算学习并实现ssh协议，结果在算法的道路上越走越远，这次就当锻炼自己了。表示以后要好好运用面向对象技术，再也不实现不必要的底层了，以后涉及到安全传输就直接用openssl了）

    以下为各个算法的讲解，先说最重要的几个基础数据结构与计算方法：

    1、**mpint: **二进制补码格式的多精度整数，储存为一个字符，每个字符8位，从高位到低位，负数最高位为1，正数最高位为0（只用到整数）。数据格式为：4字符长度+该长度字符的数值。例如：a1d8：00 00 00 03 00 a1 d8；   4：00 00 00 02 00 04

    2、**多进制大整数**:平时以16进制存储，这样便于使用的时候少转化，转二进制也很快，实现很麻烦，我虽然有自己的大整数模板，但终归是不放心，我就用cryptopp自带的大整数改写过来用（人家可是用的汇编计算的）。

    3、**高精度运算**:主要实现加法乘法和求余即可，因为dh和rsa只需要用到乘法与求余。

    4、**快速模幂**:DH算法和RSA算法都会用到相似的运算：a^b%c 。因为b是一个大整数，因此对于他的幂运算我们将b进行二分，然后对a以及a计算后的结果进行计算，这样能省下大量不必要的运算。几个例子：3^8=3\*3\*3\*3\*3\*3\*3\*3要进行7次运算。而这样化解\(3^4\)\*\(3^4）=（3^4）^2=\(\(3^2\)\*\(3^2\)\)^2=\(\(3^2\)^2\)^2只需要进行3次运算。第一种方式的时间复杂度是O\(n\),第二种二分幂的方法时间复杂度是O\(logn\)，当b为10位十进制的整数时，第一种方式要计算10^10次，而用二分幂的方式只需要计算大约30多次。因为我们要求的是余果，所以在进行幂运算的同时就进行模运算，也能极大减少运算量，否则内存可能都装不下那么大的幂果。下面给出代码实现：

**\[cpp\]**

[view plain](http://blog.csdn.net/wchrt/article/details/45752933)

[copy](http://blog.csdn.net/wchrt/article/details/45752933)

1.     Integer fastpower\_comp\(Integer a,Integer b,Integer c\)  
2.     {  
3. 
4. /\*sample unused fast power 
5.     Integer re=1; 
6.     for\(int i=0;i
   &lt;
   b;i++\) 
7.     { 
8.         re\*=a; 
9.         re%=c; 
10.     } 
11.     return re; 
12.     \*/  
13. 
14. //fast power  
15.     Integer n=c;  
16.     c=1;  
17. while\(b!=0\)  
18.     {  
19. if\(b%2!=0\)  
20.         {  
21.             b=b-1;  
22.             c=\(c\*a\)%n;  
23.         }  
24. else  
25.         {  
26.             b=b/2;  
27.             a=\(a\*a\)%n;  
28.         }  
29.     }  
30. return c;  
31.     }  



    密码算法：

** 1、diffie-hellman**:

        服务器首先产成两个数**G**、**P**,**P**为一个非常大的素数，作为DH算法的**模**；**G**为**密码发生器**，也就是**P**的一个**原根**（不理解没关系），服务器将这两个数发给客户端，用于秘钥的交换。

        客户端生成一个数（客户端的私钥）**x**（0&lt;x&lt;P,但x应该大一点，否则当G特别小时生成的秘钥长度可能会很短，服务器会拒绝），计算**e=\(G^x\)%P**。得到的**e**就是客户端的公钥，客户端将**e**发送给服务器。

        服务器也同客户端一样，生成一个数**y**，计算**f=\(G^y\)%P**。将服务器公钥**f**发送给客户端。

        现在客户端与服务器都知道了对方的公钥，双方把对方的公钥作为自己模幂运算的底数进行运算，服务器计算**K1=\(e^y\)%P**，客户端计算**K2=\(f^x\)%P**可以证明这里**K1==K2 **，得到的**K**值便是双方所交换的秘钥。

**大素数的生成：**我采用了一种猜测加枚举的的方法。做过筛法算素数的都知道当数越大，出现连续素数的概率越高，而且连续的长度越长。我们可以通过随机生成一个大数（最好是奇数），然后判断该素数是否为素数，如果不是，将这个数加二再判断，直到判断为素数即可，以后每次再要取素数就可以把这个结果加再判断（此时素数的几率很高）。



**\[cpp\]**

1. class m\_dh  
2. {  
3. public:  
4.     Integer dh\_g,dh\_p,dh\_x,dh\_e;  
5.     Integer dh\_y,dh\_f;  
6.     Integer dh\_k;  
7. 
8. void set\_g\_and\_p\(
   const Integer g,
   const Integer p\)  
9.     {  
10.         dh\_g=g;  
11.         dh\_p=p;  
12.     }  
13. void set\_y\(Integer y\)  
14.     {  
15.         dh\_y=y;  
16.     }  
17. void set\_f\(Integer f\)  
18.     {  
19.         dh\_f=f;  
20.     }  
21. void comp\_e\(\);  
22.     Integer get\_e\(\)  
23.     {  
24. return dh\_e;  
25.     }  
26. void comp\_k\(\);  
27.     Integer get\_k\(\)  
28.     {  
29. return dh\_k;  
30.     }  
31. };  

**\[cpp\]**



1. void m\_dh::comp\_e\(\)  
2. {  
3. 
4.     dh\_x=mkrandomnum\(50\)+1;  
5.     dh\_e=fastpower\_comp\(dh\_g,dh\_x,dh\_p\);  
6. }  
7. void m\_dh::comp\_k\(\)  
8. {  
9.     dh\_k=fastpower\_comp\(dh\_f,dh\_x,dh\_p\);  
10. }  



**2、rsa:**

这里讲的是裸的rsa算法。

服务器生成两个不同的素数**p**和**q**，计算出模**n=p\*q**，并计算欧拉函数**φ\(n\) = \(p-1\)\(q-1\)**。服务器再在1到φ\(n\)之间生成一个与φ\(n\)互质的的数**e**找到另一个数**d**满足**\(e\*d\)%φ\(n\)**==1。

         现在服务器有三个数**n、e、d **，**ne**的组合为rsa的**公钥**，nd为**私钥**。服务器将公钥发给客户端。在以后的加密解密中，公钥用于加密和签名验证，私钥用于解密与签名。

        加密数字K：计算**C=\(K^e\)%n**，C即为加密后的数据 解密C得到K：**K=\(C^d\)%n **

        签名采用相反的方式，即服务器用私钥加密，客户端用公钥解密，验证解密后的数据。

        然而ssh-rsa使用的是 **RSASSA-PKCS1-v1\_5 scheme**标准，他还含有一些其他的填充值，实际实现的时候需要考虑周全。

**\[cpp\]**

[view plain](http://blog.csdn.net/wchrt/article/details/45752933)

[copy](http://blog.csdn.net/wchrt/article/details/45752933)

1. class m\_rsa  
2. {  
3. public:  
4.     Integer rsa\_e;  
5.     Integer rsa\_n;  
6. void set\_e\_and\_n\(Integer e,Integer n\)  
7.     {  
8.         rsa\_e=e;  
9.         rsa\_n=n;  
10.     }  
11.     Integer comp\_rsa\_result\(Integer num\);  
12. };  
13. 
14. Integer m\_rsa::comp\_rsa\_result\(Integer num\)  
15. {  
16. return fastpower\_comp\(num,rsa\_e,rsa\_n\);  
17. }  

**3、sha256**

散列算法没什么可讲的，主要注意sha256的密文长度是64位的16进制，在进行rsa加解密以及计算sessionid的时候一定要注意关于长度的问题。使用的重点在于需要哪些值以什么样的一种组合方式去参与hash运算。**  
**

        我就直接使用cryptopp的hash算法实现了：

**\[cpp\]**

[view plain](http://blog.csdn.net/wchrt/article/details/45752933)

[copy](http://blog.csdn.net/wchrt/article/details/45752933)

1. class m\_sha  
2. {  
3. public:  
4.     string encode\_sha1\(string data\);  
5.     string encode\_sha256\(string data\);  
6. };  
7. 
8. string m\_sha::encode\_sha256\(string data\)  
9. {  
10.     string hash;   
11.     SHA256 sha256;   
12.     HashFilter hash\_filter \(sha256\);   
13. 
14.     hash\_filter.Attach\(
    new HexEncoder\(
    new StringSink\(hash\), 
    false\)\);   
15.     hash\_filter.Put\(\(byte \*\)data.c\_str\(\),data.length\(\)\);  
16.     hash\_filter.MessageEnd\(\);   
17. 
18. return hash;   
19. }  







**详细过程：**

    基本的算法了解了就可以来看**diffie-hellman-group-exchange-sha256**的整个过程了。![](/assets/ssh-exchange1.png)

    整个交换过程有5个数据包：按顺序分别是1、dh key exchange init；2、dh key exchange reply；3、dh gex init 4、dhgex reply 5、new keys

    1、dh key exchange init（30）

![](/assets/ssh-exchange2.png)

    客户端告诉服务器开始DH交换。



    2、dh key exchange reply（31\)

![](/assets/ssh-exchange3.png)

    服务器将生成的P和G发给客户端。



3、dh gex init\(32\)

![](/assets/ssh-exchange4.png)

    客户端收到服务器发过来的P和G后，自己计算出e返回给客户端



4、dh gex reply（33\)

![](/assets/ssh-exchange5.png)

    服务器收到客户端的e后，根据算法计算出秘钥值K。然后使用sha256算法将一些已知信息hash加密为**H**（具体过程后面会提到），并用rsa将hash值签名。最后发送rsa的公钥、dh的f值、rsa签名后的hash信息发回客户端。



5、new keys（21）

![](/assets/ssh-exchange6.png)

    客户端根据服务器发回的f计算出同样的k值，并根据同样的已有信息hash计算得到**H**后使用服务器发来的rsa公钥校验服务器发回的hash值的签名，根据得到的hash值**H**即会话用的**session\_id**，再进行特定的hash运算（参见下文）即可得到以后用于数据加密的秘钥。如果校验无误，返回new key（21），表示秘钥交换的过程完毕，以后的数据都将由所得秘钥进行加密。





**H**与**session\_id**的计算:

    H=hash\(V\_C\|\|V\_S\|\|I\_C\|\|I\_S\|\|K\_S\|\|e\|\|f\|\|K\);

    按顺序用到的值（注意类型）：



| 类型 | 值 | 说明 |
| :--- | :--- | :--- |
| string | V\_C | 客户端的初始报文（版本信息：SSH-2.0-xxx，不含结尾的CR和LF） |
| string | V\_S | 服务器的初始报文 |
| string | I\_C | 客户端 SSH\_MSG\_KEX\_INIT的有效载荷（不含开头的数据长度值） |
| string | I\_S | 服务器的同上 |
| string | K\_S | 主机秘钥（dh gex reply（33\)过程服务器发送host key \(RSA公钥\)） |
| mpint | e | 客户端DH公钥 |
| mpint | f | 服务器DH公钥 |
| mpint | K | 共同DH计算结果 |

    将以上内容按顺序进行拼接，不要夹杂或尾随多余字符。将拼接后的字符串进行sha256计算出结果**H**。这个**H**就是**session\_id**（会话第一次的秘钥交换生成的的H才是session\_id，以后如果还要进行秘钥交换，session\_id不会改变）。





**加密秘钥计算：**

这里的加密秘钥指的是以后数据通信所用的秘钥，一般用aes算法。**  
**

    计算方式：hash\(K,H,单个字符,session\_id\);

    单个字符指的是单个大写的ASCII字母，根据不同的加密秘钥选择不同的字符来计算。

| 字母 | 秘钥 |
| :--- | :--- |
| 'A' | 客户端到服务器的初始IV（CBC） |
| 'B' | 服务器到客户端的初始IV |
| 'C' | 客户端到服务器的加密秘钥（数据加解密秘钥） |
| 'D' | 服务器到客户端的加密秘钥 |
| 'E' | 客户端到服务器的完整性秘钥\(HMAC\) |
| 'F' | 服务器到客户端的完整性秘钥 |

    哈希计算得到字符串RE，如果我么想要的秘钥长度比RE长，则在RE后面继续加上一个hash值：hash\(K,H,RE\)成为一个加长的RE。还不够继续加上hash\(K,H,RE\)，依次类推





    ssh秘钥交换的过程就告一段落了。笔者在网上找不到合适资料，尤其是这些关于diffie-hellman-group-exchange-sha 的很多细节，自己苦逼了很长时间（本来打算两下撸完去学其他的）终于完成了了。希望给想要自己实现该算法的朋友给予帮助。如还遇到其他的问题可Q我（WCHRT）。

