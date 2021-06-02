# NextJS Memo

## Authentication

> [Source](https://auth0.com/blog/ultimate-guide-nextjs-authentication-auth0/)

Next.js 模糊了前端和后端的界限, 这使得目前的生态(已有的关于认证的包)无法提供最优的方案. 

在使用 Next.js 进行认证时, 主要涉及以下场景

- 访问 page 时. 比如: `my_invoices`
- 访问 API route 时. 比如: `api/my/invoices`
- 访问外部 API 是. 比如: 从 `www.mycompany.com` 访问 `billing.mycompany.com/api`