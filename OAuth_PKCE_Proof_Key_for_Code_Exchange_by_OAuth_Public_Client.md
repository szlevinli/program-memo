@[TOC](OAuth PKCE --- Proof Key for Code Exchange by OAuth Public Client)

## 介绍

> 在 OAuth 2.0 规范中 **授权码** 许可类型对于 Public Client (比如: SPA 或 Native APP) 这种客户端应用程序时, 安全性得不到有效的保证, 攻击者可能会拦截到认证服务器返回的授权码, 从而导致安全信息泄露.

```reStructuredText
    +~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+
    | End Device (e.g., Smartphone)  |
    |                                |
    | +-------------+   +----------+ | (6) Access Token  +----------+
    | |Legitimate   |   | Malicious|<--------------------|          |
    | |OAuth 2.0 App|   | App      |-------------------->|          |
    | +-------------+   +----------+ | (5) Authorization |          |
    |        |    ^          ^       |        Grant      |          |
    |        |     \         |       |                   |          |
    |        |      \   (4)  |       |                   |          |
    |    (1) |       \  Authz|       |                   |          |
    |   Authz|        \ Code |       |                   |  Authz   |
    | Request|         \     |       |                   |  Server  |
    |        |          \    |       |                   |          |
    |        |           \   |       |                   |          |
    |        v            \  |       |                   |          |
    | +----------------------------+ |                   |          |
    | |                            | | (3) Authz Code    |          |
    | |     Operating System/      |<--------------------|          |
    | |         Browser            |-------------------->|          |
    | |                            | | (2) Authz Request |          |
    | +----------------------------+ |                   +----------+
    +~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~+

             Figure 1: Authorization Code Interception Attack
```

> 为了解决这个问题, OAuth 提供了一种对 授权码许可类型的扩展类型, 即 `PKCE` --- Proof Key for Code Exchange

```
                                                 +-------------------+
                                                 |   Authz Server    |
       +--------+                                | +---------------+ |
       |        |--(A)- Authorization Request ---->|               | |
       |        |       + t(code_verifier), t_m  | | Authorization | |
       |        |                                | |    Endpoint   | |
       |        |<-(B)---- Authorization Code -----|               | |
       |        |                                | +---------------+ |
       | Client |                                |                   |
       |        |                                | +---------------+ |
       |        |--(C)-- Access Token Request ---->|               | |
       |        |          + code_verifier       | |    Token      | |
       |        |                                | |   Endpoint    | |
       |        |<-(D)------ Access Token ---------|               | |
       +--------+                                | +---------------+ |
                                                 +-------------------+

                     Figure 2: Abstract Protocol Flow
```

## 流程

> PKCE 的流程分为两个部分:
>
> 1. 授权码请求 --- Authorization Request: 请求授权码
> 2. 授权码交换 --- Authorization Code Exchange: 使用授权码交换访问令牌

### 授权码请求

> 在客户端应用程序启动浏览器前, 首先生成一个称之为 `code verifier` 的随机加密字符串 (由 A-Z, a-z, 0-9, -._~ 组成), 在 43 到 128 个字符长度之间.

> 接着客户端应用程序使用 `code verifier` 生成一个称之为 `code challenge` 的采用 SHA256 HASH 算法基于 BASE64-URL-encoded 生成的字符串. 也可以直接使用 `code verifier` 作为 `code challenge`.

> 现在客户端应用程序可以使用其代理程序 (比如: 浏览器) 发起 **授权码请求**, 提供如下请求参数到授权服务器的授权码请求 end-point:
>
> - **response_type=code:** 只是服务器期望接收授权码.
> - **client_id:** 客户端应用程序唯一标识符, 通常由授权服务器颁发.
> - **redirect_uri:** 指示授权完成后将用户定向到的地址.
> - **state:** 客户端应用程序产生的随机字符串, 后面的流程中会用到.
> - **code_challenge:** 之前所描述的 `code challenge`.
> - **code_challenge_method:** `plain|S256` 如果采用 SHA256 算法将 `code verifier` 转换为 `code challenge` 则填写 `S256`, 如果直接使用 `code verifier` 作为 `code challenge`, 则填写 `plain`.

### 授权码交换

> 现在客户端应用程序可以使用其代理程序 (比如: 浏览器) 发起 **授权码交换**, 提供如下请求参数到授权服务器的授权码交换请求 end-point:
>
> - **grant_type=authorization_code:** 指示令牌请求的许可类型.
> - **code:** 上面请求中获得的授权码.
> - **redirect_uri:** 在上面请求中填写的 redirect URL 地址.
> - **client_id:** 客户端应用程序唯一标识符, 通常由授权服务器颁发.
> - **code_verifer:** 之前所描述的 `code verifier`.

