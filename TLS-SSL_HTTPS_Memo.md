# TLS/SSL HTTPS Memo

## 数字签名和数字证书

- **Signature**: 数字签名
- **Digital Certificate**: 数字证书
- **Digest**: 摘要
- **Certificate Authority**: 证书中心. 简称 **CA**

由 **CA** 使用自己的 **Private Key** 加密预颁发给个人或组织的相关信息和 **Public Key** 生成该组织或个人的 **Digital Certificate**.

1. 服务器使用自己的  **Private Key** 对预传输到客户端的数据的 **Digest** 进行加密, 得到 **Signature**;
2. 服务器将数据(未加密), **Signature**, **Digital Certificate** 发给客户端;
3. 客户端使用 **CA** 中心自己的 **Public Key** 解密 **Digital Certificate** 证明证书的真实性的同时提取 **Digital Certificate** 中的 **Public Key**;
4. 客户端使用 **Digital Certificate** 中的 **Public Key** 对 **Signature** 进行解密证明签名的真实性的同时解密出 **Digest**;
5. 客户端计算数据的 **Digest** 与上一步中解密出的 **Digest** 对比, 证实数据未被第三方串改;

## 参考

[数字签名是什么-阮一峰](https://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)