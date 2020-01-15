Java安全体系
===

# Java安全体系

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





