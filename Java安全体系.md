Java安全体系
===

## 1.相关概念

- JCA (Java Cryptography Architecture)

JCA提供了基础的加密框架，包括Provider框架和一系列API，如消息摘要、密钥生成、加解密算法、数字签名、证书等。

通过不同的Provider实现不同的密码算法。

- JCE (Java Cryptography Extension)

JCE是对JCA的扩展，提供了具体的加密算法。

- JSSE (Java Secure Socket Extension)

JSSE提供对SSL和TLS的扩展，保证网络传输的安全。

- JAAS (Java Authentication Authorization Service)

提供了灵活和可伸缩的机制来保证客户端和服务端的Java程序，作为标准的用户认证和授权模型。

## 2.Java Provider体系

在Java中，JCA不提供具体的密码算法的实现，如果想要使用具体的密码算法，就会用到CSP (Cryptographic Service Provider)。在使用JCA时，Java寻找特定的密码算法提供者，然后调用它们。每个Provider都有唯一的名称，通过名称来选择具体的Provider。

JDK中JCA包含两种组件：

- 定义和支持密码服务的框架。框架包含一些package，如java.security, javax.crypto, javax.crypto.spec, javax.crypto.interfaces.
- 实际实现密码服务的Provider。如Sun, SunRsaSign, SunJCE.

### 2.1 CSP

java.security.Provider是所有CSP的base class。每个CSP包含了这个类的一个实例，包含CSP的名字和它实现的所有安全服务/算法的列表。当需要调用特定的算法时，JCA会查询Provider的database，如果找到合适的Provider，就会创建一个实例。

每个JDK默认会安装和配置一个或多个CSP。用户可以配置Provider的preference order。如果没有提供特定的Provider参数，JCA就会按照preference order进行搜索。例如：

```java
md = MessageDigest.getInstance("SHA-256");
md = MessageDigest.getInstance("ShA-256", "ProviderC");
```

如下图所示，图一表示当没有设置具体的提供者时，会按顺序搜索"SHA-256"，因此返回ProviderB的实例；图二表示当设置了具体的Provider时，会直接返回指定Provider的实例，因此返回ProviderC的实例。

![图一](https://docs.oracle.com/javase/8/docs/technotes/guides/security/overview/images/jssec_dt_011.png)
![图二](https://docs.oracle.com/javase/8/docs/technotes/guides/security/overview/images/jssec_dt_012.png)

### 2.2 Provider的实现

Algorithm independence定义了通用的high-level的接口，应用通过这些接口访问安全服务；Implementation independences使所有Provider遵从well-defined接口；Engine类路由应用的请求，并发送到下层的实现。

实现SPI（Service Provider Interface）的类将Engine类的应用API路由到Provider的实现。因此，对于每个engine class，都有一个对应的abstract SPI类，它定义了每个CSP的算法必须实现的方法。

> Each SPI class is abstract. To supply the implementation of a particular type of service for a specific algorithm, a provider must subclass the corresponding SPI class and provide implementations for all the abstract methods.

对于API中每个engine class，通过调用其中的getInstance工厂方法请求和实例化实现类的实例。

> A factory method is a static method that returns an instance of a class. The engine classes use the framework provider selection mechanism described above to obtain the actual backing implementation (SPI), and then creates the actual engine object. 

每个engine类的实例封装了对应的SPI的实例，即SPI对象。

> All API methods of an API object are declared final and their implementations invoke the corresponding SPI methods of the encapsulated SPI object.

例如：

```java
import javax.crypto.*;
Cipher c = Cipher.getInstance("AES");
c.init(ENCRYPT_MODE, key);
```

![AES调用实例](https://docs.oracle.com/javase/8/docs/technotes/guides/security/images/jca/ArchDesignPrincipals.gif)

如上图所示，应用想要获取一个AES (javax.crypto.Cipher)实例，但并不关系是哪个CSP提供的。应用调用Cipher engine class的getInstance()工厂方法，然后按顺序访问JCA框架去寻找支持AES的第一个Provider。

框架查询每一个安装的Provider，并获取Provider类的实例(the Provider class is a database of available algorithms)。框架检索每一个Provider，并在CSP3中找到合适的entry。这个entry指向了实现类com.foo.AESCiper,其继承了CipherSpi，因此满足Cipher engine class使用的要求。

一个com.foo.AESCipher实例被创建，并且封装在一个新建的javax.crypto.Cipher实例中。这个javax.crypto.Cipher实例返回给应用。

当应用调用Cipher实例的init方法时，Cipher engine class会路由到com.foo.AESCipher对应的engineInit方法.

### 2.3 Key Management

可以用一个叫做"keystore"的database管理密钥和证书库。那些需要认证、加密、签名的应用会用到keystore。应用可以通过KeyStore类获取keystore，其包含在java.security包中。

> The recommended keystore type (format) is "pkcs12", which is based on the RSA PKCS12 Personal Information Exchange Syntax Standard. 

> The default keystore type is "jks", which is a proprietary format. 

> Other keystore formats are available, such as "jceks", which is an alternate proprietary keystore format

> "pkcs11", which is based on the RSA PKCS11 Standard and supports access to cryptographic tokens such as hardware security modules and smartcards.

## Engine Class and Algorithm

Engine class提供接口给特定类性的cryptographic service, independent of a particular cryptographic algorithm or provider。Engine可以支持：

- 密码学操作 (加密、数字签名、消息摘要等)
- 密码资料的生成器和转换器 (密钥和算法参数)
- 封装加密数据的对象 (keystores and certificate) 




