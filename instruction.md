前端构建会同时产出 `app.js`、`app.js.br` 和 `app.js.gz`，但 Starlette 的 `StaticFiles` 只能返回原文件，线上要么再套一层代理配置，要么让 GZipMiddleware 每次动态压缩。希望 `StaticFiles` 能可选地直接协商这些预压缩文件，根据请求的 `Accept-Encoding` 选择合适版本；没有可用版本时仍返回原文件，默认行为不能变化。

返回压缩版本时，URL 和原文件的媒体类型保持不变，`Content-Encoding`、`Vary`、长度、缓存校验头要对应实际发送的表示。HEAD、条件请求和 Range 请求也要针对选中的表示工作，不能出现 304 的校验值来自原文件、206 却切回另一个文件这类混用。浏览器明确拒绝某种编码、请求 identity、sidecar 缺失，以及目录页、404 页面和 package 静态目录都要正常退回现有行为。

请沿用现有 `StaticFiles`/`FileResponse` 的安全检查和异步文件处理方式，补齐文档及测试，不要引入运行时压缩器，也不要改变未开启该能力的应用。
