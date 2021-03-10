# Abbreviations and Terms on IT

## RFC

A **Request for Comments (RFC)** is a publication from the <u>Internet Society</u> (ISOC) and its associated bodies, most prominently the <u>Internet Engineering Task Force</u> (IETF), the principal technical development and standards-setting bodies for the Internet.

*征求意见 (RFC) 是互联网协会 (ISOC) 及其相关机构的出版物, 其中最著名的是互联网工程任务组 (IETF), 主要的互联网技术开发和标准制定机构.*

## ISO

**International Organization for Standardization**

## GMT

Greenwich Mean Time, is a time zone.

## UTC

Coordinated Universal Time, is a time standard.

### Why is it Called UTC - not CUT?

It came about as a compromise between English and French speakers.

- **Coordinated Universal Time** in English would normally be abbreviated CUT.
- **Temps Universal Coordonne** in French would normally be abbreviated TUC.

## RESTFUL

REST: **RE**presentational **S**tate **T**ransfer (表述性状态转移) is basically an architectural style of development having some principles:

- It should be stateless
- It should access all the resources form the server using only URI
- It does not have inbuilt encryption
- It does not have session
- It uses one and only one protocal - HTTP
- For performing CRUD operations, it should use HTTP verbs such as `get`, `post`, `put` and `delete`
- It should return the result only in the form of JSON or XML, atom, OData etc.

`REST based services` follow some of the above principles and not all.

`RESTFUL services` follow all the abouve principles.

FUL: full

## TTL

**T**ime **T**o **L**ive: 生存时间. 应用在不同的两个地方

- IPv4 报文头, 由操作系统设定该值, 用于说明 IP 包被路由器丢弃前可以通过的最大网段数量, 用于解决 IP 包在网络中无限循环和收发的问题
- DNS 中, 用于说明一条域名解析记录在 DNS 服务器中的存留时间.