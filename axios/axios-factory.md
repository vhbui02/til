# AxiosFactory

## Philosophy

What does it do? Why use it instead of raw `axios`?

- Set common baselines for: `baseURL`, `timeout`, default `headers`, ...
- Domain-scoped request/response interceptors
+ Request interceptors: Bearer token injection, auto token refresh, centralized logging)
+ Response interceptors: auto unwrap `response.data`, normalize response envelope (e.g., `response: { code: 200, data: T }`
- Environment driven configuration: simplify pointing to different service instances to distinct microservices (e.g., `API_SECURITY`, `API_DATASOURCE`, ...)

