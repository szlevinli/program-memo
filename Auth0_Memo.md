# Auth0 Memo

## Get Management API Tokens for Single-Page Applications

- [Available scopes and endpoints](https://auth0.com/docs/tokens/management-api-access-tokens/get-management-api-tokens-for-single-page-applications): Auth0 官方的 Management API, 用于管理 Auth0 中的用户信息;

## Get the user role on login

- [Get the user role on login](https://community.auth0.com/t/get-the-user-role-on-login/39835): 将用户角色加入 IdToken. 使用的 rule;

## Integration with Apollo Client

## Integration with Apollo Server

需要处理两方面内容, 验证 Token 的有效性, 验证用户访问权限.

### 验证 Token 有效性

需要两个包:

- `jsonwebtoken`: An implementation of JSON Web Token;
- `jwks-rsa`: A library to retrieve RSA signing keys from a JWKS (JSON Web Key Set) endpoint.

验证 Token 有效性需要提供一个签名 Key, 该 Key 由 Auth0 官方放置在一个 Web API 的 endpoint 中提供访问, `jwks-rsa` 提供获取该 Key 的方法.

## 容易混淆的概念

- 采用 SAP 的应用程序类型, 基于安全考虑, Tockens (AccessToken, IdToken, RefreshToken) 并不会存储在 session 或 localStorage 中, 而是置入内存由 Auth0 官方提供的 SDK 负责管理. 
- 在基于角色的应用程序中, 我们可以通过 Auth0 的控制台, 将自定义 API 的 AccessToken 中附加登录用户所拥有的 permissions/scopes.

